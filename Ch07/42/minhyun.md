# 함수타입 표현 일대기

## 자바 극초반

- 추상 메서드를 하나만 담은 인터페이스(드물게 추상클래스) 사용
  - 이러한 인터페이스의 인스턴스를 함수 객체(function object)라고 했다.

## JDK 1.1 이후 (1997)

### 익명 클래스 사용

```
Collections.sort(words, new Comparator<String>() {
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        });
```

- `Comparator` 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며 구체적인 전략을 익명 클래스로 구현.
- 구현하기 위한 코드가 너무 길기 때문에 함수형 프로그래밍에 적합하지 않았다.

## JAVA 8 이후

### 람다식

```
Collections.sort(words,
	(s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- 반환값의 타입은 컴파일러가 문맥을 살펴 타입을 추론해주어 적어주지 않아도 된다.
  - 타입 추론 규칙은 매우 복잡하기 때문에, 상황에 따라 우리가 직접 명시해 주어야 할 때도 있다.
  - **타입을 명시해야 코드가 더 명확할 때만 제외하곤, 람다의 모든 매개변수 타입은 생략하자.**

### 비교자 생성 메서드 사용 & sort 메서드 사용

```java
// 비교자 생성 메서드 사용
Collections.sort(words, comparingInt(String::length));

// 추가적으로 List 인터페이스에 추가된 sort 메서드를 사용
words.sort(comparingInt(String::length));
```

# item 34의 열거타입 리팩터링

- 기존 apply 메서드를 중복적으로 각 인스턴스마다 구현해 냈던 방식을 람다를 사용하면 더 간단하게 표현이 가능하다.

```java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
```

- [DoubleBynaryOperator](https://runebook.dev/ko/docs/openjdk/java.base/java/util/function/doublebinaryoperator)는 double값 피연산자에 대한 연산과 값을 생성하는 함수형 인터페이스이다.

**람다 사용시 클래스 몸체는 필요없다 생각할 수 있지만, 람다는 이름이 없고 문서화도 못한다.**

**즉, 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.**

람다는 한줄일 때 가장 좋고, 길어도 세줄 안에 끝내는게 좋다.

→ 열거타입 생성자에 넘겨지는 인수타입들은 컴파일타임에 추론되기 때문에, 열거 타입 생성자 안의 람다는 런타임에 만들어지는 열거 타입의 인스턴스 멤버에 접근할 수 없다. **상수별 동작을 3줄 내로 구현하기 힘들거나 인스턴스 필드나 메서드를 사용해야만 하는 상황이면 상수별 클래스 몸체를 사용해야 한다.**

# 람다가 쓰이지 못하는 곳

## 1. 람다는 함수형 인터페이스에서만 쓰일 수 있다.

- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없어 익명 클래스를 써야한다.
- 추상 메서드가 여러개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 써야한다.

## 2. 람다는 자신을 참조할 수 없다.

- 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.
  - 반면 익명 클래스의 this는 익명 클래스 인스턴스 자신을 가리킨다.

## 3. 직렬화를 삼가야 한다

- 람다도 익명 클래스처럼 직렬화 형태가 구현별로(ex. 가상머신별) 다를 수 있다.
- 직렬화해야만 하는 함수 객체가 있다면 (가령 Comparator), private 정적 중첩 클래스(아이템 24)의 인스턴스를 사용하자.

# 정리

- 람다는 자바 8 이후에 도입되었다.
- 익명 클래스는 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용하라.
- 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있다.
