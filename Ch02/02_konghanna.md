
# 아이템 1. 생성자 대신 정적 팩터리 메소드를 고려하라

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
} 
```
- public 생성자: 클래스의 인스턴스를 얻는 전통적인 수단 
- public 생성자보다 나은 정적 팩터리 메서드의 장점 有

## 장점 1: 이름을 가질 수 있다
- 생성자와는 달리 이름을 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있음
- 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같은 경우, 생성자를 정적 팩터리 메소드로 바꿔주고 적절한 이름을 지어줄 것 

## 장점 2: 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다
- 불변 클래스: 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용 가능
- ex) Boolean.valueOf(boolean) 메소드
- 인스턴스 통제 클래스: 반복되는 요청에 같은 객체를 반환하는 식으로 인스턴스 통제 가능

## 장점 3: 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
- 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 유연성을 제공함
- API 만들 때 이러한 특징을 응용하면 구현 클래스를 공개하지 않고도 객체를 반환할 수 있음
- 인터페이스를 정적 팩터리 메소드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크의 핵심 기술

## 장점 4: 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다
- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체도 반환할 수 있음
- ex) EnumSet 클래스
    - 반환하는 객체 타입은 원소의 개수에 따라 다른 인스턴스를 반환함

## 장점 5: 정적 팩터리 메소드를 작정하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다 
- 이러한 유연함은 서비스 제공자 프레임워크를 만드는 근간이 됨 ex) JDBC
- cf) 서비스 제공자 프레임워크 
    - 보완 필요

## 단점 1: 상속 시 public 또는 protected 생성자가 필요하기 때문에 정적 팩터리 메소드만 제공하면 하위 클래스를 만들 수 없다
-  컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없음 
- 불변 타입을 만들기 위한 제약이자 상속보다 컴포지션을 사용하도록 유도하기 때문에 장점이 될 수도 있음

## 단점 2: 정적 팩터리 메소드를 찾기가 어렵다
- 생성자와 달리 API 설명에 드러나지 않음
- 메소드 이름을 규약을 따라 짓는 방법으로 문제를 완화할 수 있음

<br><br>

# 아이템 2: 생성자에 매개변수가 많다면 빌더 사용을 고려하라

- 정적 팩터리 메소드와 생성자 모두 매개변수가 많으면 불편해짐

## 해결책 1: 점층적 생성자 패턴
```java
NutritionFacts cocaCola =
new NutritionFacts(240, 8, 100, 0, 35, 27);
```
- 필수 매개변수만 받는 생성자부터 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식
- 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출
- 매개변수 개수가 많아지면 코드를 작성하거나 읽기 어려움

## 해결책 2: 자바빈즈 패턴
```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```
- 매개변수가 없는 생성자로 객체를 만든 후 세터(setter) 메소드를 호출해 원하는 매개변수의 값 설정
- 객체 하나를 만들려면 메소드 여러 개를 호출해야 하고 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 됨
    - 클래스를 불변으로 만들 수 없고 스레드 안전성을 얻기 위한 추가 작업이 필요함 (Q)
    - 이러한 단점을 완화하고자 생성이 끝난 객체를 freeze할 수 있음 (Q)

## 해결책 3: 빌더 패턴
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```
- 점층적 생성자 패턴의 안전성+자바빈즈 패턴의 가독성을 겸비한 대안
1) 필요한 객체를 직접 만드는 대신 필수 매개변수만으로 생성자나 정적 팩터리 메소드를 호출해 빌더 객체를 얻음 
2) 빌더 객체가 제공하는 세터 메소드로 원하는 선택 매개변수를 설정함
3) 매개변수가 없는 build 메소드를 호출해 객체를 얻음(보통은 불변)
- 보통 빌더는 클래스 안에 정적 멤버 클래스로 따로 만들어 둠

### 빌더 패턴의 특징
- 파이썬과 스칼라가 제공하는 명명된 선택적 매개변수(Named Optional Parameter)를 모방함
- 빌더 하나로 다양한 객체를 유연하게 생성할 수 있음
- 빌더를 따로 먼저 만들어야 하기 때문에 성능 문제가 생길 수 있고 코드가 장황하기 때문에 매개변수가 많은 경우에 사용하는 것이 좋음
- 유효성 검사
    - 빌더의 생성자나 메소드에서 검사 
    - build 메소드가 호출하는 생성자에서 불변식 검사
    - 빌더로부터 매개변수를 복사한 후 해당 객체 필드 검사<br>
->  IllegalArgumentException를 던져 어떤 매개변수가 잘못되었는지 알려줌
```java
public abstract class Pizza {

}
```
```java
public class NyPizza extends Pizza {

}
```
```java
public class Calzone extends Pizza {

}
```
- 계층적으로 설계된 클래스와 함께 쓰기 좋음<br>
  : 추상 클래스에 추상 빌더를 만들고 하위 클래스에서 추상 빌더를 상속받아 각각의 하위 클래스에서 사용하는 빌더를 만들 수 있음 

```java
NyPizza nyPizza = new NyPizza.Builder(SMALL)
    .addTopping(SAUSAGE)
    .addTopping(ONION)
    .build();

Calzone calzone = new Calzone.Builder()
    .addTopping(HAM)
    .sauceInde()
    .build();
```
- 빌더를 이용하면 가변인수(varargs) 매개변수를 여러 개 사용할 수 있음
- 여러 메소드를 호출하여 넘겨받은 매개변수들을 하나의 필드로 모을 수도 있음 

<br><br>

# 아이템 3: private 생성자나 enum 타입으로 싱글턴임을 보증하라
- 싱글턴(singleton): 인스턴스를 오직 하나만 생성할 수 있는 클래스
    - ex) 함수 같은 무상태(stateless) 객체, 설계상 유일해야 하는 시스템 컴포넌트
    - 싱글턴을 사용하는 클라이언트는 테스트하기가 어려움<br>
      : 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 mock 구현으로 대체할 수 없기 때문 
    - 싱글턴을 만드는 방식은 두 가지가 있는데, 모두 생성자는 private로 감춰두고 public static 멤버를 하나 마련해 둬서 유일한 인스턴스를 제공함

## 싱글턴을 만드는 방식 1: final 필드 방식

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    
    public void leaveTheBuilding(){}
}
```
- private 생성자는 Elvis.INSTANCE를 초기화할 때 한 번만 호출됨
- 예외: 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있음 -> 두 번째 객체가 생성되려 하면 예외를 던지게 생성자를 수정하면 됨

## 싱글턴을 만드는 방식 2: 정적 팩토리 메소드 방식

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding(){}
}
```
- getInstance는 항상 같은 객체의 참조를 반환함

## final 필드 방식 vs 정적 팩토리 메소드 방식

1. final 필드 방식
   -  해당 클래스가 싱글턴임이 API에 명백히 드러나고 간결함<br>
    : public static + final : 다른 객체를 참조할 수 X
2. 정적 팩토리 메소드 방식 (Q)
   - API를 변경하지 않고도 싱글턴이 아닌 클래스로 바꿀 수 있음
     : 스레드별로 다른 인스턴스를 넘겨주게 할 수 있음 
   - 원한다면 Generic 싱글턴 팩터리로 만들 수 있음
   - 정적 팩터리 메소드의 레퍼런스를 `Supplier<Elvis>`로 사용할 수 있음  

## 직렬화 (Serialization)

- 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족함, 모든 인스턴스 필드를 일시적(transisent)이라고 선언하고 readResolve 메소드를 제공해야 함<br>
-> 이렇게 하지 않으면 직렬화된 인스턴스를 역지렬화할 때마다 새로운 인스턴스가 만들어짐

```java
    //싱글턴임을 보장해주는 readResolve 메소드
    private Object readResolve() {
       // 진짜 Elvis 반환, 가짜 Elvis는 가비지 컬렉터에 맡김
        return INSTANCE;
    }

```
## 싱글턴을 만드는 방식 3: Enum 타입 선언 방식

```java
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding(){}
}
```
- 쉽게 직렬화 가능, 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 막아줌
- 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법
- 싱글턴 클래스가 Enum 외의 클래스를 상속해야 한다면 사용할 수 없음 (열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있음)
<br><br>

# 아이템 4: 인스턴스화를 막으려거든 private 생성자를 사용하라

- 정적 멤버만 담은 유틸리티 클래스에 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자(매개변수를 받지 않는 public 생성자)를 만들어줌 
- 정적 멤버가 담겨 있는 클래스를 추상 클래스로 만들어도 인스턴스화를 막을 수 없음 <br>
  : 하위 클래스를 만들고 상속 받아서 인스턴스화하면 되기 때문<br>

## 클래스의 인스턴스화를 막는 방법: private 생성자 추가 

```java
public class UtilityClass {
  //기본 생성자가 만들어지는 것을 막음 -> 인스턴스화 방지
  private UtilityClass(){
    throw new AssertionError();
    /* 
    AssertionError() 
     : 클래스 안에서 실수로라도 생성자를 호출하지 않도록 하기 위함
     */
  }
}
```
- private 생성자의 효과 <br>
  1) 클래스가 인스턴스화되는 것을 막아줌
  2) 상속을 불가능하게 함 <br>
      : 상속한 경우 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데 private 생성자는 호출이 막혔기 때문에 상속 불가능

<br><br>

# 아이템 5: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 많은 클래스가 하나 이상의 자원에 의존함
  ex) 맞춤법 검사기(SpellChecker)는 사전(Dictionary)에 의존

## SpellChecker를 구현하는 방식

### 정적 유틸리티 클래스

```java
// 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다
public class SpellChecker {

    private static final Lexicon dictionary = new KoreanDicationary(); //한국어 사전 Q.Lexicon?

    private SpellChecker() {} //객체 생성 방지

    public static boolean isValid(String word) {} 
    public static List<String> suggestion(String typo) {} 
}
```

### 싱글턴

```java
// 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
public class SpellChecker {

    private final Lexicon dictionary = new KoreanDicationary(); //한국어 사전

    private SpellChecker() {}

    public static SpellChecker INSTANCE = new SpellChecker() {};

    public boolean isValid(String word) {}
    public List<String> suggestions(String typo) {}
}
```

- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않음 
- 의존 객체 주입 패턴
  - 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식  
  - ex) SpellChecker를 생성할 때 의존 객체인 dictionary를 주입해줌 

### 의존 객체 주입 패턴

```java
public class SpellChecker {

    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {}
    public List<String> suggestions(String typo) {}
}
```

- 의존 객체 주입은 생성자, 정적 팩터리, 빌더에서 적용할 수 있음 
- 의존 객체 주입 패턴의 변형 <br>
  : 생성자에 자원의 팩터리를 넘겨주는 방식 
  - 팩터리: 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체 (의존 객체를 생성)
  - ex) `Supplier<T>` 인터페이스 <br>
    - 팩터리로 사용할 수 있음
    - `Supplier<T>`를 인자로 받는 메서드는 일반적으로 한정적 와일드카드 타입(bounded wildcard type)을 사용해 팩터리의 타입을 제한해야 함 <br>
    -> 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있음  

### 팩터리가 생성한 타일들로 모자이크를 만드는 메서드

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

### 핵심 정리
- 클래스가 내부적으로 하나 이상의 자원에 의존하고 그 자원이 클래스 동작에 영향을 준다면 정적 유틸리티 클래스, 싱글턴은 사용하지 않는 것이 좋음 
- 이 자원들을 클래스가 직접 만들게 해서도 안 됨
- 의존 객체 주입 방식을 통해 필요한 자원 or 그 자원을 만들어주는 팩터리를 생성자에 넘겨주어야 함 -> 클래스의 유연성, 재사용성, 테스트 용이성 개선

<br><br>

# 아이템 6: 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 것이 나음, 특히 불변 객체는 언제든 재사용할 수 있음

## 하나의 문자열 인스턴스 사용
```java
String s = new String("bikini");
```
- 실행될 때마다 String 인스턴스가 새로 만들어짐

```java
String s = "bikini";
```
- 하나의 String 인스턴스를 사용, 같은 자바 가상 머신 안에 동일한 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함  

## 정적 팩터리 메소드 사용
- Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩터리 메소드를 사용하는 것이 좋음 
  - Boolean(String): 자바 9에서 deprecated 됨
- 생성자는 호출될 때마다 새로운 객체를 만들지만 팩터리 메소드는 그렇지 않음

## 비싼 객체는 캐싱하여 재사용
- 생성 비용이 비싼 객체는 캐싱하여 재사용하는 것이 좋음

```java
    static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"+"(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
```
- `String.matches`는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 메소드지만 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않음
  - 정규표현식용 Pattern 객체: 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 인스턴스 생성 비용이 높음  

```java
public class RomanNumerals {

    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})"+"(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }

}
```
- 정규표현식을 표현하는 Pattern 인스턴스를 클래스 초기화 가정에서 직접 생성해 캐싱해두고, 나중에 isRomanNumeral 메소드가 호출될 때마다 이 인스턴스를 재사용
- but, isRomanNumeral 메소드가 호출되지 않는다면 ROMAN 필드는 쓸데없이 초기화된 것이 됨. 
  - 지연 초기화(lazy initialization)로 불필요한 초기화를 없앨 수는 있지만 권하지는 않음 -> 코드를 복잡하게 만들고 성능은 크게 개선하지 못하기 때문 

## 어댑터(뷰) 객체는 여러 개 만들어도 이득이 없음

- 어댑터: 실제 작업은 뒷단 객체에 위임하고 자신은 제2의 인터페이스 역할을 하는 객체
  - 뒷단 객체만 관리하면 되므로 여러 개를 만들 필요가 없고 뒷단 객체 하나당 어댑터 하나씩만 만들면 충분함
  - ex) Map 인터페이스의 keySet 메소드 
    - Set 어댑터를 반환
    - keySet을 호출할 때마다 매번 새로운 Set 인스턴스가 아닌 같은 Set 인스턴스를 반환함 
    - 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀜(모두 똑같은 Map 인스턴스를 대변하기 때문)   

## 오토박싱

- 오토박싱: 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술
  - 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만 완전히 없애주는 것은 아님

```java
//모든 양의 정수의 총합을 구하는 메소드
private static long sum() {
    Long sum = 0L;
    for(long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;

    return sum;
}
```
-  sum 변수를 long이 아닌 Long으로 선언해서 불필요한 Long 인스턴스가 만들어짐 Q. long vs. Long?<br>
-> 의도치 않은 오토박싱을 피하기 위해서는 박싱된 기본 타입보다는 기본 타입을 사용해야 함<br><br>
- 객체 생성을 무조건 피해야 한다는 것이 X, 불필요한 객체 생성은 코드 형태, 성능에만 영향을 주지만 방어적 복사(defensive copy)가 필요한 상황에서 객체를 재사용한다면 버그와 보안에 문제가 생김

<br><br>
# 아이템 7: 다 쓴 객체 참조를 해제하라

## 스택 구현 코드 - 메모리 누수 발생
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
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size]; // ★
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
- 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들은 더 이상 사용되지 않더라도 여전히 레퍼런스-다 쓴 참조(obsolete reference)-를 가지고 있음 -> 쓸데없이 메모리를 차지
  - elements 배열의 활성 영역 밖의 참조들이 이 경우에 해당됨
    - 활성 영역: 인덱스가 size보다 작은 원소들로 구성

## pop 메소드 수정

```java
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object value = this.elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return value;
    }
```
- 스택에서 꺼내지는 원소의 참조는 필요 없어지기 때문에 해당 참조를 null 처리(참조 해제)함  
  - null 처리한 참조를 실수로 사용하려는 경우 NullPointerException을 던지며 종료됨
- 그러나 객체 참조를 null 처리하는 일은 예외적인 일이며 모든 객체를 다 쓰자마자 null 처리할 필요는 없음
- 필요없는 참조를 정리하는 가장 좋은 방법: 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내기 -> 변수를 가능한 가장 최소의 범위에서 사용하면 됨
- 객체 참조를 null 처리해야 하는 경우: 메모리를 직접 관리하는 경우
  - 스택처럼 elements 배열로 원소들을 관리할 때 가비지 컬렉터는 어떤 원소들이 사용되고 사용되지 않는지 알 수 없음 (비활성 영역에서 참조하는 객체도 똑같이 유효한 객체로 인식)<br> ->  프로그래머가 비활성 영역에 속하는 객체를 null 처리하여 해당 객체를 쓰지 않겠다고 가비지 컬렉터에 알려줘야 함<br>
  -> 자기 메모리를 직접 관리하는 클래스라면 프로그래머가 항시 매모리 누수에 주의해야 함

## 캐시

- 객체 참조를 캐시에 넣고 그 객체를 다 쓴 뒤에도 놔두는 경우 메모리 누수 문제가 발생함
- 해결책
  - WeakHashMap: 캐시의 key를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 경우 사용 -> 다 쓴 엔트리는 캐시에서 자동으로 제거됨 cf)Weak reference
  - 백그라운드 스레드(ScheduledThreadPoolExecutor) 활용/ 새로운 엔트리 추가 시 기존 캐시 비우기(LinkedHashMap 클래스의 removeEldestEntry 메소드): 캐시를 만들 때 보통은 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에 시간이 지날수록 엔트리의 가치를 떨어트림 -> 쓰지 않는 엔트리를 청소해야 함

## 콜백(리스너)

- 클라이언트가 콜백을 등록만 하고 해지하지 않으면 콜백이 계속 쌓여서 메모리 누수 문제가 발생함
- 해결책
  - WeakHashMap: 콜백을 약한 참조(weak reference)로, key로 저장하면 가비지 컬렉터가 자동으로 즉시 수거해감
  
  <br><br>

# 아이템 8: finalizer와 cleaner 사용을 피하라

- 자바가 제공하는 두 가지 객체 소멸자
  - finalizer: 예측할 수 없고, 상황에 따라 위험, 일반적으로 불필요
  - cleaner: 자바9에서 finalizer의 대안으로 제안, finalizer보다는 덜 위험하지만 예측할 수 없고, 느리고, 일반적으로 불필요
  - cf) C++의 destructor
      - 특정 객체와 관련된 자원을 회수하는 보편적인 방법, 자바에서는 접근할 수 없게 된 객체를 회수하는 역할을 가비지 컬렉터가 담당함
      - 비메모리 자원을 회수하는 용도로도 쓰임, 자바에서는 try-with-resources와 try-finally를 사용해 해결함

## finalizer와 cleaner의 문제점

1. 제때 실행되어야 하는 작업을 할 수 없음

2. 특히 Finalizer는 인스턴스의 자원 회수를 지연시킬 수 있음<br>
: finalizer 스레드는 우선 순위가 낮아서 실행될 기회를 잘 얻지 못함, cleaner는 자신이 수행할 스레드를 제어할 수 있다는 면에서 낫지만 여전히 백그라운드에서 수행되며 가비지 컬렉터 통제 하에 있기 때문에 즉각 수행되리라는 보장이 없음

3. 상태를 영구적으로 수정하는 작업에서 사용하면 안됨<br>
: 수행 시점뿐만 아니라 수행 여부조차 보장이 안됨, 데이터베이스 같은 자원의 락 해제를 맡겨 놓으면 전체 분산 시스템이 멈춰버릴 수도 있음 

4. 성능 문제<br>
: 가비지 컬랙터의 효율을 떨어뜨림, but 안전망 형태로만 사용하면 훨씬 빨라짐

5. finalizer를 사용한 클래스는 finalizer 공격에 노출되어 보안 문제를 일으킬 수 있음 <br>
: 생성자나 직렬화 과정에서 예외 발생 시 생성되다 만 이 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있음 -> finalizer 공격을 방어하려면 final로 finalzier 메소드를 선언하여 하위 클래스를 만들지 못하게 할 것 

## finalizer와 cleaner의 대안
- AutoCloseable 인터페이스 구현, 인스턴스를 다 쓰고 나면 closer 메소드 호출<br>
  - 파일, 스레드 등 종료해야 할 자원을 담고 있는 객체의 클래스에서 finalizer, cleaner를 대신해 줌
  - 인터페이스 구현 시 try-with-resources를 사용, close 메소드에서 현재 인스턴스의 상태가 종료된 상태임을 필드에 기록, 다른 메소드에서 이 필드를 검사하여 이미 종료된 후에 불렀다면 IleegalStateException을 던질 것

## finalizer와 cleaner의 사용법1: 안전망 역할
- 자원의 소유자가 close 메소드를 호출하지 않는 경우에 대비한 안전망 역할<br>
   - 클라이언트가 close 메소드를 호출하지 않는 경우 대신 늦게라도 자원 회수를 해주는 것이 낫기 때문
   - 자바 라이브러리가 제공하는 안전망 역할의 finalizer: FileInputStream/FileOutputStream/ThreadPoolExecutor

## finalizer와 cleaner의 사용법2: 네이티브 피어 정리

- 네이티브 피어: 일반 자바 객체가 네이티브 메소드를 통해 기능을 위임한 네이티브 객체
  - 자바 객체가 아니기 때문에 가비지 컬렉터는 존재를 알지 못함 -> 자바 피어 회수 시 네이티브 객체까지 회수하지 못함 -> cleaner, finalizer가 처리할 수 있음
  - 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 사용할 수 있으며 그렇지 않으면 close 메소드를 사용해야 함

<br><br>


# 아이템 9: try-finally보다는 try-with-resources를 사용하라

- close 메소드를 호출해 직접 닫아줘야 하는 자원(InputStream, OutputStream, java.sql.Connection) 을 닫지 않으면 성능 문제가 발생할 수 있음

## try-finally
- 전통적으로는 try-finally를 사용해 자원을 닫아줬음
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
      return br.readLine();
    } finally {
        br.close();
    }
}
```
```java
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            ...
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
- 예외는 try, finally에서 모두 발생 가능<br>
    - readLine 메소드에서 예외를 던지고, close 메소드도 실패하는 경우: readLine 메소드의 예외는 기록되지 않아 디버깅이 어려워짐
- 자원이 두 개 이상일 경우에는 코드가 복잡해짐 

## try-with-resources
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
      return br.readLine();
    }
}
```
```java
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
          byte[] buf = new byte[BUFFER_SIZE];
          int n;
          while ((n = in.read(buf)) >= 0){
            out.write(buf, 0, n);
          }
}
```
- readLine 메소드, close 메소드에서 예외 발생 시: readLine에서 발생한 예외가 기록됨 
- 다른 예외는 스택 추적 내역에 'suppressed'를 달고 출력됨
  - Throwable의 getSuppressed 메소드로 가져올 수 있음(자바 7)
- try-with-resources에서도 catch절 사용 가능

