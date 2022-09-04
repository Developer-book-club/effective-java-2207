함수 객체를 람다보다 더 간결하게 만드는 ‘메서드 참조'가 존재한다.

```java
// 람다 사용
map.merge(key, 1, (count, incr) -> count + incr);

// 메서드 참조 사용
map.merge(key, 1, Integer::sum);
```

- 자바 8이 되면서 Integer 클래스에서는 정적 메서드 `sum()` 을 지원하여, 해당 메서드를 메서드 참조로 처리했다.
- 람다로 할 수 없는 일이면 메서드 참조로도 할 수 없다.

# 코드의 간결함에 따라 선택하라

- 때론 람다가 메서드 참조보다 간결할 때가 있다.
  - 주로 메서드와 람다가 같은 클래스에 있을 때

```java
// 클래스 안에서 호출할 때

service.execute(GoshThisClassNameIsHumongous::action);

service.execute(() -> action());
```

이런 부분에서는 람다쪽이 훨씬 보기 좋다.

# 메서드 참조 유형

| 메서드 참조 유형    | 예                     | 같은 기능을 하는 람다         |
| ------------------- | ---------------------- | ----------------------------- |
| 정적                | Integer::parseInt      | str → Integer.parseInt(str)   |
| 한정적(인스턴스)    | Instant.now()::isAfter | Instant then = Instant.now(); |
| t → then.isAfter(t) |
| 비한정적(인스턴스)  | String::toLowerCase    | str → str.toLowerCase()       |
| 클래스 생성자       | TreeMap<K,V>::new      | () → new TreeMap<K,V>()       |
| 배열 생성자         | int[]::new             | len → new int[len]            |

- **한정적 인스턴스 메서드 참조**
  - 수신 객체를 특정함
  - 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다. (정적참조와 비슷)
- **비한정적 인스턴스 메서드 참조**
  - 수신 객체를 특정하지 않음
  - 함수 객체를 적용하는 시점에 수신 객체를 알려준다.
  - 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다.

# 정리

- 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.

# 추가

**람다로는 불가능하나 메서드 참조로는 가능한 유일한 예는 바로 제네릭 함수타입 구현**이다.

함수형 인터페이스의 추상 메서드가 제네릭일 수 있듯이 함수 타입도 제네릭일 수 있다.

```java
interface G1 {
	<E extends Exception> Object m() throws E;
}

interface G2 {
	<F extends Exception> String m() throws Exception;
}

interface G extends G1, G2 {}

// 이떄 함수형 인터페이스 G를 함수 타입으로 표현한다면
<F extends Exception> () -> String throws F
```

→ 사실 아직 이해하기 어렵다….
