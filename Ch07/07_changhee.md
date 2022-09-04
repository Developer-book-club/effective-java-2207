# 6. 열거 타입과 애너테이션
## 아이템 34. int 상수 대신 열거 타입을 사용하라
* 열거 타입 : 일정 개수의 상수값을 정의하고, 그 외의 값은 허용하지 않는 타입
* 열거 타입 전에는 정수 열거 패턴으로 대신해서 사용
```
    public static final int APPLE_FUJI = 0;
    public static final int APPLE_PIPPIN = 1;
    public static final int APPLE_GRANNY_SMITH = 2;
```
* 정수 열거 패턴의 단점
    1. 평범한 상수를 나열한 것이기 때문에 타입 안전 보장 불가능
    2. 표현력도 좋지 않다.
    3. 문자열 표현이 까다로움
    4. 컴파일 시 값이 클라이언트 파일에 그대로 새겨짐
* 열거 타입(enum type)
    1. 자체는 클래스이며 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개(밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final)
    ```
        public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
        public enum Orange { NAVEL, TEMPLE, BLOOD }
    ```
    3. 상수 인스턴스가 하나씩임을 보장
    4. 컴파일 타임 타입 안정성 제공(다른 타입과 비교하려고하면 컴파일 에러 발생)
    5. 공개된것이 필드 이름뿐이라 정수 열거 패턴과 달리 새로운 상수가 추가되거나 순서가 바뀌어도 클라이언트를 다시 컴파일할 필요가 없음)
    6. toString 메서드로 출력하기에 적합한 문자열 표현 가능
    7. 임의의 메서드나 필드 추가, 인터페이스 구현도 가능(단순 상수 모음일 뿐만 아니라 실제로는 클래스 이므로 고차원의 추상 개념 표현 가능)
    ```
        public enum Planet {
            MERCURY(3.302e+23, 2.439e6),
            VENUS(4.869e+24, 6.052e6),
            EARTH(5.975e+24, 6.378e6);
            MARS(6.419e+23, 3.393e6);
            
            private final double mass;          
            private final double radius;        
            private final double surfaceGravity; 
        
            private static final double G = 6.67300E-11;
        
            Planet(double mass, double radius) {
                this.mass = mass;
                this.radius = radius;
                this.surfaceGravity = G * mass / (radius * radius);
            }
        
            public double mass() { return mass; }
            public double radius() { return radius; }
            public double surfaceGravity() { return surfaceGravity; }
        
            public double surfaceWeight(double mass) {
                return mass * surfaceGravity;
            }
        }
    ```
    9. 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장(모든 필드는 final이어야 함)
    10. 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values 제공
    11. 상수마다 분기하여 수행하는 메서드가 필요하면, case 문 대신 상수 별 메서드 구현 사용
    12. 상수별 메서드와 상수별 데이터 결합 가능
    ```
        public enum Operation {
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
                
                public abstract double apply(double x, double y);
            };

    ```
    13. 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메소드 존재
    14. toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드 제공 고려 필요
    15. 전략 열거 타입 패턴 : 상수별 메서드 구현 시 상수 끼리 코드를 공유할 필요가 있다면 전략 열거 타입을 분리하여 사용
    ```
        public enum PayrollDay {
            MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY),
            WEDNESDAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY),
            FRIDAY(PayType.WEEKDAY),
            SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
        
            private final PayType payType;
        
            PayrollDay(PayType payType) {
                this.payType = payType;
            }
        
            double pay(double hoursWorked, double payRate) {
                return payType.pay(hoursWorked, payRate);
            }
        
            // 전략 열거 타입
            private enum PayType {
                WEEKDAY {
                    double overtimePay(double hours, double payRate) {
                        return hours <= HOURS_PER_SHIFT ? 0 : (hours - HOURS_PER_SHIFT) * payRate / 2;
                    }
                },
                WEEKEND {
                    double overtimePay(double hours, double payRate) {
                        return hours * payRate / 2;
                    }
                };
                private static final int HOURS_PER_SHIFT = 8;
        
                abstract double overtimePay(double hours, double payRate);
        
                double pay(double hoursWorked, double payRate) {
                    double basePay = hoursWorked * payRate;
                    return basePay + overtimePay(hoursWorked, payRate);
                }
            }
        }
    ```
    15. 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입 사용
## 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라
* 열거 타입 상수가 하나의 정수값에 대응하기 때문에, 위치를 나타내는 ordinal이라는 메서드 존재
```
    public enum Ensemble {
        SOLO, DUET, TRIO, OCTET, DOUBLE_QUARTET;
        
        public int numberOfMusicians() { return ordinal() + 1; }
    }
    기능 목적으로 ordinal을 사용하고 싶을 수는 있지만, 코드가 오동작 할 수 있다.
```
* ordinal은 위치를 나타내는 용도 외의 다른 목적(정수값을 나타내는)이라면 대신 인스턴스 필드를 사용하는 것이 좋다.
```
    public enum Ensemble {
        SOLO(1), DUET(2), TRIO(3), OCTET(8), DOUBLE_QUARTET(8);
        
        private final int numberOfMusicians;
        Ensenble(int size) { this.numberOfMusicians = size; }
        public int numberOfMusicians() { return numberOfMusicians; }
    }
```
## 아이템 36. 비트 필드 대신 EnumSet을 사용하라
* 비트 필드 : 열거한 값들이 주로 집합으로 사용될 경우 정수 열거 패턴을 주로 사용했으며 이때 비트별 OR를 사용해 여러 상수를 하나의 집합(옵션)으로 표현
```
    public class Text {
        public static final int STYLE_BOLD = 1 << 0;    // 1
        public static final int STYLE_ITALIC = 1 << 1;    // 2
        public static final int STYLE_UNDERLINE = 1 << 2;    // 4
        public static final int STYLE_STRIKETHROUGH = 1 << 3;    // 8
        
        public void applyStyles(int styles) {
            ~~
        }
    }
    
    text.applyStype(STYLE_BOLD | STYLE_ITALIC);
```
* 비트 필드의 장점 : 합집합과 교집합 같은 집합 연산에 효율
* 비트 필드의 단점 : 정수 열거 상수의 단점을 그대로 지님
    1. 해석이 어려움
    2. 원소 순회가 어려움
    3. 필요한 비트에 따라 적절한 타입(int나 long)을 미리 예측해서 선택해야함
* EnumSet 클래스는 집합을 효과적으로 표현해줄 뿐만 아니라 Set 인터페이스를 완벽히 구현하여 타입 안전하다. 그리고 내부는 비트 벡터로 되어있어 성능도 좋다.
```
    public class Text {
        public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
        
        public void applyStyles(Set<Style> styles) {
            ~~
        }
    }
    
    text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
## 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라
* 배열의 인덱싱 등의 목적으로 ordinal을 사용할 경우 타입 안정성을 보장할 수 없고, ArrayIndexOutOfBoundsException 등의 예외 발생 가능
* EnumMap : 열거 타입을 키로 사용하는 Map 구현체 : 내부에서 배열을 사용하여 성능과 타입 안정성 둘다 보장
```
    Map<열거 타입 ,Set<열거형 클래스>> enumMap = new EnumMap<>(열거형 클래스.class)
    enumMap.get(열거 타입)으로 해당 열거 타입의 Set를 얻을 수 있음
```
## 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라(미완)
* 열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수
## 아이템 39. 명명 패턴보다 애너테이션을 사용하라(미완)
* 명명 패턴 : 도구나 프레임워크가 특별히 다뤄야할 요소에 적용(ex. junit3 까지의 테스트 메서드는 test로 시작)
* 명명 패턴의 단점
    1. 오타 인식 불가능(tset로 시작하면 junit에서 테스트 수행 안함)
    2. 올바른 프로그램 요소에서만 사용되리라는 보증 불가능(클래스 이름을 Test로 시작해도 junit에서 테스트 실행 안함)
    3. 프로그램 요소를 매개변수로 전달할 방법이 없다.(예외를 나타내는 이름을 컴파일러가 인지 불가능)
 * 
## 아이템 40. @Override 애너테이션을 일관되게 사용하라
* 재정의하는 메소드라면 @Override 애너테이션으로 의도를 명확히 명시
* 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션 필요(추상 클래스 예외)
## 아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라
* 마커 인터페이스 : 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스(ex. Serializable : 자신이 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 직렬화할 수 있다고 알려줌)
* 마커 애너테이션보다 마커 인터페이스가 나은점
    1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 불가능함(런타임에 오류 발견 가능)
        * Serializable은 이러한 이점을 살리지 못함
    2. 적용 대상을 더 정밀하게 지정 가능(애너테이션은 클래스, 인터페이스, 열거 타입, 애너너테이션에만 적용 가능하여 부착할 수 있는 타입을 더 세밀하게 제한 불가)
* 마커 애너테이션이 마커 인터페이스보다 나은점
    1. 거대한 애너테이션 시스템의 지원
* 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야할 때는 마커 애너테이션을 쓸수밖에 없다.
* 마킹된 객체를 매개변수로 받는 메서드를 작성할 일이 있으면 마커 인터페이스를 써야한다.
* 애너테이션을 활발히 활용하는 프레임워크에서 사용하려는 마커라면 일관성을 위해 마커 애너테이션을 쓰는 것이 좋다.