람다를 지원하며 템플릿 메서드 패턴의 매력이 크게 줄었다. 해법은 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것인데…

즉, **함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다. 이때 함수형 매개변수 타입을 올바르게 선택해야 한다.**

# 예시

## LinkedHashMap

해당 클래스의 `removeEldestEntry` 를 재정의 하면 캐시처럼 사용이 가능하다.

→ 해당 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거해준다.

```java
// 100개 제한하도록 재정의
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
	return size() > 100;
}
```

Map.Entry를 받아 boolean을 반환해야 할 것 같지만 꼭 그렇진 않다.

`size()` 를 호출해 맵 안의 원소 수를 알아내는데 인스턴스 메서드라서 가능한 방식이다.

하지만 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아니다. 팩터리나 생성자를 호출할 땐 인스턴스가 존재하지 않기 때문이. 즉, 그래서 자기 자신(map) 도 함수 객체에 건네줘야 한다.

```java
// 위 메서드와 동일한 함수형 인터페이스
@FunctionalInterface
interface EldestEntryRemovalFunction<K,V> {
	boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

대신 이 인터페이스는 자바 표준 라이브러리에 동일한 모양의 인터페이스가 준비되어 있어 굳이 사용할 이유는 없다.

→ 직접만든 위 인터페이스 대신 표준 인터페이스인 `BiPredicate<Map<K,V>` , `Map.Entry<K,V>>` 를 사용할 수 있다.

**필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.**

# 함수형 인터페이스

## 기본 인터페이스

모두 참조 타입용이다.

### 1. Operator

반환값과 인수의 타입이 같은 함수

- UnaryOperator
  - 인수가 1개
- BinaryOperator
  - 인수가 2개

### 2. Predicate

인수 하나를 받아 boolean을 반환하는 함수

### 3. Function

인수와 반환 타입이 다른 함수

### 4. Supplier

인수를 받지 않고 값을 반환(혹은 제공)하는 함수

### 5. Consumer

인수를 하나 받고 반환값은 없는(인수를 소비하는) 함수

| 인터페이스        | 함수 시그니처       | 예                  |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T,R>     | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

## 기본 타입에 대한 변형 3가지

1. int
2. long
3. double

위 세가지 용으로 각 3개씩 변형이 있다.

(ex) Predicate

→ IntPredicate, LongPredicate, DoublePredicate…

## Function 인터페이스 변형

Function의 변형, 정확히는 반환 타입만 매개변수화 되었다.

(ex) LongFunction<int[]>

→ long 인수를 받아 int[]를 반환

추가적으로 기본 타입을 반환하는 변형이 총 9개가 더 있다.

1. *Src*ToResult
   - 입력과 결과 타입이 모두 기본타입 (총 6개)
   - (ex) long을 받아 int를 반환 → LongToIntFunction
2. To*Result*
   - 입력이 객체 참조이고 결과가 int, long, double인 변형 (3개)
   - (ex) int[] 인수를 받아 long 반환 → ToLongFunction<int[]>

## 인수를 2개씩 받는 변형

- `BiPredicate<T,U>`
- `BiFunction<T,U,R>`
- `BiConsumer<T,U>`

추가적으로 `BiFunction` 에는 기본 타입을 반환하는 세 변형 (int, long, double) 이 있다.

`Consumer` 에도 `ObjDoubleConsumer` , `ObjLongConsumer` 가 존재한다.

이리 하여 기본 인터페이스의 인수 2개짜리 변형은 총 9개이다.

## BooleanSupplier 인터페이스

boolean을 반환하도록 한 `Supplier` 의 변형

(사실 Predicate와 그 변형 4개도 boolean값을 반환할 수 있다.)

## 그래서

기본 타입 (6개) + 기본타입에 대한 변형 3가지 (18개) + Function 인터페이스 변형 (9개) + 인수를 2개씩 받는 변형 (9개) + Boolean Supplier 인터페이스 (1개)

= 총 43개

- 표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 하지만 **기본 함수형 인터페이스에 박싱된 기본타입을 넣어 사용하진 말자.**
  - 동작하긴 하지만 성능이 처참히 느려진다.

# 표준 함수형 인터페이스 말고 직접 작성해야 할 때는?

표준 인터페이스 중 필요한 용도에 맞는게 없다면 직접 작성해야 한다. (당연)

(ex) 매개변수 3개를 받는 Predicate, 검사 예외를 던지는 경우

## 대표적 예시인 Comparator<T>

구조적으로는 `ToIntBiFunction<T, U>` 와 동일하지만 사용할 수 없다.

1. API에서 굉장히 자주 사용되는데, Comparator 라는 이름이 용도를 아주 훌륭이 설명해준다.
2. 구현하는 쪽에서 반드시 지켜야 할 규약을 담고있다.
3. 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 담고 있다.

## 특징

이 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는건 아닌지 고민해보자.

1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
2. 반드시 따라야 하는 규약이 있다.
3. 유용한 디폴트 메서드를 제공할 수 있다.

## 주의

자신이 작성하는 것은 ‘인터페이스'라는 걸 잊지말자. 아주 주의해서 설계해야 한다는 뜻이다.

# @FunctionalInterface

@Override를 사용하는 이유와 비슷하다.

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 해당 인터페이스가 람다용으로 설계된 것임을 알려준다.
2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일 되게 해준다.
3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

**직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하자.**

# 함수형 인터페이스를 API에서 사용할 때의 주의점

1. 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의하지 말라.
   - (ex) ExecutorService의 submit 메서드는 Callable<T>를 받는것과 Runnable을 받는 것을 다중정의 했다. → 형변환 해야 할 때가 왕왕 생긴다. (아이템 52)

# 정리

- 자바가 람다를 지원함으로써 입력값과 반환값에 함수형 인터페이스 타입을 활용하자.
- 보통은 표준 함수형 인터페이스를 사용하자.
