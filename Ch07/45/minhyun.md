스트림은 다량의 데이터 처리작업을 돕고자 자바 8에 추가되었다.

# 핵심 개념

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

# 스트림 파이프라인

1. 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간연산이 있을 수 있다.
2. 각 중간연산은 스트림을 어떠한 방식으로 변환한다.
3. 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있따.
4. 종단 연산은 마지막 중간연산이 내놓은 스트림에 최후의 연산을 가한다.

## 지연 평가

스트림 파이프 라인은 지연 평가(lazy evaluation) 된다.

평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.

## 플루언트 API

메서드 연쇄를 지원하는 플루언트 API(fluent API) 이다.

→ 파이프 라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다.

파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 `parallel` 메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황은 많지 않다.

# 과도한 스트림 사용 예시

사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력하는 코드이다.

## 1. 기본 자바8 메서드 사용

```java
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

- `computeIfAbsent` 를 사용하여, 키가 있따면 해당 키에 매핑된 값을 반환, 키가 없다면 건네진 함수 객체를 키에 적용하여 값을 계산하고, 계산된 값을 반환한다.
- 해당 메서드를 사용하여 각 키에 다수의 값을 매핑하는 맵을 쉽게 구현가능하다.

## 2. 스트림을 과하게 사용 (사용X)

```java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

- 짧지만 읽기 어렵다.
- 스트림을 과용하면 프로그램이 읽거나 유지보수 하기 어려워진다.

## 3. 적절한 스트림의 활용

```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

- alphabetize 메서드도 스트림으로 다르게 구현할 수 있으나, 명확성이 떨어지고 잘못 구현할 가능성이 커진다. 특히나 느려질 수도 있다.
    - 자바가 char용 스트림을 지원하지 않기 떄문

## 추가로 주의할 점

1. 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
2. 도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복 코드에서보다 스트림 파이프라인에서 훨씬 크다.
3. **기존 코드는 스트림을 사용하도록 리팩터링 하되, 새 코드가 더 나아보일 때만 반영하자.**
    1. 모든 반복문을 스트림으로 바꾸고 싶겠지만 가독성과 유지보수 측면에서 손해를 볼 수 있다.

# 자바가 char용 스트림을 지원하지 않는 이유

```java
"Hello world!".chars().forEach(System.out::print);
// 결과값: 721011081081113211911111410810033
```

- `chars()` 가 반환하는 스트림의 원소는 char가 아니라 int값이기 때문
- (char)를 통한 형변환을 통해 처리할 수 있지만 char를 사용할 땐 그냥 스트림을 삼가는 편이 낫다.

# 함수 객체로는 할 수 없지만 코드 블록으로는 할 수 있는 일

- 범위 안의 지역 변수를 읽고 수정할 수 있다.
    - 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역 변수를 수정하는건 불가능
- return문을 사용해 메서드에서 빠자나가거나 반복문 종료 혹은 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다.
    - 람다는 하나도 할 수 없다.

# 스트림과 안성맞춤인 로직

- 원소들의 시퀀스를 일관되게 변환
- 원소들의 시퀀스를 필터링
- 원서들의 시퀀스를 하나의 연산을 사용해 결합 (더하기, 연결하기, 최솟값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모은다 (공통된 속성을 기준으로)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

# 스트림으로 처리하기 어려운 일

- 한 데이터가 파이프 라인의 여러 단계를 통과할 때 각 단계에서의 값들에 동시에 접근
    - 스트림 파이프라인은 한 값을 다른 값에 매핑하면 원래의 값은 잃는 구조이기 때문

# 참고

- 스트림을 반환하는 메서드 이름은 원소의 정체를 알려주는 복수 명사로 쓰자.
    - (ex) primes → 소수들

# 메르센 소수 예시

종단연사에서 매핑을 거꾸로 수행하는 예시를 알려주는데.. 어떻게 이런 생각을 할 수 있지..

```java
// 2진수 비트로 변환한 길이를 세어 지수를 출력한다.
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

# 카드 덱을 초기화하는 예시

```java
//기존은 2중 for문을 사용하여 돌리는 방식으로 진행하였으나 스트림 방식으로 변경
private static List<Card> newDeck() {
	return Stream.of(Suit.values())
		.flatMap(suit -> Stream.of(Rank.values())
												.map(rank -> new Card(suit, rank)))
		.collect(toList());
}
```

# 정리

- 스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라.