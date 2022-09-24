# 아이템 49: 매개변수가 유효한지 검사하라
- 매개변수는 메서드 몸체가 시작되기 전에 검사해야 함
- 매개변수에 대한 제약, 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 함 
- 매개변수 검사를 제대로 하지 못하면? 실패 원자성을 어길 수 있음 
  - 메서드가 수행되는 중간에 모호한 예외를 던지며 실패하거나 메서드가 잘못된 결과를 반환할 수 있음
  - 어떤 객체의 상태를 바꿔놓아서 메서드와 관련이 없는 오류를 낼 수 있음
- 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수도 신경 써서 검사해야 함

### 매개변수의 유효성 검사 방법
1. null 검사
    - @Nullable과 같은 애너테이션 사용: 표준적인 방법은 아님
    - requireNonNull 메서드(자바7~): 예외 메시지 지정 가능, 입력된 값을 그대로 반환하므로 값을 사용할 수 있음
2. 범위 검사
    - checkFromIndexSize, checkFromToIndex, checkIndex 메서드(자바9~): 예외 메시지 지정 불가, 리스트와 배열 전용, 양 끝단 값을 포함하는 닫힌 범위는 다루지 못함
3. assert(단언문) 사용
    ```java
    private static void sort(long a[], int offset, int length){
        assert a != null;
        assert offset >= 0 && a.length;
        ...
    }
    ```
    - 공개되지 않은 메서드(private)가 호출될 때 메서드에 넘겨지는 값을 통제하기 위해 사용
    - 실패하면(매개변수가 조건을 참으로 만들지 않으면) AssertionError를 던짐
    - 런타임에 영향을 주지 않음(-ea 또는 --enableassertions 플래그를 설정하면 런타임에 영향을 줌)  

### 매개변수의 유효성 검사가 필요 없는 경우
1. 유효성 검사 비용이 지나치게 높거나 계산 과정에서 암묵적으로 검사가 수행될 때
    - ex) 객체 리스트를 정렬하는 Collections.sort(List) 메서드: 객체끼리 비교하는 과정에서 비교할 수 없는 타입의 객체가 있으면 ClassCastException을 던지기 때문에 리스트 안의 객체가 상호 비교될 수 있는지 검사할 필요가 없음
    - 계산 과정에서 유효성 검사가 이뤄져도 잘못된 예외를 던질 수 있음 -> 예외 번역 관용구를 사용하여 API 문서에 기재된 예외로 번역해줘야 함 

<br><br>

# 아이템 50: 적시에 방어적 복사본을 만들라

### 자바의 특징 
- 네이티브 메서드를 사용하지 않아서 버퍼 오버런, 배열 오버런, 와일드 포인터 같은 메모리 충돌 오류에서 안전함
- 메모리 전체를 배열로 다루는 언어와 달리 자바로 작성한 클래스는 불변식이 지켜짐
  - but, 클라이언트가 불변식을 깨뜨리려고 한다고 가정하고 방어적으로 프로그래밍해야 함   
    - ex) Date를 사용하는 경우: Date 클래스는 가변이기 때문에 불변식을 깨뜨릴 수 있음

### 방어적 복사 시 주의사항
- 외부 공격으로부터 인스턴스 내부를 보호하려면 생성자에서 받은 가변 매개변수를 방어적으로 복사하고 인스턴스 안에서 복사본을 사용해야 함
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if(this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
    }
}
```
1. 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사할 것
    - 멀티스레딩 환경에서는 원본 객체의 유효성을 검사한 후 복사본을 만드는 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문 (검사시점/사용시점 공격 - TOCTOU 공격)
2. 매개변수가 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone 메서드를 사용하지 말 것
    - Date는 final 클래스가 아니므로 clone으로 하위 클래스의 인스턴스를 반환할 수도 있음
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if(this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
    }
}
```
3. 접근자(getter) 메서드에서 가변 필드의 방어적 복사본을 반환할 것
    - 생성자와 달리 접근자 메서드에서는 clone 메서드를 사용해도 됨: Date 객체인 것이 확실하기 때문 but, 인스턴스를 복사할 때에는 일반적으로 생성자나 정적 팩터리를 쓰는 것이 좋음
```java
// 접근자 메서드가 내부의 가변 정보를 직접 드러내지 못하게 방어적 복사본을 반환함
public Date start() { 
    return new Date(start.getTime());
}
public Date end() { 
    return new Date(end.getTime());
}
```
4. 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지 생각해야 함 
    - -> 변경되었을 때 문제가 발생하지 않는다고 확신하지 못하면 복사본을 만들어서 저장해야 함 
    - 내부 객체를 클라이언트에 건네줄 때도 마찬가지로 방어적 복사본을 반환해야 함

### 예외상황
- 방어적 복사에는 성능 저하가 따르기 때문에 호출자가 컴포넌트 내부를 수정하지 않으리라 확신한다면 방어적 복사를 생략할 수 있음
  - 가변 매개변수를 항상 방어적으로 복사해야하는 것은 아님, 때로는 매개변수로 넘기는 행위가 그 객체의 통제권을 명백히 이전함을 뜻하기도 함 
    - 이때 클라이언트에서 해당 객체를 수정하지 않는다고 확실히 약속하고 문서화해야 함
  - 클래스와 클라이언트가 상호 신뢰할 수 있고, 불변식이 깨지더라도 그 영향이 클라이언트에만 국한될 때는 방어적 복사 생략 가능
    - ex) 래퍼 클래스 패턴 - 클라이언트는 래퍼에 넘긴 객체에 직접 접근할 수 있으나(불변식을 파괴할 수 있으나) 그 영향은 클라이언트 자신만 받게 됨 

<br><br>

# 아이템 51: 메서드 시그니처를 신중히 설계하라

### API 설계 팁
1. 메서드 이름은 신중히 짓자
    - 표준 명명 규칙을 따를 것 
    - 같은 패키지에 속한 다른 이름들과 일관되게 짓고 긴 이름은 피할 것 
    - 자바 라이브러리 가이드를 참조할 것 
2. 편의 메서드를 너무 많이 만들지 말자
3. 매개변수 목록은 짧게 유지하자 
    - 4개 이하가 적절함
    - 같은 타입의 매개변수가 여러 개 나오는 경우는 해로움

### 매개변수 목록을 짧게 줄여주는 기술
1. 여러 메서드로 쪼개기 
    - 직교성을 높여 오히려 메서드 수를 줄여줄 수도 있음
      - cf) 직교성: 기능을 원자적으로 쪼개 제공하는 것, 기능들을 쪼개고 필요 시 조립해서 사용한다면 메서드 수를 줄일 수 있음
      - ex) List 인터페이스: subList 메서드(부분리스트 반환) + indexOf 메서드(원소의 인덱스를 알려줌) = 부분리스트의 인덱스를 찾는 기능 구현 가능
2. 매개변수 여러 개를 묶어주는 도우미 클래스 만들기
    - 일반적으로 도우미 클래스는 정적 멤버 클래스
3. 빌더 패턴을 메서드 호출에 응용 
    - 매개변수가 많고, 일부를 생략해도 될 때 사용 
    - 모든 매개변수를 하나로 추상화한 객체를 정의 -> 클라이언트에서 세터 메서드 호출해 필요한 값 설정 (+execute 메서드로 매개변수의 유효성 검사)

### 매개변수 설정 시 주의사항 
1. 매개변수의 타입은 클래스보다는 인터페이스를 사용할 것
    - ex) HashMap보다는 Map
2.  boolean보다는 원소 2개짜리 열거 타입을 사용할 것
      ```java
      public enum TemperatureScale { FAHRENHEIT, CELSCIUS }

      //boolean
      Thermometer.newInstance(true)

      //enum : 하는 일을 명확하게 알려줌
      Thermometer.newInstance(TemperatureScale.CELSCIUS)
      ```

<br><br>

# 아이템 52: 다중정의는 신중히 사용하라

### 재정의 vs 다중정의 
- 재정의한 메서드
  - 매개변수의 런타임 타입에 의해 호출되는 메서드가 정해짐 (동적으로 선택됨)
- 다중정의한 메서드
  - 매개변수의 컴파일타임 타입에 의해 호출되는 메서드가 정해짐 (정적으로 선택됨) -> 예상대로 작동하지 않아 혼란을 일으킬 수 있음
  - cf) 정적 메서드에서 instanceof를 사용하여 매개변수의 런타임 타입을 동적으로 검사할 수 있음  

### 다중정의 시 주의사항
1. 매개변수 수가 같을 경우 다중정의 메서드를 만들지 말 것
2. 가변인수를 사용할 경우 다중정의 메서드를 만들지 말 것
3. 다중정의하는 대신 메서드 이름을 다르게 짓는 방법을 고려해볼 것 
    - ex) ObjectOutputStream 클래스 - writeBoolean(boolean), writeInt(int), writeLong(long)
    - 생성자는 이름을 다르게 지을 수 없으니 두 번째 생성자부터는 무조건 다중정의가 됨 -> 정적 팩터리라는 대안을 활용할 수 있음
4. 다중정의한 메서드에 각각 근본적으로 다른 매개변수를 사용할 것 
   - ex) ArrayList - int를 받는 생성자, Collection을 받는 생성자 
   - 제네릭 및 오토박싱(자바5~)이 도입되면서 매개변수가 각각 기본 타입과 참조 타입이어도 근본적으로 다르다고 볼 수 없게 되었음 -> 형변환을 통해 다중정의된 메서드들 중 올바른 메서드를 선택할 수 있게 해야함
     - ex) `List<E>` 인터페이스 - remove(Object), remove(int) 메서드  
    - 람다와 메서드 참조(자바8~)가 도입되면서 매개변수가 각각 서로 다른 함수형 인터페이스라도 근본적으로 다르다고 볼 수 없게 되었음 -> 같은 위치의 인수로 받으면 안됨

<br><br>

# 아이템 53: 가변인수는 신중히 사용하라
- 가변인수 메서드 호출 시, 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 메서드에 건네줌

```java
// 인수가 1개 이상이어야 하는 가변인수 메서드 ex) 최소값 구하는 메서드
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs) {
        if (arg < min) {
            min = arg;
        }
    }
    return min;
}
```
- 인수를 0개만 받을 수 있게 설계하면 런타임 오류 발생
  - -> 평범한 매개변수, 가변인수를 모두 받게 함

### 가변인수 메서드 사용 시 효과적인 패턴
- 가변인수 메서드는 호출될 때마다 배열을 새로 할당하고 초기화함 -> 성능에 문제가 생길 수 있음 
- -> 다중정의 메서드를 사용하여 배열 생성 비용을 줄일 수 있음
  - ex) EnumSet의 정적 팩터리도 이 패턴을 사용하여 열거 타입 집합 생성 비용을 최소화함
```java
public void foo() {}
public void foo(int arg1) {}
public void foo(int arg1, arg2) {}
public void foo(int arg1, arg2, arg3) {}
public void foo(int arg1, arg2, arg3, int... rest) {}
```

<br><br>

# 아이템 54: null이 아닌 빈 컬렉션이나 배열을 반환하라

### 컬렉션이 비었을 때 null을 반환하는 메서드
```java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheese() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```
- 클라이언트에서 null을 처리하는 방어 코드를 넣어줘야 함 (방어 코드를 빼먹으면 오류가 발생할 수 있음)

### 빈 컨테이너를 반환해야 하는 이유
- null을 반환하든 컬렉션 or 배열을 반환하든 성능 차이는 크게 존재하지 않음
- 빈 컬렉션과 배열을 새롭게 할당하는 과정에서 성능이 떨어진다면 -> 새로 할당하지 않고도 반환할 수 있음 
  1. 매번 똑같은 빈 '불변' 컬렉션을 반환하면 됨 (최적화: 실제로 성능이 개선되는지 확인할 것)
  ```java
  // 빈 컬렉션 반환
  public List<Cheese> getCheeses(){
      return new ArrayList<>(cheeseInStock); 
  }

  // 매번 똑같은 빈 불변 컬렉션 반환
  public List<Cheese> getCheeses() {
      return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
  }
  ```
  2. 매번 미리 선언해둔 길이 0짜리 배열을 반환하면 됨(최적화)
   ```java
   // 길이가 0인 배열 반환
  public Cheese[] getCheeses() {
      return cheesesInStock.toArray(new Cheese[0]);
  }
  
  // 배열을 새로 할당하지 않고 미리 선언해둔 배열을 반환 
  private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

  public Cheese[] getCheeses() { // cheesesInStock이 비었을 때면 언제나 EMPTY_CHEESE_ARRAY를 반환하게 됨
      return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
      // 아래처럼 toArray에 넘기는 배열을 미리 할당하면 성능이 떨어질 수도 있음
      // return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
  }
  ```

<br> <br>

# 아이템 55: 옵셔널 반환은 신중히 하라

### 특정 조건에서 메서드가 값을 반환할 수 없을 때 (~자바8)
1. 예외를 던짐
    - 진짜 예외적인 경우에만 사용해야 함, 예외 생성 시 스택 추적 전체를 캡쳐하므로 비용 ↑
2. null을 반환함
    - null을 반환할 수 있는 메서드일 경우 null 처리 코드를 추가해야 함 (그렇지 않으면 NullPointerException 발생)

### Optional (자바8~)
- 보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 `Optional<T>`를 반환하도록 선언할 것 
  - null이 아닌 T타입 참조를 하나 담거나, 또는 아무것도 담지 않을 수 있음
  - 원소를 최대 1개 가질 수 있는 불변 컬렉션 
  - 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적음
  - 반환값이 없을 수도 있음을 명확하게 알려줌 (검사 예외와 비슷함)
  - 옵셔널을 반환하는 메서드에서는 절대 null을 반환하면 안됨 

      ```java
    public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
        //빈 옵셔널 반환
        if (c.isEmpty()) return Optional.empty();

        E result = null;
        for (E e : c ) {
            if (result == null || e.compareTo(result) > 0) {
                result = Objects.requireNonNull(e);
            }
        }

        // 값이 든 옵셔널 반환 (null 반환 시 오류)
        // cf) ofNullable(value) : null값도 허용
        return Optional.of(result);
    }
       ```

### 옵셔널 활용 방법 (클라이언트가 값을 받지 못했을 때)
1. 기본값 설정
    ```java
    String lastWordInLexicon = max(words).orElse("단어 없음...");
    ```
2. 원하는 예외 던지기
    - 실제 예외가 아닌 예외 팩터리 -> 예외 생성 비용이 들지 않음
    ```java
    Toy myToy = max(toys).orElseThrow(TemperTantrumException::new); 
    ```
3. 항상 값이 채워져 있다고 가정하고 바로 값을 꺼내 쓰기
    - 값이 없다면 NoSuchElementException이 발생   
    ```java
    Element lastNobleGas = max(Elements.NOBLE_GASES).get();
    ``` 
4. 기본값 설정 비용이 큰 경우
    - `Supplier<T>`를 인수로 받는 orElseGet 사용 -> 값이 처음 필요할 때 `Supplier<T>`를 사용해 생성하므로 초기 설정 비용을 낮출 수 있음
5. optional의 고급 메서드 사용하기
    - ex) filter, map, flatMap, ifPresent
    - isPresent : 옵셔널이 채워져 있으면 true, 비어 있으면 false 반환 (but, 신중히 사용해야 함)
    - stream(Optional을 Stream으로 변환) + Stream의 flatMap 메서드와 조합

### 옵셔널 사용 시 주의사항
1. 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 됨
    - 빈 `Optional<List<T>>`를 반환하기보다는 빈 `List<T>`를 반환하는 것이 나음
    - 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 따로 만들지 않아도 됨
2. 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 옵셔널을 반환할 것
    - 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수도 있음
3. 박싱된 기본 타입을 담은 옵셔널은 반환하지 말 것
    - 기본 타입 자체보다 무거울 수밖에 없으므로 OptionalInt, OptionalLong, OptionalDouble과 같은 전용 클래스를 사용할 것
4. 옵셔널을 컬렉션의 키, 값, 원소 or 배열의 원소로 사용하는 것은 적절하지 않음
    - ex) map의 값으로 사용할 때: 키 자체가 없는 경우와 키는 있지만 그 키가 속이 빈 옵셔널인 경우로 나뉘어서 혼란을 가져옴  

<br> <br>

# 아이템 56: 공개된 API 요소에는 항상 문서화 주석을 작성하라

- 자바독: 소스코드 파일에서 문서화 주석으로 기술된 설명을 추려 API 문서로 변환해줌

### 문서화 주석 작성 방법 
- API를 올바르게 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 함
    - 직렬화할 수 있는 클래스라면 직렬화 형태에 관해서도 적어야 함
- 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 함
    - 상속용으로 설계된 클래스의 메서드가 아니라면 어떻게 동작하는지가 아니라 무엇을 하는지를 기술해야 함
    - 클라이언트가 메서드를 호출하기 위한 전제조건(@throws 태그로 비검사 예외 선언, @param 태그로 매개변수 기술), 메서드가 수행된 후에 만족해야 하는 사후조건을 모두 나열해야 함
    - 부작용(=시스템 상태 변화)도 기술해야 함

### 태그 종류 및 사용 가이드라인  
1. @param, @return
    - 관례상 명사구를 사용함
    - 마침표를 붙이지 않음
2. @throws
    - 해당 예외를 던지는 조건을 설명하는 절을 기술
    - 마침표를 붙이지 않음
3. @code
    - 태그로 감싼 내용을 코드용 폰트로 렌더링
    - 태그로 감싼 내용에 포함된 HTML 요소, 다른 자바독 태그를 무시함
4. @implSpec 
    - 해당 메서드와 하위 클래스 사이의 계약을 설명함 

### 문서화 주석 작성 시 주의사항
- 가독성을 우선적으로 고려해야 함
- 제네릭 타입, 제네릭 메서드의 경우 모든 타입 매개변수에 주석을 달아야 함
- 열거 타입의 경우 상수들에도 주석을 달아야 함
- 애너테이션 타입의 경우 멤버들에도 모두 주석을 달아야 함 
- 클래스 혹은 정적 메서드는 스레드 안전 수준을 반드시 포함해야 함
- 직렬화할 수 있는 클래스라면 직렬화 형태도 기술해야 함
- 메서드 주석은 상속할 수 있음. 주석이 없는 경우 자바독이 가장 가까운 문서화 주석을 찾아줌 (클래스보다는 인터페이스를 먼저 찾음)
-여러 클래스가 상호작용하는 복잡한 API라면 주석 외에도 전체 아키텍처를 설명하는 문서가 필요할 수 있음





