# 목차

[01. 생성자 대신 정적 팩터리 메서드를 고려하라](#01.-생성자-대신-정적-팩터리-메서드를-고려하라)

[02. 생성자에 매개변수가 많다면 빌더를 고려하라](#02-생성자에-매개변수가-많다면-빌더를-고려하라)

[03. private 생성자나 열거 타입으로 싱글턴임을 보증하라](#03-private-생성자나-열거-타입으로-싱글턴임을-보증하라)

[04. 인스턴스화를 막으려거든 private 생성자를 사용하라](#04-인스턴스화를-막으려거든-private-생성자를-사용하라)

[05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.](#05-자원을-직접-명시하지-말고-의존-객체-주입을-사용하라)

[06. 불필요한 객체 생성을 피하라](#06-불필요한-객체-생성을-피하라)

[07. 다 쓴 객체 참조를 해제하라](#07-다-쓴-객체-참조를-해제하라)

[08. finalizer와 cleaner 사용을 피하라](#08-finalizer와-cleaner-사용을-피하라)

[09. try-finally 보다는 try-with-resources를 사용하라](#09-try-finally-보다는-try-with-resources를-사용하라)

---

# 01. 생성자 대신 정적 팩터리 메서드를 고려하라

> 해당 파트의 정적 팩터리 메서드는 디자인 패턴에서의 팩터리 메서드와 다르다. 디자인 패턴중엔 이와 일치하는 패턴은 없다.

클래스는 클라이언트에 public 생성자 (or 생성자) 대신 정적팩터리 메서드를 제공할 수 있다.

# 장점

## 1. 이름을 가질 수 있다.

생성자에 넘기는 매개변수와 생성자로는 반환 객체의 특성을 제대로 설명할 수 없지만, 정적팩터리는 이름만 잘 짓는다면 반환 객체의 특성을 쉽게 묘사할 수 있다.

**_예시 (값이 소수인 BigInteger를 반환하는 코드)_**

```java
//Bad: 생성자 사용
new BigInteger(int, int, Random);

//Good: 정적 팩터리 메서드 사용 (더 특성이 명확하다)
BigInteger.probablePrime(int, int, Random);
```

또한 하나의 시그니처로는 생성자를 하나만 만들 수 있다. 살짝 틀어 여러개의 생성자를 만들 수는 있지만 정확히 어떤 역할을 하는것인지 기억하기 어려워 엉뚱한 것을 호출하는 실수를 할 수 있다.

**한 클래스에 시그니처가 같은 생성자가 여러개 필요할 것 같다면, 생성자를 정적 팩터리 메서드로 바꾸고 각 차이를 잘 드러내는 이름을 지어주자.**

## 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

이부분으로 **불변 클래스는 인스턴스를 미리 만들어 두거나 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.**

_(ex)_ `Boolean.valueOf(boolean)` 메서드는 객체를 아예 생성하지 않는다. 따라서 **생성 비용이 있는 객체가 자주 요청되는 상황이면 성능에 도움**이 된다.

(비슷한 패턴) Flyweight pattern

또한 **같은 객체를 반환하는 방식으로 인스턴스 통제 클래스 클래스로 사용할 수 있다.**

인스턴스를 통제하면 singleton 혹은 인스턴스화 불가(noninstantiable)로 만들 수 있다. 또한 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다. (equals)

이러한 인스턴스 통제는 플라이웨이트 패턴, 열거 타입의 근간이 된다.

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

API를 만들 때 해당 유연성을 응용하면 **구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.** 이는 인터페이스 기반 프레임워크를 만드는 핵심 기술이다.

**자바 8 이전**

인터페이스에 정적 메서드를 선언할 수 없어 동반 클래스를 만들어 내부에 정의하였다. 덕분에

_(ex)_ 자바 컬렉션 프레임 워크(`java.util.Collections`) 내의 정적 팩터리 메서드

**자바 8 이후**

인터페이스 내 정적 메서드 제한이 풀려 정적 멤버들을 인터페이스 자체에 둔다.

대신 **대부분의 정적메서드 구현 코드는 별도 package-private 클래스에 두어야 할 수밖에 없다.**

자바 8에서는 public 정적 멤버만 허용한다. 자바 9에서는 private 정적 메서드까지 허락하지만 정적 필드와 정적 멤버 클래스는 public 이어야한다.

## 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

반환타입의 하위타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없다.

클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴지인지 알 수도 없고 알 필요도 없다. 단순 하위 클래스기만 하면 된다.

_(ex)_ EnumSet 클래스

## 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

이런 유연함은 **서비스 제공자 프레임워크(service provider framework)를 만드는 근간**이 된다.

→ 제공자(provider)는 서비스의 구현체인데, 이런 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여 클라이언트를 구현체로부터 분리해준다.

서비스 제공자 프레임워크는 3개 + 1개의 핵심 컴포넌트로 이뤄진다.

1. 서비스 인터페이스
   - 구현체의 동작을 정의
2. 제공자 등록 API
   - 제공자가 구현체를 등록 할 때 사용
3. **서비스 접근 API → ‘상기의 유연한 정적 팩터리’**
   - 클라이언트가 서비스의 인스턴스를 얻을 때 사용
4. (종종사용) 서비스 제공자 인터페이스
   - 서비스 인터페이스의 인스턴스르 생성하는 팩터리 객체를 설명
   - 없다면 각 구현체를 인스턴스로 만들 때 리플렉션을 사용해야 한다.

_(ex)_

1. JDBC
   - Connection → 서비스 인터페이스 역할
   - DriverManager.registerDriver → 제공자 등록 API 역할
   - DriverManager.getConnection → 서비스 접근 API 역할
   - Driver → 서비스 제공자 인터페이스 역할
2. 브릿지 패턴 (Bridge pattern)
3. 의존 객체 주입(DI) 프레임워크

# 단점

## 1. 정적 팩터리 메서드만 제공시 하위 클래스를 만들 수 없다.

상속을 하려면 `public` 이나 `protected` 생성자가 필요하기 때문이다.

다른 관점에서 생각하면 해당 제약은 **상속보다 컴포지션을 사용하도록 유도하고, 불변타입으로 만들려면 해당 제약을 지켜야 한다는 점에서 오히려 장점이 될 수도 있다.**

## 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자처럼 API설명에 명확히 드러나지 않기 때문이다.

**API 문서를 잘 쓰고, 또한 메서드 이름도 알려진 규약을 따라 짓는식으로 문제를 완화시켜야 한다.**

### 정적 패터리 메서드 네이밍 컨벤션

- **from**
  - 매개변수를 하나 받아 해당 타입의 인스턴스를 반환하는 형변환 메서드
  - `Date d = Date.from(instant);`
- **of**
  - 여러 매개변수를 받아 적합 타입 인스턴스를 반환하는 집계 메서드
  - `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
- **valueOf**
  - from과 of의 더 자세한 버전
  - `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
- **instance 혹은 getInstance**
  - 매개변수로 명시한 인스턴스를 반환하지만 같은 인스턴스임을 보장 X
  - `StackWalker luke = StackWalker.getInstance(options);`
- **create 혹은 newInstance**
  - instance, getInstance와 같지만 매번 새로운 인스턴스를 생성해 반환함을 보장
  - `Object newArray = Array.newInstance(classObject, arrayLen);`
- **getType**
  - getInstance와 같지만 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할때 사용.
  - ‘Type’ 부분은 팩터리 메서드가 반환할 객체의 타입이다.
  - `FileStore fs = Files.getFileStore(path)`
- **newType**
  - getType과 동일하게 사용하며, newInstance와 같다.
  - `BufferedReader br = Files.newBufferedReader(path)`
- **type**
  - getType과 newType의 간결한 버전
  - `List<Complaint> litany = Collections.list(legacyLitany)`

# 요약

정적 팩터리 메서드와 public 생성자는 각 쓰임새에 따라 장단점을 이해하고 사용하자.

하지만 대부분 정적 팩터리를 사용하는게 유리한 경우가 더 많다.

---

# 02. 생성자에 매개변수가 많다면 빌더를 고려하라

정적팩터리와 생성자에는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다. 이 경우 아래와 같은 방식으로 해결해 왔다.

# 1. 점층적 생성자 패턴

필수 매개변수만 받는 생성자, 필수 + 선택 1개를 받는 생성자 … 형태로 **매개변수의 개수대로 생성자를 만드는 방식**이다.

```java
public class User {
	private final String name;
	private final int age;
	private final String phone;

	public User(int name, int age) {
		this(name, age);
	}

	public User(String name, int age, String phone) {
		this(name, age, phone);
	}
}
```

## 단점

1. **매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기가 어려워진다.**
2. 매개변수에 딱 알맞은 생성자가 없다면 불필요한 매개변수에 임의의 값을 지정해주어야 한다.

이러한 부분으로 실수가 잦아질 수 있으며, 버그로 이어질 수 있다.

# 2. 자바빈즈 패턴

매개변수가 하나도 없는 생성자로 객체를 만든 후, **Setter 메서드 들을 호출해 원하는 매개변수의 값을 설정하는 방식**이다.

```java
public class User {
	private final String name;
	private final int age;
	private final String phone;

	public User() {}

	public void setName(String name) { this.name = name; }
	public void setAge(String age) { this.age = age; }
	public void setPhone(String phone) { this.phone = phone; }
}
```

## 단점

1. 객체 하나를 만들기 위해 메서드를 여러개 호출해야 한다.
   - 객체가 완전히 생성되기 전 까지는 일관성(consistency)가 무너진 상태에 놓인다.
   - 클래스를 불변으로 만들 수 없다.

이러한 단점을 완화하고자 생성이 끝난 객체를 수동으로 ‘얼리고(freezing)’ 얼리기 전에는 사용할 수 없도록 한다.

→ JavaScript를 사용할때 const 사용이 불가할 때 freeze를 사용해봤지만 여간 불편한게 아니였다.

# 3. 빌더 패턴 (해결사)

클라이언트가 필요한 객체를 직접 만드는 대신, **필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 이후에 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정**한다. 마지막으로 `build` 메서드를 호출해 객체를 얻는다.

```java
public class User {
	private final String name; //필수
	private final int age; //필수

	private final String phone;//선택

	public static class Builder {
    private final String name;
    private final int age;
    private final String phone;

    public Builder(String name, int age) { //필수
      this.name = name;
      this.age = age;
    }

    public Builder phone(String phone) { //선택
      this.phone = phone;
      return this;
    }

    public User build() {
      return new User(this);
    }
  }

  private User(Builder builder) {
    this.name = builder.name;
    this.age = builder.age;
    this.phone = builder.phone;
  }
}
```

빌더 패턴을 사용하면 **불변 클래스를 만들 수 있다.** 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 **메서드 체이닝도 가능**하다.

> **불변(immutable)이란 어떠한 변경도 허용하지 않는다는 뜻**이다. 대표적으로 String 객체가 있다.
> **불변식(invariant)이란 정해진 기간동안 반드시 만족해야 하는 조건**을 말한다. 예컨대 리스트의 크기는 반드시 0 이상이어야 하니 한순간이라도 음수값이 된다면 불변식이 깨진것이다.

해당 빌더 패턴은 파이썬과 스칼라에 있는 명명된 선택적 매개변수(named optional parameters)를 흉내낸 것이다.

아래는 빌더 패턴을 사용한 예시이다.

```java
User minhyun = new User.Builder("minhyun", 99)
	.phone("010-0000-0000")
	.build();
```

## 장점

1. 쓰기 쉽고, 읽기 쉽다.
2. 계층적으로 설계된 클래스와 함께 쓰기 좋다. (책에 피자로 든 예제가 있다.)
3. 가변인수(varargs) 매개변수를 여러개 사용할 수 있다.
4. 생성에 유연하다.
   1. 빌더 하나로 여러 객체를 순회하면서 만들 수 있따.
   2. 매개변수에 따라 다른 객체를 만들 수 있다.
   3. 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수도 있다.

## 단점

1. 객체를 만들기 전 빌더부터 만들어야 한다.
   1. 생성비용이 크지는 않지만 성능에 민감한 상황에선 문제가 될 수 있다.
2. 코드가 장황하여 매개변수가 4개 이상은 되어야 값어치를 한다.
   1. 하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있으니 큰 문제는 없다.

# 정리

**생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다.** 점층적 생성자보다 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.

---

# 03. private 생성자나 열거 타입으로 싱글턴임을 보증하라

> 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 전형적인 예로는 함수와 같은 무상태 객체나 유일해야하는 시스템 컴포넌트를 들 수 있다.

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 질 수 있다.

# 싱글턴을 만드는 방식

두 방식 모두 생성자는 private로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 둔다.

## 1. public static final 필드 방식의 싱글턴

```java
public class Tom {
	public static final Tom INSTANCE = new Tom();
	private Tom() { ... }

	public void goToSchool() { ... }
}
```

- private 생성자는 Tom.INSTANCE를 초기화 할 때 딱 한번만 호출되기 때문에 protect나 public 생성자가 없으므로 **초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.**
  - 리플렉션을 이용해 private 생성자를 호출할 수 있지만, 두번째 객체가 생성되려 할 때 예외를 던지도록 방어하면된다.

### 장점

1. 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
   1. public static 필드가 `final`이라 절대 다른 객체를 참조할 수 없다.
2. 간결하다.

## 2. 정적 팩터리 방식의 싱글턴

```java
public class Tom {
	private static final Tom INSTANCE = new Tom();
	private Tom() { ... }
	public static Tom getInstance() { return INSTANCE; }

	public void goToSchool() { ... }
}
```

- `getInstance` 는 항상 같은 객체의 참조를 반환하므로 인스턴스가 하나뿐임이 보장된다.

### 장점

1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
   1. 유일한 인스턴스를 반환하는 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘기게 할 수 있다.
2. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.
   1. (ex) `Tom::getInstance` → `Supplier<Tom>` 으로 사용 가능하다.

위 장점들이 굳이 필요하지 않다면 public 필드 방식이 좋다.

## (1번, 2번방식 해당) 싱글턴 클래스 직렬화

단순히 `Serializable` 을 구현하는것이 아니라 `readResolve` 메서드를 제공해야 한다.

→ 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화 할 때 마다 새로운 인스턴스가 만들어진다.

```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
	// '진짜' Tom을 반환하고, 가짜 Tom은 가비지 컬렉터에 맡긴다.
	return INSTANCE;
}
```

## 3. 원소가 하나인 열거 타입 선언

```java
public enum Tom {
	INSTANCE;

	public void goToSchool() { ... }
}
```

### 장점

1. 다른 방식보다 간결하고, 추가 노력 없이 직렬화 할 수 있으며, 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다.
2. 조금 부자연스럽지만, 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

### 단점

1. 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

---

# 04. 인스턴스화를 막으려거든 private 생성자를 사용하라

가끔 **정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있을 것**이다.

하지만 다음과 같은 나름의 쓰임새가 있다.

1. 안티패턴이지만 유사한 그룹으로 묶인 유틸성 로직들을 모아놓을 수 있다.
   1. (ex) java.util.Arrays, java.util.Collections
2. final 클래스와 관련한 메서드들을 모아놓을 때 사용한다.

하지만 이런 **유틸리티성 클래스는 인스턴스로 만들어 쓸 이유가 없으나, 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다. 하지만 사용자는 해당 부분을 알 수 없기 때문에 의도치 않게 인스턴스화 할 수 있다.**

**이러한 부분을 막기 위해 private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**

→ 컴파일러가 기본 생성자를 만드는 경우는 명시된 생성자가 없을 때뿐이기 때문.

```java
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
	private UtilityClass() {
		throw new AssertionError();
	}

 ...
}
```

- 명시적 생성자가 `private` 라 클래스 바깥에서는 접근 불가능
  - 생성자를 호출하지 못하게 한 이유를 직관적으로 알 수 있게 적절한 주석을 달자.
- 상속이 불가능하게 하는 효과도 있다.

---

# 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.

많은 클래스는 하나 이상의 자원에 의존한다. \*\*

_(ex) 맞춤법 검사기 ↔ 사전_

**이런 클래스를 정적 유틸리티 클래스, 혹은 싱글턴으로 구현하는 경우가 흔한데 해당 방식은 그리 완벽하지는 않다.**

→ 맞춤법 검사기를 예로 들면 사전이 언어별로 따로 있을 수 있으며, 테스트용 사전이 필요할 수 있는데 사전 하나로 모든 경우에 대응할 수 없다.

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

클래스가 여러 자원 인스턴스를 지원하고, 클라이언트가 원하는 자원을 사용할 수 있도록 하는 방식은 **‘인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식' 이다. 이는 객체주입의 한 형태**이다.

→ 맞춤법 검사기를 생성할 때 의존 객체인 사전을 주입해주는 방식으로 사용한다.

# 의존성 주입의 예시

```java
public class SpellChecker {
	private final Lexicon dictionary;

	public SpellChecker(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}

	public boolean isValid(String word) { ... }
	public List<String> suggestions(String typo) { ... }
}
```

- 자원이 몇개든 의존 관계가 어떻든 상관없이 잘 작동한다.
- 불변을 보장한다.

# 의존성 주입의 응용

## 생성자에 자원 팩터리를 넘겨주는 방식

**자바 8의 `Supplier<T>` 인터페이스가 팩터리를 표현한 완벽한 예**다. 해당 인터페이스를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입(아이템 31) 을 사용해 팩터리 타입 매개변수를 제한해야 한다.

## 예시

클라이언트가 제공한 팩터리가 생성한 타일(Tile) 들로 구성된 모자이크를 만드는 메서드

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 수천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만든다.

→ 하지만 스프링같은 프레임워크를 사용하면 해결된다. 이러한 프레임 워크들은 의존 객체를 직접 주입하도록 설계된 API를 알맞게 응용해 사용하고 있기 때문이다.

# 정리

**클래스가 내부적으로 하나 이상의 자원에 의존하고 해당 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스 대신 의존 객체 주입 기법을 사용**하자.

의존 객체 주입 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

---

# 06. 불필요한 객체 생성을 피하라

객체 하나를 재사용 하는 편이 더 좋다. (특히 불변 객체는 언제든 재사용이 가능하다.)

# 재사용의 예시

## 1. String 객체 생성

`new String` 대신 String pool을 이용하도록 코드를 작성하라.

→ 이 내용은 [본인 블로그](https://mintheon.com/devlog/2021/12/05/String%EA%B3%BC-new-String%EC%9D%98-%EC%B0%A8%EC%9D%B4,-%EA%B7%B8%EB%A6%AC%EA%B3%A0-StringPool%EC%9D%B4%EB%9E%80/)에 자세히 설명해두었으니 참고바람.

## 2. 정적 팩터리 메서드 사용

boolean 생성자 대신 boolean 정적 팩터리 메서드를 사용하라.

`Boolean(String)` → `Boolean.valueOf(String)`

(boolean 생성자는 자바 9에서 deprecated 되었다.)

## 3. 캐싱 사용

생성 비용이 아주 비싼 객체들이 더러 있는데 이런 객체가 반복하여 필요하다면 캐싱하여 재사용하라.

### 예시

- **최적화 되지 않은 코드**

  ```java
  static boolean isRomanNumeral(String s) {
  	return s.matches("^...정규식...$");
  }
  ```

  - `String.matches` 는 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요할때 반복해서 사용하긴 적합하지 않다.
  - 정규표현식용 Pattern 인스턴스는 한번 쓰고 버려져 바로 가비지 컬렉션 대상이 된다.
  - Pattern은 인스턴스 생성 비용이 높다.

- **최적화 된 코드**

  ```java
  public class RomanNumerals {
  	private static final Pattern ROMAN = Pattern.compile("^...정규식...$");

  	static boolean isRomanNumeral(String s) {
  		return ROMAN.matcher(s).matches();
  	}
  }
  ```

  - 성능이 향상되었다.
  - 패턴 로직을 필드로 꺼내어 이름을 지었기 때문에 코드의 의미가 훨씬 잘 드러난다.

하지만 재사용을 함으로써 덜 명확하거나 안전하지 않은 상황도 있다.

# 객체 생성이 불필요한 예시

## 1. 어댑터

어댑터는 실제 작업은 뒷단 객체에 위임하고 자신은 제 2의 인터페이스 역할을 해주는 객체다.

결과적으로 뒷단 객체 외에는 관리할 상태가 없으므로 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분하다.

### 예시

**Map인터페이스의 KeySet 메서드**

- KeySet이 뷰 객체를 여러개 만들어도 상관은 없지만, 그럴 필요도 없고 이득도 없다.

## 2. 오토박싱

기본 타입과 그의 대응하는 박싱된 기본타입의 구분을 흐려주지만 성능에선 별로 좋지 않다.

### 예시

Long과 long

- Long을 long으로 바꿔주기만 해도 몇배의 성능 개선을 할 수 있다.
  - Long의 경우 래퍼객체라 Long으로 선언할 경우 인스턴스가 매번 만들어지게 된다.

**박싱된 기본타입보다는 기본타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**

# 정리

**무조건 ‘객체 생성은 비싸니 피해야한다’라는 뜻은 아니다. 아주 무거운 객체가 아니고서야 단순히 객체 생성을 피하고자 객체 풀(pool)을 만들지는 말자.**

최근의 JVM의 가비지 컬렉터는 상당히 잘 최적화 되어서 가벼운 객체를 다룰 땐 직접 만든 객체풀보다 훨씬 빠르다. 그러니 프로그램의 명확성, 간결성, 기능을 위해 객체를 추가로 생성하는 것이면 일반적으로 좋다.

→ DB 연결같은 경우 생성비용이 비싸 재사용하는게 낫지만, 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리며 성능을 떨어뜨린다.

**기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라.**

이번 장은 아이템 50번(새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라)과 대조적이지만 정확한 의미를 알자.

방어적 복사가 필요한 상황에서 객체를 재사용했을때의 피해 > 필요없는 객체를 반복 생성했을때의 피해

---

# 07. 다 쓴 객체 참조를 해제하라

C, C++과 다르게 자바는 가비지 컬렉터를 갖추어 자원관리에서 조금 더 편하다.

하지만 메모리관리에 더이상 신경쓰지 않아도 되는것은 아니다. 아래는 메모리 누수의 예시이다.

# 메모리 누수의 예시

## 1. 자기 메모리를 직접 관리하는 클래스

- **메모리 누수가 진행되는 스택 코드**

  ```java
  public class Stack {
      private Object[] elements;
      private int size = 0;
      private static final int DEFAULT_INITIAL_CAPACITY = 16;

      public Stack() {
          elements = new Object[DEFAULT_INITIAL_CAPACITY];
      }

      public void push(Object e) {
          ensureCapacity();
          elements[size++] = e;
      }

      public Object pop() {
          if (size == 0)
              throw new EmptyStackException();
          return elements[--size];
      }

      /**
       * 원소를 위한 공간을 적어도 하나 이상 확보한다.
       * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
       */
      private void ensureCapacity() {
          if (elements.length == size)
              elements = Arrays.copyOf(elements, 2 * size + 1);
      }
  ```

  - 스택에서 값을 꺼낼 때(pop()) 값만 꺼내고 있지 해당 배열 값은 계속 참조가 되고 있기 때문에 가비지 컬렉터가 회수하지 않는다.
    → elements 배열의 size보다 컸던 원소들

- **제대로 구현한 스택 코드**
  ```java
  ...
  public Object pop() {
  	if (size = 0)
  		throw new EmptyStackException();
    Object result = elements[--size];
  	elements[size] = null; // 다 쓴 참조 해제
  	return result;
  }
  ...
  ```

위와 같은 예시로 처리했지만 그렇다고 모든 객체 참조를 null 처리 할 필요는 없다. **객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.**

**다쓴 참조를 해제하는 최고의 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것**이다.

→ 변수의 범위를 최소가 되게 정의하자. (아이템 57)

위 Stack 클래스가 메모리 누수에 취약한 이유는 **자기 메모리를 직접 관리했기 때문**이다.

`elements` 배열로 저장소 풀을 만들어 원소들을 관리했는데 따로 배열의 활성 영역, 비활성 영역을 가비지 컬렉터는 알 길이 없다.

→ 가비지 컬렉터가 보기엔 참조되어 있는 객체들은 모두 유효한 객체이기 때문에 사용하지 않는 값은 null 처리해주어야 한다.

**자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.**

## 2. 캐시

객체 참조를 캐시에 넣고나서, 까먹어 버려 객체를 그대로 계속 놔두는 일을 접할 수 있다. 이경우 해결방법은 여러가지이다.

### 1. 키(key)를 참조하는 동안만 캐시가 필요한 경우 WeakHashMap 사용

대신 `WeakHashMap` 은 키를 참조하는 동안만 캐시가 필요한 상황에서만 유리하다.

### 2. 시간이 지날수록 가치를 떨어뜨리는 방식

캐시를 만들때 보통 캐시 엔트리의 유효기간을 정확히 정의하기 어렵기 때문에 시간이 지날수록 엔트리의 가치를 떨어뜨린다.

이런 경우 **쓰지않는 엔트리를 이따금 청소해줘야 하기 때문에(리텐션) 백그라운드 캐시를 활용하거나, 새 엔트리를 추가할때 부수작업으로 수행**한다.

→ LinkedHashMap은 `removeEldestEntry` 메서드를 사용하여 후자의 방식으로 처리한다.

## 3. 리스너 혹은 콜백

클라이언트가 콜백을 등록만하고 명확히 해지하지 않으면 콜백은 쌓여갈 것이다.

이때, 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해 간다.

→ (ex) WeakHashMap에 키로 저장하는 방법

# 정리

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. **이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요**하다.

---

# 08. finalizer와 cleaner 사용을 피하라

자바는 두가지 객체 소멸자를 제공하는데 **그중 `finalizer` 는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요**하다.

→ 자바 9에서는 deprecated 되었고, `cleaner` 를 대안으로 소개했지만.. 얘도 사실 똑같다..

**자바의 `finalizer`와 `cleaner`는 C++의 파괴자(destructor)와는 다른 개념**이다.

- **C++에서의 파괴자**
  - 생성자의 꼭 필요한 대척점으로 특정 객체와 관련된 자원을 회수하는 보편적인 방법
  - 비메모리 자원을 회수하는 용도
- **Java에서의 finalizer와 cleaner**
  - 즉시 수행된다는 보장이 없어, 제때 실행되어야 하는 작업은 절대 할 수 없다.(수행 시점 보장 X)
    - 자원 회수가 제멋대로 지연될 수 있다.
    - 해당 로직을 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸다.
  - 제대로 수행된다는 보장이 없다. (수행 여부 보장 X)
    - 제대로 수행되지 못한채 프로그램이 중단될 수도 있다.

**상태를 영구적으로 수정하는 작업에서는 절대 `finalizer` 나 `cleaner` 에 의존해선 안된다.**

→ (ex) 데이터베이스의 영구 락 해제를 얘네한테 맡겨놓으면 분산 시스템 전체가 서서히 멈출것이다…

또한 해당 소멸자를 보장해 준다는 모든 것을 믿지 말자. (보장해 준다는 메서드가 있었지만 심각한 결함때문에 수십년간 욕먹었다.)

# finalizer를 더 까봅시다.

## 1. 동작 예측 불가

**`finalizer` 동작 중 발생한 예외는 무시되며 처리할 작업이 남아도 그 순간 종료**된다.

잡지못한 예외때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있으며, 다른 스레드가 훼손된 해당 객체를 사용하려 한다면 어떻게 동작할지 예측할 수 없다.

보통의 경우엔 잡지 못한 예외가 스레드를 중단시키고 stack trace를 출력하겠지만, 해당 일이 `finalizer`에서 일어난다면 경고조차 출력하지 않는다. (그나마 `cleaner`는 자신의 스레드를 통제하기 때문에 이런 문제는 없다.)

## 2. 심각한 성능 문제

`finalizer` 가 가비지 컬렉터의 효율을 떨어뜨린다. (저자의 테스트로는 가비지 컬렉터보다 50배 느렸음)

## 3. 심각한 보안 문제

`finalizer` 공격에 노출되어 심각한 보안 문제가 일어날 수 있다.

→ 해당 공격은 생성자나 직렬화 과정에서 예외가 발생시 훼손된 객체에서 악위적인 하위 클래스의 finalizer가 수행될 수 있다.

**final이 아닌 클래스를 `finalizer` 공격으로부터 방어하려면 아무일도 하지 않는 finalize 메서드를 만들고 final로 선언하라.**

**종료해야 할 자원을 담고있는 객체의 클래스에서 위 두 친구 대신 `AutoCloseable` 을 구현해주고, 인스턴스를 다 쓰고나면 close 메서드를 호출하도록 한다. (try-with-resources를 사용하는 방법이 best)**

# 저 두놈은 어디에 쓰는걸까

## 1. 자원의 소유자가 close 메서드를 호출하지 않는것에 대비한 안전망 역할

즉시 혹은 끝까지 호출되리라는 보장은 없지만 **아예 안하는것보다는 낫기 때문**이다.

_(ex) FileInputStream, FileOutputStrema, ThreadPoolExecutor_

## 2. 네이티브 피어(native peer)와 연결된 객체

> 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체이다.

네이티브 피어는 자바객체가 아니라 가비지 컬렉터는 그 존재를 알지 못하고, 결국 회수하지 못한다. 이 경우 저 두 친구를 사용한다.

→ 대신, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당된다. **그냥 close 메서드를 사용하자…**

_(이후에 책 내에서 cleaner의 사용방법에 대한 내용이 있는데.. 굳이 필사할 정도는 아닌것 같아 pass함)_

# 정리

**cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용**하자.

물론 불확실성과 성능 저하에 주의해야한다.

---

# 09. try-finally 보다는 try-with-resources를 사용하라

자바 라이브러리에서는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다. 이러한 자원 닫기는 놓치기 쉬워 예측할 수 없는 성능문제로 이어지기도 한다.

_(ex) InputStream, OutputStream, java.sql.Connection 등_

# 기존의 try-finally 방식

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);

	try {
		OutputStream out = new FileOutputStream(dst);

		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		}
	} finally {
		out.close();
	}
}
```

- **예외는 try 블록과 finally 블록 모두에서 발생할 수 있다.**
  - 가령 두 곳에서 모두 예외가 발생한다면 이후에 생긴 finally 블록의 예외가 try에서 생긴 예외를 집어 삼켜 첫번째 예외에 관한 정보가 남지 않게 되어 실 시스템의 디버깅을 어렵게 한다.

# 자바 7 이후의 try-with-resources 방식

해당 방식 사용을 원한다면 `**AutoCloseable` 인터페이스를 구현해야 하는데, 이미 많은 클래스와 인터페이스가 구현하거나 확장해뒀다.\*\*

(혹시나 현재 읽고있는 사람도 차후에 닫아야 하는 자원을 뜻하는 클래스를 작성한다면 반드시 구현하자.)

```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
				OutputStrem out = new FileOutputStream(dst)) {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
	}
}
```

- 읽기 수월하고, 문제를 진단하기도 훨씬 좋다.
- 숨겨진 예외들도 버려지지 않고 스택 추적 내역에 ‘숨겨졌다(suppressed)’는 꼬리표를 달고 출력된다.
  - `Throwable` 에 추가된 `getSuppressed` 메서드를 이용하면 프로그램 코드에서 가져올 수도 있다.

**두 방식 다 `catch` 절을 쓸 수 있다. `catch` 절을 사용하면 `try` 문을 중첩하지 않고도 다수의 예외를 처리할 수 있다.**

# 정리

**꼭 회수해야 하는 자원을 다룰 때는 try-finally 대신 try-with-resources를 사용하자. (사실 무조건 사용하자)**
