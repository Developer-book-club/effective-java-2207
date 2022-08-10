# compareTo

아래의 두가지만 뺴면 `Object` 의 `equals` 와 같다.

1. 단순 동치성 비교에 순서까지 비교할 수 있다.
2. 제네릭하다.

`Comparable` 을 구현한 클래스의 인스턴스에는 자연적인 순서가 있음을 뜻하므로, 해당 객체들의 배열은 정렬이 가능하다.

```java
Arrays.sort(a);
```

또한 검색, 극단값 계산, 자동 정렬 컬렉션 관리도 쉽게 할 수 있다.

## 일반 규약

실제로 `equals`의 규약과 비슷하다.

- 객체와 주어진 객체의 순서를 비교한다.
  - 해당 객체 < 주어진 객체 → 음의 정수
  - 해당 객체 == 주어진 객체 → 0
  - 해당 객체 > 주어진 객체 → 양의 정수
  - 비교할 수 없는 타입의 객체 → ClassCastException
- (필수는 아니지만 꼭 지키자) `(x.compareTo(y) == 0) == (x.equals(y))` 여야 한다.
  - 정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용한다.
- 이후는 equals랑 비슷

## 예시

- 정렬된 컬렉션
  - TreeSet, TreeMap
- 검색과 정렬 알고리즘을 화룡하는 유틸리티 클래스
  - Collections, Arrays

## 작성 요령

1. 입력 인수의 타입을 확인하거나 형변환 할 필요가 없다.
   1. `Comparable` 은 타입을 인수로 받는 제네릭 인터페이스라 `compareTo` 메서드의 인수타입은 컴파일 타임에 정해진다.
   2. 타입이 잘못되었땀녀 컴파일 자체가 되지 않고, null을 인수로 넣어 호출하면 NPE가 던져진다.
2. `compareTo` 메서드는 필드가 동치인지를 비교하는게 아니라 순서를 비교한다.
   1. `Comparable` 을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(`Comparator`)를 대신 사용하자.

      ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0072a7ea-5ffe-4b21-9d1a-d7907f1dd310/Untitled.png)

자바 8에서는 `Comparator` 인터페이스를 메서드 체이닝 방식으로 비교자를 생성할 수 있다. 코드상 멋질 수 있으나 약간의 성능 저하가 있다.

흥미로운 부분인데, 자바의 정적 임포트 기능을 이용하면 정적 비교자 생성 메서드들을 이름만으로 사용할 수 있어 코드가 훨씬 깔끔해진다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
	comparingInt((PhoneNumber pn) -> pn.areaCode)
	.thenComparingInt(pn -> pn.prefix)
	.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```

⇒ 해당 방식은 3depth의 정렬 프로세스를 처리중이다. 첫번째 비교자를 적용한 다음 동일 데이터가 있을 가능성이 있어 두번째, 세번째 비교자를 적용한다.

# 정리

- 순서를 고려해야하는 값 클래스 작성시 `Comparable` 인터페이스를 구현하자.
- `compareTo` 메서드에서 필드의 값을 비교할때 <와 > 연산자는 쓰지 말고 정적 compare 메서드나 Comparator 인터페이슥 ㅏ제공하는 비교자 생성 메서드를 사용하자.
