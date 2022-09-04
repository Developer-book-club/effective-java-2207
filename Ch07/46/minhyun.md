**스트림 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성 하는 것**이다.

각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.

> 순수함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다.
> 

```java
// 스트림 패러다임을 이해하지 못한 완전 틀려버린 로직
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
	words.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
});

// 제대로 활용한 로직
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
	freq = words
		.collect(groupingBy(String::toLowerCase, counting()));
}
```

- forEach는 스트림 계산 결과를 보고할때만 사용하고 계산하는 데는 쓰지 말자.
    - 가끔 계산 결과를 기존 컬렉션에 추가하는 등의 다른 용도로 쓸 순 있다.

# collector

- `java.util.stream.Collectors` 클래스
- 메서드를 39개나 가지고 있고, 그중 타입 매개변수가 5개나 되는 것도 있다.
- 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있따.
    - toList()
    - toSet()
    - toCollection(collectionFactory)

## 예시

```java
List<String> topTen = freq.keySet().stream()
                .sorted(comparing(freq::get).reversed()) // 요부분
                .limit(10)
                .collect(toList());
```

## toMap

스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.

```java
private static final Map<String, Operation? stringToEnum = 
	Stream.of(values()).collect(
		toMap(Object::toString, e -> e));
```

- 키 매퍼와 값 매퍼는 물론 병합 함수까지 제공할 수 있다.
    - 병합 함수의 형태는 BinaryOperator<U> 이며, U는 해당 맵의 값 타입이다.
    - 같은 키를 공유하는 값들은 해당 병합 함수를 사용해 기존 값에 합쳐진다.

```java
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

- 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다.

### `**toConcurrentMap**`

- 병렬 실행 된 후 결과로 ConcurrentHashMap 인스턴스를 생성한다.

### `**groupingBy**`

- 입력으로 분류 함수를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환

```java
// 아나그램 프로그램에서 사용한 수집기
words.collect(groupingBy(word -> alphabetize(word)));
```

- **리스트 외의 값을 갖는 맵을 생성하게 하려면 분류 함수와 함께 다운스트림 수집기도 명시해야 한다.**
    - 다운스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다.
    - `toSet()` 을 넘긴다면 원소들의 리스트가 아닌 Set을 값으로 갖는 맵을 만들어 낸다.

추가적으로 `groupingByConcurrent` 메서드도 볼 수있는데, 메서드의 동시 수행 버전으로 `ConcurrentHashMap` 인스턴스를 만들어 준다.

- **partitioningBy**
    - 분류 함수 자리에 Predicate를 받고 키가 Boolean인 맵을 반환한다.

### counting()

- 다운스트림 수집기 전용이다.
- 비슷한 메서드가 16개나 더있다.
    - (ex) summing, averaging, summarizing 등인데, 각 int, long ,double tㅡ트림용으로 하나씩 존재한다.

### minBy, maxBy

- 인수로 받은 비교자를 이용해 스트림에서 값이 가장 작은 혹은 큰 원소를 찾아 반환한다.

### joining

- CharSequence 인스턴스의 스트림에만 적용할 수 있다.
- 매개변수를 보내지 않으면 단순 연결하고, 구분 문자 매개변수를 준다면 각 연결 부위에 구분문자를 삽입한다.

# 정리

- 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다.
- forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다.
- 수집기를 잘 알아둬야한다. 가장 중요한 수집기는 toList, toSet, toMap, groupingBy, joining 이다.