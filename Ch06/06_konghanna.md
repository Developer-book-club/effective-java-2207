# 아이템 34: int 상수 대신 열거 타입을 사용하라

### 열거 타입(enum type)
- 열거 타입을 지원하기 전에는 정수 상수를 묶음으로 선언해 사용했음
- 열거 타입: 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입
  - 열거 타입 자체가 완전한 형태의 클래스 -> 상수 하나당 인스턴스를 하나씩 만들어 public static final 필드로 공개
  - 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이라고 볼 수 있음 -> 클라이언트가 인스턴스를 직접 생성/확장할 수 없으므로 인스턴스들은 딱 하나씩만 존재함 
    - 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 열거 타입은 싱글턴을 일반화한 형태라고 할 수 있음
  - 컴파일타임 타입 안전성 제공 
    - public enum Apple { FUJI, PRPPIN, GRANNY_SMITH } 와 같은 Apple 열거 타입을 매개변수로 받는 메서드를 선언했다면 건네받은 참조는 Apple의 세 가지 값 중 하나, 다른 타입의 값을 넘기려 하면 컴파일 오류 발생
  - 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 됨 
    - 필드의 이름만 공개되고 상수 값은 컴파일되어 각인되지 않음
    - 추가 or 제거된 상수를 참조하지 않는 클라이언트에는 영향이 없음
  - 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 함
    - 열거 타입은 불변이므로 모든 필드는 final이어야 함

### 열거 타입이 제공하는 메서드 
1.  values: 자신 안에 정의된 상수들의 값을 배열에 담아 반환
2.  toString: 상수 이름을 문자열로 반환함 -> println, printf로 출력하기에 적합함

### 상수별 메서드 구현
- 열거 타입에 추상 메서드를 선언하고, 각 상수(상수별 클래스 몸체)에서 그 상수에 맞게 메서드를 재정의하는 것 
```java
public enum Operation {
  PLUS {public double apply(double x, double y){return x + y;}}, 
  MINUS {public double apply(double x, double y){return x - y;}}, 
  TIMES {public double apply(double x, double y){return x * y;}}, 
  DIVIDE {public double apply(double x, double y){return x / y;}};

  public abstract double apply(double x, double y);
}
```
- apply 메서드가 상수 선언 바로 옆에 붙어 있어서 새로운 상수를 추가하면 메서드도 같이 재정의할 수 있게 됨
  - 또한, apply는 추상메서드 -> 재정의되지 않으면 컴파일 오류 발생

### 상수별 메서드 구현 + 상수별 데이터 
```java
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y){return x + y;}
    }, 
  MINUS("-") {
    public double apply(double x, double y){return x - y;}
    }, 
  TIMES("*") {
    public double apply(double x, double y){return x * y;}
    }, 
  DIVIDE(""/) {
    public double apply(double x, double y){return x / y;}
    };

  private final String symbol;
  Operation(String symbol) {this.symbol = symbol; }

  @Override public String toString() { return symbol; }

  public abstract double apply(double x, double y);
}

public static void main(Stirng[] args){
  double x = Double.parseDouble(args[0]); // 2
  double y = Double.parseDouble(args[1]); // 4 
  for (Operation op : Operation.values())
     System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x,y));
}
``` 
- 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf 메서드가 자동 생성됨
  - toString 메서드를 재정의할 때, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 것을 고려해볼 것 (Q) 216p. 

### 전략 열거 타입 패턴
- switch문이나 상수별 메서드를 구현하지 않고 새로운 상수를 추가할 때마다 전략을 선택하도록 함
```java
enum PayrollDay {
	MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);

	private final PayType payType;

	PayrollDay(PayType payType) {
		this.payType = payType;
	}

  int pay(int minutesWorked, int payRate){
    return payType.pay(minutesWorked, payRate); 
  }

	enum PayType {
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
      }
		} ,
		WEEKEND {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked * payRate / 2;
			}
		};

		abstract int overtimePay(int mins, int payRate);

		private static final int MINS_PER_SHIFT = 8 * 60; // 하루 8시간

		int pay(int minutesWorked, int payRate) {
			int basePay = minutesWorked * payRate;
			return basePay + overtimePay(minutesWorked, payRate);
		}
	}
}
``` 
### 열거 타입을 써야하는 경우
- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합일 경우 열거 타입을 사용할 것
- 열거 타입에 정의된 상수 개수가 영원히 고정불변일 필요는 없음 
  - 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었음

<br><br>

# 아이템 35: ordinal 메서드 대신 인스턴스 필드를 사용하라

```java
// ordinal 메서드 사용
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET; 

    public int numberOfMusicians() { return ordinal() + 1; }   
}

// 인스턴스 필드에 저장
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10), DOUBLE_QUARTET(8), TRIPLE_QUARTET(12); 

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }

    public int numberOfMusicians() { return numberOfMusicians; }
}
```
- ordinal 메서드: 해당 상수가 그 열거 타입에서 몇 번째 위치인지 반환하는 메서드
  - 상수 선언 순서를 바꾸는 순간 오동작함
  - 이미 사용 중인 정수와 값이 같은 상수는 추가할 수 없음 
    - 8중주(octet) 상수가 이미 있으므로 똑같이 8명이 연주하는 복4중주(double quartet)는 추가할 수 없음
  - 중간에 값을 비워둘 수도 없음

### 해결책: 인스턴스 필드에 저장
- 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하여 사용할 것

<br><br>

# 아이템 36: 비트 필드 대신 EnumSet을 사용하라

```java
// 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴
public class Text {
    public static final int STYLE_BOLD          = 1 << 0;  // 1
    public static final int STYLE_ITALIC        = 1 << 1;  // 2
    public static final int STYLE_UNDERLINE     = 1 << 2;  // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값
    public void applyStyles(int styles) { ... }
}
```

### 비트 필드
```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC); // 결과값: 3
```
- 비트별 OR를 사용하여 여러 상수를 모은 하나의 집합
  - 비트별 연산을 사용해 합집합, 교집합 같은 집합 연산을 효율적으로 수행 가능 
  - but 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 어려움
  - 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다로움
  - 최대 몇 비트가 필요한지 미리 예측하여 타입을 선택해야 함 

### EnumSet
```java
public class Text {
    public enum Style { BOLD, ITALIC, INDERLINE, STRIKETHROUGH }

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋음
    // 인터페이스로 받는 것이 좋은 습관
    public void applyStyles(Set<Style> styles) { ... }
}
```
- 비트 필드의 대안, 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해줌
- EnumSet의 장점 
  - Set 인터페이스를 구현, 타입 안전, 다른 Set 구현체와도 함께 사용 가능 
  - EnumSet의 내부는 비트 벡터로 구현되었기 때문에 원소가 64개 이하면 EnumSet 전체를 long 변수 하나로 표현 -> 비트 필드에 비견되는 성능을 보여줌
  - removeAll, retainAll과 같은 대량 작업은 비트 필드를 사용할 때와 동일하게 비트를 효율적으로 처리할 수 있는 산술 연산을 사용하여 구현, but 비트를 직접 다룰 때 발생하는 오류에서는 자유로움
- EnumSet의 단점 
  - 불변 EnumSet을 만들 수 없음 
    - 구글의 구아바 라이브러리 사용 (Collections.unmodifiableSet로 감싸서 사용)

<br><br>

# 아이템 37: ordinal 인덱싱 대신 EnumMap을 사용하라

### 인덱스를 얻는 방법1: ordinal 메서드 사용
- ordinal 메서드를 배열이나 리스트의 인덱스로 사용하는 것은 위험  
```java
class Plant {
  enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

  final String name;
  final LifeCycle lifeCycle;

  // +생성자, toString()
}

// 배열은 제네릭과 호환되지 않으니 비검사 형변환 필요
Set<Plant>[] plantByLifeCycle = 
    (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```
- 배열은 각 인덱스의 의미를 모르기 때문에 출력 결과에 직접 레이블을 달아야 함 (Q)
- 정확한 정수값을 사용한다는 것을 보증해야 함

### 인덱스를 얻는 방법2: EnumMap 사용
- 열거 타입을 키로 사용하도록 설계한 Map 구현체
- EnumMap은 내부에서 배열을 사용함
  - Map의 타입 안전성과 배열의 성능을 모두 얻어냄
```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
    new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}

for (Plant p : garden) {
    plantsByLifeCycle.get(p.lifeCycle).add(p);
}
System.out.println(plantsByLifeCycle);
```
- 안전하지 않은 형변환은 쓰지 않음
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열 제공 -> 출력 결과에 직접 레이블을 달 일이 없음
- 인덱스 계산하는 과정에서 오류가 발생하지 않음

+ cf. 스트림(Stream)을 사용해 맵을 관리하는 방법 (228p~)

<br><br>

# 아이템 38: 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
- 열거 타입을 확장하는 것은 지양해야 함
  - 열거 타입은 타입 안전 열거 패턴처럼 열거한 값들을 그대로 가져온 다음 값을 더 추가하여 다른 목적으로 쓰는 것이 불가능함 
  - 예외: 연산 코드를 구현할 때는 확장된 열거 타입이 어울림 -> 열거 타입이 인터페이스를 구현할 수 있다는 사실을 이용, 연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 됨(이때, 열거 타입은 그 인터페이스의 표준 구현체 역할을 함)

```java
// 인터페이스를 이용해 열거 타입을 확장 가능하게 만듦
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation{
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
    BasicOperation(String symbol) { this.symbol = symbol; }
    @Override public String toString() { return symbol; }
}
```
```java
// 인터페이스를 구현한 또 다른 열거 타입 정의
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
    ExtendedOperation(String symbol) { this.symbol = symbol; }
    @Override public String toString() { return symbol; }
}
```
- 열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스 Operation은 확장 가능
  - 인터페이스를 연산의 타입으로 사용
  - 인터페이스를 구현한 또 다른 열거 타입을 정의할 수 있음 
```java
//test 메서드에 ExtendedOperation의 class 리터럴을 넘김
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```
- `<T extends Enum<T> & Operation>` : Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻 

```java
//test 메서드에 Class 객체 대신 한정적 와일드카드 타입을 넘김
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```
### 인터페이스를 이용하는 방법의 문제점
- 열거 타입끼리는 구현을 상속할 수 없다는 점
- 열거 타입 간 공유하는 기능이 많으면 별도 클래스 또는 메서드로 분리하는 것이 좋음

<br><br>

# 아이템 39: 명명 패턴보다 애너테이션을 사용하라

- 전통적으로 도구나 프레임워크가 특별하게 다뤄야할 요소에는 딱 구분되는 명명 패턴을 적용해옴 
  - ex. JUnit 3까지 - 테스트 메서드의 이름을 test로 시작하게 함

### 명명 패턴의 단점
1. 오타가 실행여부에 영향을 줌 
    - ex. JUnit - tset~ 메서드는 그냥 지나침
2. 올바른 프로그램 요소에 사용되리라 보증할 수 없음
    - ex. 메서드가 아닌 클래스 이름을 Test~로 지을 경우 그냥 지나침
3. 프로그램 요소를 매개변수로 전달하는 방법이 없음 
    - ex. 특정 예외를 던져야 성공하는 테스트 - 예외 타입을 테스트에 매개변수로 전달해야 하는 경우 표현할 방법이 없음
- -> 마커 에너테이션을 도입하여 이러한 문제들을 해결

### 마커 애너테이션
```java
// 테스트 메서드임을 선언하는 애너테이션
// 매개변수 없는 정적 메서드 전용 (애너테이션 처리기를 통해 구현해야 함)
@Retention(RetentionPolicy.RUNTIME) // @Test가 런타임에도 유지되어야 한다는 뜻
@Target(ElementType.METHOD) // @Test가 반드시 메서드 선언에서만 사용돼야 한다는 뜻
public @interface Test {
}
```

- 메타애너테이션: 애너테이션 선언에 다는 애너테이션
  - @Retention, @Target
- javax.annotation.processing API 문서를 참고하여 적절한 애너테이션 처리기를 통해 제약을 구현

```java
public class Sample {
  @Test public static void m1() { }           

  public static void m2() { }

  @Test public static void m3() {
      throw new RuntimeException("실패");
  }

  public static void m4() { }

  @Test public void m5() { }  // 잘못 사용된 예: 정적 메서드가 아님 

  public static void m6() { }

  @Test public static void m7(){
      throw new RuntimeException("실패");
  }

  public static void m8() { }
}
```
- @Test를 붙이지 않는 메서드는 테스트 도구가 무시
- m1: 성공 / m3, m7: 실패
- @Test 애너테이션: Sample 클래스의 의미에 직접적인 영향을 주지는 않으나 이 애너테이션에 관심 있는 프로그램에서 특별하게 처리할 기회를 줌

### 매개변수를 받는 애너테이션 
```java
// 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD) 
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

public class Sample2 {
  @ExceptionTest(ArithmeticException.class)
  public static void m1() { // 예외 던지기 성공
      int i = 0;
      i = i / i;
   }           
  @ExceptionTest(ArithmeticException.class)
  public static void m2() { // 실패- 다른 예외 발생
     int[] a = new int[0];
     int i = a[1];
  }

  @ExceptionTest(ArithmeticException.class)
  public static void m3() { } // 실패- 예외 발생 x
} 

```
- Class<? extends Throwable>: Throwable을 확장한 클래스의 Class 객체 (모든 예외 타입을 다 수용함) 

+ 242p~ 
### 배열 매개변수를 받는 애너테이션

### 반복 가능 애너테이션

<br><br>

# 아이템 40: @Override 애너테이션을 일관되게 사용하라

### @Override
- 상위 타입의 메서드를 재정의했다는 뜻 
- 메서드 선언에만 달 수 있음
- 일관되게 사용하면 여러 버그들을 예방해줌 

```java
public class Bigram {
    private final char first;
    private final char second;
(...)
    // equals를 재정의(overriding)가 아닌 다중정의(overloading) 함 
    // Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야 함 
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    //hashCode도 함께 재정의
    public int hashCode() {
        return 31 * first * second;
    }
(...)
}
```
```java
    // @Override 애너테이션을 달면 컴파일 오류 발생
    @Override
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    
    //수정
    @Override
    public boolean equals(object o) {
      if (!(o instanceof Bigram)) return false;
      Bigram b = (Bigram) o;
      return b.first == first && b.second == second;
    }
```
- 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달 것
  - 예외: 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때 (구현되지 않은 추상 메서드가 있으면 컴파일러가 알아서 알려줌)
  - 인터페이스의 디폴트 메서드를 재정의할 때도 @Override를 달 것 
  - 추상 클래스, 인터페이스에서는 상위 클래스(or 인터페이스)의 메서드를 재정의하는 모든 메서드에 @Override를 달 것 

<br><br>

# 아이템 41: 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

### 마커 인터페이스(marker interface) 
- 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 갖게 될 특정 속성을 표현해주는 인터페이스
   - ex. Serializable 인터페이스: 자신을 구현한 클래스의 인스턴스가 ObjectOutputStream을 통해 쓸(write) 수 있음을(=직렬화할 수 있음을) 알려줌 

### 마커 인터페이스의 장점
- 클래스의 인스턴스를 구분하는 타입으로 쓸 수 있음 
  - 마커 인터페이스는 타입이기 때문에 컴파일타임에 오류가 발생함
  - 마커 애너테이션은 타입이 아니기 때문에 런타임이 되어서야 오류가 발생함  
- 적용 대상을 더 정밀하게 지정할 수 있음
  - 마커 애너테이션: 적용 대상(@Target)을 elementType.TYPE으로 선언했으면 모든 타입(클래스, 인터페이스, 열거 타입, 애너테이션)에 적용 가능
  - 마커 인터페이스: 마킹하고 싶은 클래스(or 인터페이스)에서만 인터페이스를 구현(or 확장)하면 됨 

### 마커 애너테이션의 장점
- 애너테이션 시스템의 지원을 받음

### 마커 인터페이스 or 마커 애너테이션
- 마커 애너테이션을 써야할 때 
  - 클래스, 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야 할 때
- 마커 인터페이스를 써야할 때
  - 마킹이 된 객체(클래스, 인터페이스)를 매개변수로 받는 메서드를 작성할 경우 -> 이런 메서드가 필요하지 않으면 마커 애너테이션이 더 나은 선택일 것
