해당 챕터는 [깊은 복사, 얕은 복사](https://zzang9ha.tistory.com/372) 등의 개념을 알고 있어야 이해가 쉽다.

# Cloneable

`Cloneable` 은 복제해도 되는 클래스임을 명시하는 용도의 mixin interface 이지만 단점이 크다.

## 단점

- `clone` 메서드가 선언된 곳이 `Object` 이고 그마저도 `protected` 여서 `Cloneable` 을 구현하는 것 만으론 외부 객체에서 `clone` 메서드를 호출할 수 없다.
- 리플렉션을 사용하면 가능하지만 해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없어서 100% 성공하는것도 아니다.

**하지만 Cloneable 방식은 널리 쓰이고 있으니 잘 알아두자.**

## 하는일

**이 인터페이스는 `Object` 의 protected 메서드인 `clone` 의 동작 방식을 결정**한다.

- `Cloneable`을 구현한 클래스의 인스턴스에서 `clone()`을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환
- 그렇지 않은 클래스 인스턴스에서 호출하면 `CloneNotSupportedException` 반환

실제 인터페이스 구현이라고 함은 해당 클래스가 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위이지만, `**Cloneable` 의 경우에는 상위 클래스에 정의된 메서드의 동작 방식을 변경\*\*한다.

# clone 메서드 재정의 시 주의할 점

## 일반적인 clone

```java
class PhoneNumber implements Cloneable {
	@Override
	public PhoneNumber clone() {
		try {
			return (PhoneNumber) super.clone();
		} catch (ClassNotSupportedException e) {
			// 아무 처리를 하지 않거나, 다른 Exception을 반환
		}
	}
}
```

- `super.clone()` 을 실행하면 해당 클래스에 대한 완벽한 복제가 이뤄진다.
- 자바의 공변 타이핑 기능을 통해 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입으로 정의 가능하다.
  → 그래서 Object의 하위 타입인 PhoneNumber 로 정의 가능한 것
- `super.clone()` 에서 Checked Exception을 리턴하기 때문에 try-catch 문을 사용했지만 해당 클래스가 `Cloaneable` 을 구현하기 때문에 절대 실패하지 않는다. 그래서 해당부분은 아무 처리를 하지 않거나, 다른 Exception을 반환해야한다.

## 가변객체 clone

```java
public class Stack implements Cloneable {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object o) {
		ensureCapacity();
		elements[size++] = e;
  }
  ...
}
```

### 주의점

해당 클래스를 복제할 때 원시타입인 size는 올바르게 복제되겠지만, elements 필드는 주소값을 복제할 것이다.

→ 생성자를 호출한다면 위 상황은 일어나지 않을 것이다.

**clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.**

### 방법 1. clone을 재귀적으로 호출

위 상황을 해결하는 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출하는 것이다.

```java
@Override
public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
```

- **단점**
  - elements 필드가 final이면 작동하지 않는다.
    - 결국 ‘가변 객체를 참조하는 필드는 final로 선언하라' 라는 일반 용법과 충돌한다.
  - 해시 테이블 같은 자료구조에는 사용할 수 없다.

### 방법 2. 깊은 복사를 재귀 호출 대신 반복자를 써서 순회

```java
Entry deepCopy() {
	Entry result = new Entry(key, value, next);
	for (Entry p = result; p.next != null; p = p.next)
		p.next = new Entry(p.next.key, p.next.value, p.next.next);

	return result;
}
```

### 방법 3. 원본 객체를 다시 생성하는 고수준 메서드 호출

1. `super.clone` 을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정
2. 원본 객체의 상태를 다시 생성하는 고수준 메서드 호출

- **장점**
  - 모든 값을 제대로 복제하여 간단하고 우아한 코드를 얻을 수 있다.
- **단점**
  - 저수준에서 바로 처리할 때 보단 느리다.
  - `Cloneable` 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하여 해당 아키텍처와 어울리지 않는다.

## 그래서

- clone이 재정의 될 수 있는 메서드를 호출하면 안된다. (아이템 19)
  - 혹시나 재정의된 메서드를 호출하면 복제 과정에서 자신의 상태를 교정할 기회를 읽어, 원본과 복제본의 상태가 달라질 수 있다.
- public인 clone 메서드에서는 throws 절을 없애야 한다. (아이템 71)
- 상속용 클래스는 Cloneable을 구현해선 안된다.
  - 하위 클래스에서 Cloneable을 지원하지 못하게 clone에 접근하면 Exception을 내도록 처리할 수 있다.
- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드도 적절히 동기화 해줘야한다.

### 정리

- Cloneable을 구현하는 모든 클래스는 clone을 재정의해야한다.
- 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다.
- 가장 먼저 `super.clone` 을 호출한 후, 필요한 필드를 전부 적절히 수정한다.
  → 깊은 복사

## 복사 생성자와 복사 팩터리

### 복사생성자

```java
public Yum(Yum yum) { ... };
```

### 복사팩토리

```java
public static Yum newInstance(Yum yum) { ... };
```

복사 생성자를 모방한 정적 팩터리이다.

### Cloneable/clone 방식과 비교

- 생성자를 쓰지 않고 객체를 생성하는 메커니즘을 사용하지 않는다.
- 엉성하게 문서화된 규약에 기대지 안흔다.
- 정상적인 final 필드 용법과도 충돌하지 않는다.
- 불필요한 검사 예외를 던지지 않는다.
- 형변환도 필요치 않다.
- 해당 클래스가 구현한 ‘인터페이스'타입의 인스턴스를 인수로 받을 수 있다.
  - 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.

# 정리

- 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장하면 안되며, 새로운 클래스도 이를 구현해선 안된다.
  - final 클래스라면 위험이 크진 않지만 성능 최적화 관점에서 검토한 후 문제 없을때 드물게 허용해야 한다.
- 복제기능은 생성자와 팩터리를 이용하는게 최고이다.
  - 단, 배열은 clone 메서드 방식이 가장 깔끔한 합당한 예외이다.
