상속을 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다.

**메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.**

→ 상위 클래스의 구현 방식에 따라 하위 클래스 동작에 이상이 생길 수 있다. (상위 클래스는 릴리스마다 내부 구현이 달라질 수 있다보니, 코드 한 줄 건드리지 않은 하위 클래스가 오동작 할 수 있다.)

# 상속으로 인한 문제 예시

HashSet을 사용하는 counter 클래스를 만드는 예시를 보여준다.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
	private int addCount = 0; //추가된 원소의 카운트

	....

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}
}

//실행구문
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```

위 구문은 문제가 있다. 결과값이 3으로 나올것으로 예상하지만 6을 반환한다.

→ 이유는 `HashSet` 의 `addAll` 메서드가 `add` 메서드를 사용해 구현되어 있기 때문이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8782318a-8164-47fb-8699-7206893034cc/Untitled.png)

이러한 예시를 통해 해결해보기 위한 방법은 아래와 같다.

1. addAll 메서드를 재정의 하지 않는다.
    
    → 당장은 제대로 동작하나, HashSet의 addAll이 add 메서드를 이용해 구현했다는 가정을 한 것이다. 차후 로직이 바뀔경우 어떻게 대처할것인지 해결이 어렵다.
    
2. addAll 메서드의 내용을 바꿔 add를 호출하도록 한다.
    
    → 성능이 떨어질 수 있고, 또한 하위 클래스에서 접근할 수 없는 private 필드를 써야 하는 상황이면 구현이 불가능하다.
    

## 그래서?

클래스를 확장하더라도 메서드를 재정의하는 대신 새 메서드를 추가하면 괜찮다고 생각이 들 수 있지만 이것도 위험하다.

→ 상위 클래스에 새 메서드가 추가됐는데 하필 정의한 메서드와 시그니처가 같고 반환 타입이 다르다면 아예 컴파일 자체도 되지 않을 것이다. 반환 타입이 같다면 재정의한 꼴이니 동일 상황에 부딪힌다.

이 문제를 피해가는 방식은, **기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하는 것이다.**

**기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이런 설계를 컴포지션(구성)이라고 한다.**

**새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환하는데, 이 방식을 전달(forwarding)이라 하며 해당 메서드들을 전달 메서드라 부른다.**

이런 새 클래스를 통해 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 기존 클래스에 새 메서드가 추가되더라도 전혀 영향받지 않는다.

아래는 위 코드를 컴포지션을 통해 구성한 코드이다.

```java
//전달 클래스
public class ForwadingSet<E> implements Set<E> {
	private final Set<E> s;
	
	public void clear() { s.clear(); }
	...
	public boolean add(E e) { return s.add(e); }
	public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
}

//컴포지션 클래스
public class InstrumentedHashSet<E> extends ForwadingSet<E> {
	private int addCount = 0; //추가된 원소의 카운트

	....

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}
}

//실행구문
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
```

- Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다.
    - 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.
- 한번 구현해 두면 어떠한 Set 구현체라도 계측할 수 있다.

# 래퍼클래스

위의 코드처럼 다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedHashSet 클래스는 **‘래퍼클래스'라고 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 ‘데코레이터 패턴'이라고 한다.**

- 컴포지션과 전달의 조합은 넓은 의미로 위임이라고 부르지만, 엄밀히 따지면 래퍼객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당하긴 한다.

단점이 딱 한가지 있는데, **래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다.**

→ 콜백 프레임 워크에서는 자기 자신의 참조를 다른 객체에 넘겨 사용하게 하는데, 내부 객체는 래퍼의 존재를 몰라 자신의 참조를 넘기고 콜백떄는 래퍼가 아닌 내부 객체를 호출하게 된다. 이를 SELF 문제라고 한다.

**성능에 영향을 줄 수 있다고 생각하지만 별다른 영향이 없다고 밝혀졌다.**

전달 클래스를 인터페이스당 하나씩 만들어두면 원하는 기능을 덧씌우는 전달 클래스들을 아주 손쉽게 구현할 수 있는데, 구아바 라이브러리의 경우는 전부 구현해두었다.

# 상속을 할 때 필수 체크

반드시 **하위 클래스가 상위 클래스의 ‘진짜' 하위 타입인 상황에서만 쓰여야 한다.**

→ 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다. (B가 정말 A인가?)

## 자바 상속을 잘못 사용한 예

- Stack → Vector를 확장해서 사용하였음
- Properties → Hashtable을 확장하여 사용하였음

## 그래서?

- 컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 것이다.
    - API가 내부 구현에 묶이고 클래스의 성능도 영원히 제한된다.
    - 클라이언트가 노출된 내부에 직접 접근할 수 있다. (사용자를 혼란스럽게 할 수 있다.)
        - *(ex) p.getProperty(key) → Properties의 기본 동작, p.get(key) → Hashtable로부터 물려받은 메서드*
- 컴포지션 대신 상속을 사용하기로 결정하였다면 아래의 질문에 자문해보자.
    - 확장하려는 클래스의 API에 아무런 결함이 없는가?
    - 결함이 있다면, 이 결함이 해당 클래스의 API까지 전파되어도 괜찮은가?

# 정리

- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야한다.
    - 해당 관계임에도 안심할 순 없는데, 상위클래스가 확장을 고려해 설계되지 않았다면 문제가 될 수 있다.
- 상속의 취약점을 피하려면 컴포지션과 전달을 사용하자.
- 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.