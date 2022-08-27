# 정수 열거 패턴

```jsx
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
```

## 단점

- 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
  - 오렌지를 건네야 할 메서드에 사과를 보내고 ‘==’로 비교하더라도 컴파일러는 경고 메시지를 출력하지 않는다.
- 깨지기 쉽다.
  - 평범한 상수를 나열한 것 뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨져 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.
- 문자열로 출력하기 까다롭다.
  - 값을 출력하거나 디버거로 살펴보면 의미보단 단순 숫자로 보여 도움이 되지않는다.
- 같은 열거그룹에 속한 상수를 한바퀴 순회하는 방법도 마땅치 않다.
  - 열거그룹 내 상수가 몇개인지도 알 수 없다.

# 문자열 열거 패턴

정수대신 문자열 상수를 사용하는 변형 패턴.

## 단점

- 정수형 열거패턴보다 더 나쁘다.
- 상수의 의미를 출력할 수 있다는 점은 좋지만, 문자열 상수의 이름대신 문자열 값을 그대로 하드코딩하게 만들 수 있다.
  - 해당 문자열에 오타가 있어도 컴파일러는 확인을 못하니 런타임 버그가 생길 수 있다.
- 문자열 비교에 따른 성능 저하가 생긴다.

# 열거 타입(enum type)

```jsx
public enum Apple { FUJI, PIPPIN }
public enum Orange { NAVEL, TEMPLE }
```

## 특징

- 열거타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.
  - 열거타입은 밖에서 접근할 수 있는 생성자를 제공하지 않아 사실상 final이다. 그래서 인스턴스들은 딱 하나씩만 존재함이 보장된다.
- 완전한 형태의 클래스라 (단순한 정수값일 뿐인) 다른 언어의 열거타입보다 훨씬 강력하다.
- 컴파일타임 타입 안정성을 제공한다.
- 각자의 이름공간이 있어 이름이 같은 상수도 평화롭게 공존한다.
- 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지않아도 된다.
  - 공개되는 것이 필드의 이름뿐이라 상수값이 클라이언트로 컴파일 되어 각인되지 않는다.
- `toString` 메서드를 제공하여 출력하기 적합한 문자열을 내어준다.
- 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.

## 메서드나 필드 추가 예시

```jsx
public enum Planet {
	MERCURY(3.302e+23, 2.439e6),
	VENUS (4.869e+24, 6.052e6),
	...
	URANUS (8.683e+25, 2.556e7),
	NEPTUNE(1.024e+26, 2.47737)

	private final double mass; // 질량
	private final double radius; // 반지름
	private final double surfaceGravity; // 표면중력

	// 중력상수
	private static final double G = 6.67300E-11;

	// 생성자
	Planet(double mass, double radius) {
		this.mass = mass;
		this.radius = radius;
		surfaceGravity = G * mass / (radius * radius);
	}

	public double mass() { return mass; }
	public double radius() { return radius; }
	public double surfaceGravity() { return surfaceGravity; }

	public double surfaceWeight(double mass) {
		return mass * surfaceGravity;
	}
}

```

- 열거타입 상수 각각을 특정 데이터와 연결 지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장한다.
- 모든 필드는 final이어야 한다.

### 혹시 필드가 삭제된다면?

현재 코드상에서는 삭제함으로써 추가 문제는 없으며, 클라이언트 프로그램에서는 컴파일을 새로 할 경우, 제거된 상수를 참조하는 줄에서 컴파일 오류가 발생한다.

정수형처럼 아예 오류가 나지 않는 것 보다 훨씬 바람직한 대응이 가능하다.

### 메서드

- values
  - 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드
  ```jsx
  for (Planet p : Planet.values())
  	System.out.printf("%s에서의 무게는 %f이다. %n", p, p.surfaceWeight(mass));

  //MERCURY에서의 무게는 69.912739이다.
  //VENUS에서의 무게는 167.434436이다.
  //...
  ```
- valueOf
  - 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 메서드

## 상수마다 동작을 설정하는 예시

### 분기를 통한 동작 설정 (비추천)

```jsx
//비추천
public enum Operation {
	PLUS, MINUS, TIMES, DIVIDE;

	public double apply(double x, double y) {
		switch(this) {
			case PLUS: return x + y;
			case MINUS: return x - y;
			case TIMES: return x * y;
			case DIVIDE: return x / y;
		}

		throw new AssertionError("알 수 없는 연산: " + this);
	}
}
```

- 새로운 상수를 추가하면 해당 case 문을 추가해야 한다.
- 혹시 깜빡한다면 컴파일은 되지만, 연산 수행시 런타임 오류를 내며 프로그램이 종료된다.

### 상수별 메서드 구현을 통한 동작 설정 (세미 추천)

```jsx
public enum Operation {
	PLUS {public double apply(double x, double y) { return x + y;}},
	MINUS {public double apply(double x, double y) { return x - y;}},
	TIMES {public double apply(double x, double y) { return x * y;}},
	DIVIDE {public double apply(double x, double y) { return x / y;}};

	public abstract double apply(double x, double y);
}
```

- apply를 재정의 하지 않았을 경우 컴파일 오류로 알려준다.

### 상수별 클래스 몸체와 데이터를 사용한 동작 설정 (추천)

```jsx
public enum Operation {
	PLUS("+") {
    public double apply(double x, double y) { return x + y;}
  },
	MINUS("-") {
    public double apply(double x, double y) { return x - y;}
  },
	TIMES("*") {
    public double apply(double x, double y) { return x * y;}
  },
	DIVIDE("/") {
    public double apply(double x, double y) { return x / y;}
  };

  private final String symbol;

  Operation(String symbol) { this.symbol = symbol; }

  @Override
  public String toString() { return symbol; }

	public abstract double apply(double x, double y);
}
```

추가적으로 `toString` 을 오버라이드 해서 새로 설정했다면 toString이 반환하는 문자열을 해당 열거타입 상수로 변환해주는 `fromString` 메서드도 제공하는걸 고려해보자

### fromString 메서드 구현

```java
private static final Map<String, Operation? stringToEnum = Stream.of(values())
	.collect(toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol) {
	return Optional.ofNullable(stringToEnum.get(symbol));
}
```

- 열거타입 상수 생성 후 정적필드가 초기화 될 때 stringToEnum 맵에 Operation 상수가 추가된다.

## 상수끼리 코드를 공유하는 예시

위의 상수별 메서드 구현에서는 열거타입 상수끼리 코드를 공유하기 어렵다.

### 값에 따라 분기하여 코드를 공유하는 예시 (비추천)

```java
enum PayrollDay {
  MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

  private static final int MINS_PER_SHIFT = 8 * 60;

  int pay(int minutesWorked, int payRate) {
    int basePay = minutesWorked * payRate;

    int overtimePay;
    switch(this) {
      case SATURDAY: case SUNDAY:
        overtimePay = basePay / 2;
        break;
      default:
       overtimePay = minutesWorked <= MINS_PER_SHIFT ?
         0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
    }

    return basePay + overtimePay;
  }
}
```

- 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case 문을 넣어줘야 한다.
  - 넣는것을 깜빡할 경우 사단이 벌어진다. (돈문제이기 때문)

### 전략 열거 타입 패턴 사용 (추천)

```java
enum PayrollDay {
  MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
  SATURDAY(WEEKEND), SUNDAY(WEEKEND);

  private final PayType payType;

  PayrollDay(PayType payType) { this.payType = payType; }

  int pay(int minutesWorked, int payRate) {
    return payType.pay(minutesWorked, payRate);
  }

  // 전략 열거 타입
  enum PayType {
    WEEKDAY {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked <= MINS_PER_SHIFT
          ? 0
          : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
      },
      WEEKEND {
        int overtimePay(int minsWorked, int payRate) {
          return minsWorked * payRate / 2;
        }
      };

    abstract int overtimePay(int mins, int payRate);
    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overtimePay(minsWorked, payRate);
    }
  }
}
```

## 그래서 switch 문은..

열거 타입의 상수별 동작을 구현하는데 적합하진 않지만, **기존 열거 타입에 상수별 동작을 혼합해 넣을땐 switch 문이 좋은 선택이 될 수 있다.**

위에서 보여줬던 사칙 연산의 반대 연산을 반환하는 메서드가 필요하다고 할때, 이런 효과를 내주는 아래와 같은 정적 메서드를 만들 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/22b93e43-9dc7-449d-b720-d915c1320395/Untitled.png)

# 정리

- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거타입을 사용하자.
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.
- 하나의 메서드가 상수별로 다르게 동작해야 할 떄, switch문 대신 상수별 메서드 구현을 사용하자.
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거타입 패턴을 사용하.
