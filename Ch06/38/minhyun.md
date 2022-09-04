타입 안전 열거 패턴을 열거한 값들을 그대로 가져온 다음 값을 더 추가하여 다른 목적으로 쓸 수  있지만, 열거 타입은 그렇게 할 수 없다.

대부분 상황에서 열거 타입을 확장하는 건 좋지 않은 생각이다.

하지만 확장 열거 타입이 어울리는 곳이 딱 한가지 있는데, ‘연산코드’ 이다.

# 연산코드 예시

```java
public interface Operation {
    double apply(double x, double y);
}
```

## 기본 열거 타입

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

## 확장 열거 타입

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

- Operation 인터페이스를 사용하도록 작성되어 있기만 하면 어디든 쓸 수 있다.

# class 제한

## 1. class 리터럴

```java
<T extends Enum<T> & Operation>
```

- Class 객체가 열거 타입과 동시에 Operation의 하위 타입이어야 한다.

## 2. 한정적 와일드 카드 타입

```java
Collection<? extends Operation>
```

- Operation의 하위 타입이어야 한다.
- 대신 EnumSet과 EnumMap을 사용하지 못한다.

# 인터페이스를 통한 확장 열거타입 흉내의 단점

- 열거 타입끼리 구현을 상속할 수 없다.
    - 아무 상태에도 의존하지 않는 경우 디폴트 구현을 이용해 인터페이스에 추가한다.

## 자바라이브러리의 예시

- java.nio.file.LinkOption

# 정리

- 열거타입 자체는 확장이 불가하지만 인터페이스를 통해 같은 효과를 낼 수 있다.
- API가 인터페이스 기반으로 작성되었따면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.