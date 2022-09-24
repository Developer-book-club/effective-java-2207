# 아이템 42: 익명 클래스보다는 람다를 사용하라

### 함수 객체 (function object)
- 추상 메서드를 하나만 담은 인터페이스(or 추상클래스)의 인스턴스 
- 자바에서 함수 타입을 표현할 때 사용 

### 익명 클래스
- JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단이 됨
- 익명 클래스 방식은 코드가 너무 긺 ->  함수형 프로그래밍에 적합하지 않음

```java
// 익명 클래스의 인스턴스를 함수 객체로 사용
Collections.sort(words, new Comparator<String>() {
        public int compare(String s1, String s2) {
            return Integer.compare(s1.length(), s2.length());
        }
});
```

### 람다(lambda)
- 자바 8부터 추상 메서드 하나 짜리 인터페이스(함수형 인터페이스)의 인스턴스를 람다식을 사용해 만들 수 있게 됨

```java
// 람다식을 함수 객체로 사용 (익명 클래스 대체)
Collections.sort(words,
                (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```
- 컴파일러가 코드의 문맥을 살펴 타입을 추론해줌
  - 람다의 타입: (`Comparator<String>`)
  - 매개변수(s1, s2)의 타입: String, int (s1, s2)
  - 반환값의 타입: int
  - cf) 컴파일러는 대부분의 타입 정보를 제네릭에서 얻음
- 컴파일러가 타입을 결정해주지 못하는 상황에서는 직접 명시해야 함  
- => 타입을 명시해야 코드가 더 명확할 때를 제외하고, 람다의 모든 매개변수 타입은 생략할 것

```java
// 코드를 간결하게 만드는 방법
// 1.람다 자리에 비교자 생성 메서드 사용
Collections.sort(words, comparingInt(String::length));
// 2.List 인터페이스에 추가된 sort 메서드 사용 (JAVA 8~)
words.sort(comparingInt(String::length));
```
```java
// 람다 방식으로 열거 타입의 인스턴스 필드를 작성 -> 상수별로 다르게 동작하는 코드 구현
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op; // DoubleBinaryOperator: Double 타입 인수 2개를 받아 Double 타입 결과를 반환해주는 인터페이스

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

### 람다식의 한계
1. 람다는 이름이 없고 문서화를 할 수 없음 
    - 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 사용을 지양해야 함
2. 람다는 함수형 인터페이스에서만 쓰임
    - 추상 클래스, 또는 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때는 익명 클래스를 사용해야 함
3. 람다는 자신을 참조할 수 없음
    - 함수 객체가 자신을 참조해야 한다면 익명 클래스를 써야 함
4. 람다를 직렬화하는 일을 삼가야 함 (Q)


<br><br>

# 아이템 43: 람다보다는 메서드 참조를 사용하라

### 메서드 참조(method reference)
- 함수 객체를 람다보다도 더 간결하게 만드는 방법
- 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용
```java
// 람다 사용
map.merge(key, 1, (count, incr) -> count + incr);
//메서드 참조 사용 (Integer 클래스 - 정적 메서드 sum)
map.merge(key, 1, Integer::sum);
```
- 람다가 메서드 참조보다 더 간결한 경우도 있음 
  - ex. 메서드와 람다가 같은 클래스에 있을 때
```java
// 메서드 참조 (GoshThisClassNameIsHumongous  클래스 - action 메서드)
servie.execute(GoshThisClassNameIsHumongous::action);
// 람다
service.execute(() -> action());
```

### 메서드 참조의 유형
- 가리키는 메서드의 종류에 따라 
1. 정적 메서드
    - ex) Integer::parseInt
    - 람다) str -> Integer.parseInt(str) 
2. 한정적 인스턴스 메서드: 수신 객체를 특정
    - ex) Instant.now()::isAfter
    - 람다) Instant then = Instant.now();
t -> then.isAfter(t)
3. 비한정적 인스턴스 메서드: 수신 객체를 특정하지 않음
    - ex) String::toLowerCase
    - 람다) str -> str.toLowerCase()
4. 클래스 생성자
    - ex) TreeMap<K,V>::new
    - 람다) () -> new TreeMap<K,V>()
5. 배열 생성자
    - ex) Int[]::new
    - 람다) len -> new int[len]

<br>

- cf) 제네릭 함수 타입 구현: 람다로는 불가능하나 메서드 참조로는 가능한 유일한 예 (제네릭 람다식이라는 문법이 존재하지 않기 때문) (Q)

<br><br>

# 아이템 44: 표준 함수형 인터페이스를 사용하라

### 람다 지원 후 API 작성 방식의 변화
- 템플릿 메서드 패턴(상위 클래스의 기본 메서드를 재정의) 대신 함수 객체를 매개변수로 받는 생성자, 메서드가 사용됨
  - 함수형 매개변수 타입을 올바르게 선택해야 함
  - ex.  LinkedHashMap 클래스의 removeEldestEntry 메서드 
    - size()를 호출해서 사용: removeEldestEntry가 인스턴스 메서드라 가능한 방식
    - 람다를 사용해서 정적 팩터리나 생성자로 구현한다면 Map 자신도 함수 객체에 넘겨주어야 함
```java
// removeEldestEntry를 함수형 인터페이스로 선언한 예시
@FunctionInterface interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K,V> map, Map.Entry<K, V> eldest);
}
```

### 표준 함수형 인터페이스 (java.util.function)
- 유용한 디폴트 메소드 제공
- 대부분 기본 타입만 지원
  - 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말 것 -> 성능이 느려질 수 있음

### 기본 표준 함수형 인터페이스 6개 (총 43개 중) 
1. `UnaryOperator<T>`
    - T apply(T t) : 인수와 반환값의 타입이 같은 함수(인수 1개)
    - ex) String::toLowerCase
2. `BinaryOperator<T>`
    - T apply(T t1, T t2) : 인수와 반환값의 타입이 같은 함수(인수 2개)
    - ex) BigInteger::add
3. `Predicate<T>`
    - boolean test(T t) : 인수 하나를 받아 boolean을 반환하는 함수
    - ex) Collection::isEmpty
4. `Function<T,R>`
    - R apply(T t) : 인수와 반환값의 타입이 다른 함수
    - ex) Arrays::asList
5. `Supplier<T>`
    - T get() : 인수를 받지 않고 값을 반환하는 함수
    - ex) Instant::now
6. `Consumer<T>`
    - void accept(T t) : 인수 하나를 받고 반환값은 없는 함수
    - ex) System.out::println

### 함수형 인터페이스를 직접 작성해서 사용하는 경우
- 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 다음 중 하나 이상의 조건을 만족하면 전용 함수형 인터페이스를 구현해서 사용할 것
  - 1. 이름이 용도를 잘 설명해줌
  - 2. 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있음
  - 3. 유용한 디폴트 메서드를 담고 있음 
- ex) `Comparator<T>`
  - `ToIntBiFunction<T, U>`와 구조적으로 동일(인수 두 개를 받아서 정수형을 반환하는 인수와 반환 타입이 다른 함수)하나 독자적인 인터페이스로 사용됨  

### 함수형 인터페이스 작성 시 주의사항
1. @FunctionalInterface 애너테이션을 사용할 것 
   - 인터페이스가 람다형으로 설계된 것임을 알려줌
   - 해당 인터페이스가 추상 메서드를 오직 하나만 가질 수 있게 함(그렇지 않으면 컴파일 불가능)
   - 유지보수 과정에서 메서드를 추가하지 못하게 막아줌 
2. 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의해서는 안됨

<br><br>

# 아이템 45: 스트림은 주의해서 사용하라
- 스트림 API: 다량의 데이터 처리 작업을 위해 자바8에 추가됨
  - 스트림, 스트림 파이프라인이라는 두 가지의 핵심 개념을 제공

### 스트림과 스트림 파이프라인
- 스트림
  - 데이터 원소의 유한 혹은 무한 시퀀스
  - 원소들은 컬렉션, 배열, 파일, 정규표현식 패턴 매처, 난수 생성기, 다른 스트림 등을 통해 만들어짐
- 스트림 파이프라인
  - 원소들로 수행하는 연산 단계를 표현하는 개념
  - 소스 스트림에서 시작해 종단 연산으로 끝나며 그 사이에 하나 이상의 중간 연산이 있을 수 있음
    - 중간 연산: 스트림을 변환함 ex) 원소에 함수를 적용, 원소를 걸러냄
    - 종단 연산: 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가함 ex) 원소를 정렬해 컬렉션에 담음, 특정 원소를 선택함, 원소를 출력함
  - 지연 평가됨: 평가는 종단 연산이 호출될 때 이루어짐
    - 무한 스트림을 다룰 수 있게 해줌 
  - 파이프라인 하나를 구성하는 모든 호출을 연결하거나, 파이프라인 여러 개를 연결해 표현식 하나로 만들 수 있음

### 스트림 사용 시 주의할 점
1. 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야함
2. 도우미 메서드를 사용하여 반복문과 같은 로직을 스트림에서 분리해줘야 함 

### 코드 블록 vs 람다 블록
- 스트림 파이프라인은 되풀이 되는 계산을 함수 객체(람다나 메서드 참조)로 표현하나 반복문은 코드 블록을 사용해 표현함
1. 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있으나, 람다에서는 final이거나 혹은 사실상 final인 변수만 읽을 수 있음. 지역 변수를 수정하는 것은 불가능.
2. 코드 블록에서는 return 문으로 메서드를 빠져나가거나, break, continue 문을 통하여 블록 바깥의 반복문을 종료하거나 건너뛸 수 있음. 람다에서는 불가능

### 스트림이 적합한 경우
- 원소들의 시퀀스를 일관되게 변환하는 경우
- 원소들의 시퀀스를 필터링하는 경우
- 원소들의 시퀀스를 하나의 연산을 사용해 결합하는 경우(더하기, 연결하기, 최솟값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모으는 경우
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는 경우

### 스트림이 적합하지 않는 경우
- 스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조
  - 데이터의 각 단계의 값들에 동시에 접근하기 어려운 구조는 스트림으로 처리하기 어려움

<br><br>

# 아이템 46: 스트림에서는 부작용 없는 함수를 사용하라
- 스트림 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 것
  - 이때 각 변환 단계는 가능한 이전 단계의 결과를 받아 처리하는 순수 함수(입력만이 결과에 영향을 주는 함수)여야 함
 
 ### 스트림의 특성을 이해하지 못한 사례
 1. ex) forEach 연산: 종단 연산 -> 계산을 진행하는 용도로 사용하지 말고 스트림 연산 결과를 보여줄 때만 사용할 것 
```java
// forEach에서 연산이 일어나고 외부 상태를 수정하는 람다가 실행되면서 문제가 생김
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);;
    })
}
// 수정본
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

### collector 클래스
- 스트림을 사용할 때 필요한 요소
- 스트림의 원소들을 컬렉션 객체 하나에 취합
- 메서드의 종류
  - toList(), toSet(), toCollection(collectionFactory) : 각각 리스트, 집합, 지정한 컬렉션 타입 반환하는 메서드  
  - toMap
  - groupingBy, groupingByConcurrent, partitioningBy
  - counting
  - reducing 메서스들
  - minBy, maxBy
  - joining  

<br><br>

# 아이템 47: 반환 타입으로는 스트림보다 컬렉션이 낫다
- 원소 시퀀스(일련의 원소를 반환하는) 반환 타입: Collection 인터페이스, Iterable, 배열 + 스트림(자바 8~)
 
### 스트림 
- 반복(iteration)을 지원하지 않기 때문에 스트림과 반복을 알맞게 조합해야 함
  - Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하였고 Iterable 인터페이스가 정의한 방식대로 동작하지만 for-each로 스트림을 반복할 수는 없음 -> Stream이 Iterable을 확장(extends)하지 않았기 때문

```java
// 1. Stream<E>를 Iterable<E>로 중개해주는 어댑터
// 어댑터 메서드를 사용하여 스트림을 반복할 수 있게 함
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

//스트림도 for-each문으로 반복 가능
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // 프로세스 처리
}

// 2. Iterable<E>를 Stream<E>로 중개해주는 어댑터
// 반환한 값을 스트림 파이프라인에서 처리할 때
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

- 스트림 반환: 객체 시퀀스를 반환하는 메서드가 스트림 파이프라인에서만 쓰일 때
- Iterable 반환: 객체들이 반복문에서만 쓰일 때 

### 반환 타입으로 컬렉션을 사용해야 하는 이유
- 공개 API의 반환 타입: 스트림 파이프라인을 사용하는 경우, 반복문에서 사용하는 경우 모두를 고려해야 함 -> 컬렉션, 컬렉션의 하위 타입을 쓰는 것이 최선
1. Collection 인터페이스는 Iterable의 하위 타입이자 Stream 메서드를 제공함 -> 반복과 스트림을 동시에 지원
2. Arrays는 Arrays.asList와 Stream.of 메서드 지원 -> 반복과 스트림을 동시에 지원

### collection 반환 vs Stream or Iterable 반환 (Q)
- 반환하는 시퀀스의 크기가 작다면 ArrayList, HashSet 같은 표준 컬렉션 구현체를 반환할 것
  - 시퀀스의 크기가 크다면 전용 컬렉션을 구현할 것
    - ex) 멱집합 반환 시 AbstractList를 이용, 각 원소의 인덱스를 비트 벡터로 사용 
- contain, size를 구현하기 어려울 때는 컬렉션보다는 스트림이나 Iterable을 반환 (??)

<br><br>

# 아이템 48: 스트림 병렬화는 주의해서 적용하라
- parallel 메서드: 스트림 파이프라인을 병렬 실행할 수 있는 메서드  
- 데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 사용하면 파이프라인 병렬화로는 성능 개선을 기대할 수 없음 

### 병렬화가 효과적인 경우 
1. 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int, long 범위 일 때
    - 데이터를 원하는 크기로 정확하고 쉽게 나눌 수 있어 다수의 스레드에 분배하기 좋기 때문
    - 참조 지역성(locally of reference) 뛰어남: 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있음
      - 참조 지역성이 낮으면 스레드가 주 메모리에서 캐시 메모리로 전송되는 데이터를 기다리는 시간이 길어짐 -> 참조 지역성은 다량의 데이터를 처리하는 벌크 연산 병렬화 시 중요한 요소로 작용
      - 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열 -> 참조가 아닌 데이터 자체가 메모리에 연속해서 저장되기 때문
2. 종단 연산 중 축소 연산을 수행하는 경우
      - 축소: 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업
        - ex) Stream의 reduce 메서드 중 하나, 또는 min, max, count, sum과 같이 완성된 형태로 제공되는 메서드 
        - ex) anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환하는 메서드도 적합
        - cf) 가변 축소(mutable reduction)을 수행하는 collect 메서드는 적합하지 않음 (합치는 비용이 크기 때문)


