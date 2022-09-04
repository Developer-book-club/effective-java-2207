어렵다..!

기존에는 Collection 메서드를 구현할 수 없을 때 Iterable 인터페이스를 사용하였으나, 자바 8에서 스트림이라는 개념이 생기며 복잡해져버렸다.

스트림은 iteration을 지원하지 않는다.

> 사실 Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고, 정의한 방식대로 동작하지만 iteration이 불가한 까닭은 Stream이 Iterable을 확장(extned) 하지 않아서이다.
> 

# Iterable로 적절히 형변환한 방법

```java
public class Adapters {
    // 코드 47-3 Stream<E>를 Iterable<E>로 중개해주는 어댑터 (285쪽)
    public static <E> Iterable<E> iterableOf(Stream<E> stream) {
        return stream::iterator;
    }

    // 코드 47-4 Iterable<E>를 Stream<E>로 중개해주는 어댑터 (286쪽)
    public static <E> Stream<E> streamOf(Iterable<E> iterable) {
        return StreamSupport.stream(iterable.spliterator(), false);
    }
}
```

## 반환 타입으로 컬렉션을 사용해야하는 이유

- 공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 고려해야함.
- Collection 인터페이스는 Iterable의 하위 타입이고 Stream 메서드도 제공하여 반복과 스트림을 동시에 지원한다.
    - 그리하여 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다.

# 전용 컬렉션

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토하자.

집합의 멱집함을 반환하는 로직 구현시, 원소 개수가 n개면 멱집합의 원소 개수는 2^n개가 되기 때문에 표준 컬렉션에 저장하는 것은 위험하다.

아래 예시는 각 원소의 인덱스를 비트벡터로 사용하도록 하는 전용 컬렉션이다.

```java
public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException(
                "집합에 원소가 너무 많습니다(최대 30개).: " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
                return 1 << src.size();
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
```

### 대신…

입력 리스트의 부분 리스트를 모두 반환하는 메서드를 작성했을 경우 표준 컬렉션에 담는 코드는 단 3줄이면 구현이 가능하다.

대신 해당 컬렉션은 입력 리스트 크기의 거듭제곱만큼 메모리를 차지한다. (부분 리스트이다 보니..)

그렇다고 전용 컬렉션을 구현하긴 그렇다 보니 약간의 통찰을 통해 스트림으로 구현하자.

# Stream 구현

```java
public class SubLists {
    // 코드 47-6 입력 리스트의 모든 부분리스트를 스트림으로 반환한다. (288-289쪽)
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

- 어떤 리스트의 부분 리스트는 단순히 그 리스트의 프리픽스의 서픽스 (혹은 서픽스의 프리픽스)에 빈 리스트 하나만 추가하면 된다.

# 그래서

하지만 이런 어댑터는 클라이언트 코드를 어수선하게 만들고 속도도 2.3배 더 느리다.

직접 구현한 전용 Collection을 사용하니 코드는 지저분 해졌지만 스트림을 활용한 구현보다 1.4배 빨랐다.

# 정리

- 원소 시퀀스를 반환하는 메서드를 작성할 때는 Iterator와 Stream 모두를 신경쓰자.
- 컬렉션을 반환할 수 있다면 그렇게 하라.
- 컬렉션을 반환하는게 불가능하면 Stream과 Iterable중 더 자연스러운 것을 반환하라.