```java
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }

    public static void main(String[] args) {
				//1. 비추방법
        Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant p : garden) {
            plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
        }

				//2. 추천방법
				Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
      	
      	for(Plant.LifeCycle lc : Plant.LifeCycle.values())
          	plantsByLifeCycle.put(lc, new HashSet<>());
      
      	for (Plant p : garden)
          	plantsByLifeCycle.get(p.lifeCycle).add(p);
        }

				//3. 스트림을 사용한 추천방법
				Arrays.stream(garden)
					.collect(groupingBy( p -> p.lifeCycle,
						() -> new EnumMap<>(LifeCycle.class), toSet())));
    }
}
```

## 비추방법

- 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야하고 깔끔히 컴파일 되지 않음.
- 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 함.
- 정수값 사용으로 인해 타입이 안전하지 않음.

## 추천방법

- 짧고 명료함
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하여 레이블 달 필요 없음.

## 스트림을 사용한 추천방법

- 매개변수 3개짜리 Collectors.groupingBy 메서드를 사용하여 mapFactory매개변수에 원하는 맵 구현체를 명시해 호출함
- EnumMap 자체를 사용했을때랑 다른점이, 해당 생애주기에 속하는 식물이 있을 때만 만들게 된다.

# 중첩 EnumMap 사용 예시

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

//        // 코드 37-7 EnumMap 버전에 새로운 상태 추가하기 (231쪽)
//        SOLID, LIQUID, GAS, PLASMA;
//        public enum Transition {
//            MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
//            BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
//            SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
//            IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

“이전 상태에서 ‘이후 상태에서 전이로의 맵'에 대응시키는 맵” 이다.

## 로직

1. 첫번째 groupingBy에서는 전이를 이전 상태를 기준으로 묶는다.
2. toMap에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성한다.
    1. 병합 함수인 `(x, y) → y` 는 선언만 하고 실제 쓰이진 않음 → EnumMap을 얻으려면 맵 팩터리가 필요하고, 수집기들은 점층적 팩터리를 제공하기 때문
    
    ```java
    Collector<T, ?, M> toMap(
      Function<? super T, ? extends K> keyMapper,
      Function<? super T, ? extends U> valueMapper,
      BinaryOperator<U> mergeFunction,
      Supplier<M> mapSupplier)
    ```
    

# 정리

- 배열의 인덱스를 얻기위해 EnumMap을 사용하라.
- 다차원 관계는 EnumMap<…, EnumMap<…>> 으로 표현하라.