# 거듭제곱 값을 할당한 정수 열거 패턴

열거할 값들이 주로 집합으로 사용될 경우, 거듭제곱 값을 할당한 정수 열거 패턴(아이템 34)을 사용해왔다.

```java
public class Text {
  public static final int STYLE_BOLD = 1 << 0; //1
  public static final int STYLE_ITALIC = 1 << 1; //2
  ...
  public static final int STYLE_STRIKETHROUGH = 1 << 3; //8
  
  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR 한 값이다.
  public void applyStyles(int styles) { ... }
}
```

- 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있었다.

## 단점

- 정수 열거 상수의 단점을 그대로 지닌다.
- 비트 필드 값이 그대로 출력 되면 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어렵다.
- 비트필드 하나에 녹아있는 모든 원소를 순회하기 까다롭다.
- 최대 몇비트가 필요한지를 API 작성시 미리 예측하여 적절한 타입(보통은 int나 long)을 선택해야 한다.
    - API를 수정하지 않곤 비트수를 더 늘릴 수 없기 때문.

# EnumSet

- EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해 준다.
- 내부는 비트 벡터로 구현되어 있어 원소가 총 64개 이하라면(대부분의 경우) EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.

## 기존의 정수 열거 패턴을 다시 구현시킨 EnumSet

```java
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
	
	// 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
	public void applyStyles(Set<Style> styles) { ... }
}

//인스턴스를 건네는 코드
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

EnumSet 은 RegularEnumSet , JumboEnumSet 으로 구현되며 길이가 64개 이하일때는 long 타입 비트로 표현되는 RegularEnumSet, 그 이상일 때는 long array로 표현되는 JumboEnumSet 이 사용된다.

```java
if (universe.length <= 64)
    return new RegularEnumSet<>(elementType, universe);
else
    return new JumboEnumSet<>(elementType, universe);
```

아래와 같이 `allOf` 를 사용해서 Enum 안에 있는 모든 변수를 담을 수 있다.

```java
EnumSet<Color> set = EnumSet.allOf(Color.class);
```

또한 EnumSet 을 사용할 때는 아래 내용을 참고해야한다.

---

열거형 값만 포함할 수 있으며 모든 값은 동일한 열거형에 속해야 합니다.

null 값을 추가하는 것을 허용하지 않으며 그렇게 하려는 시도에서 NullPointerException이 발생합니다.

스레드로부터 안전하지 않으므로 필요한 경우 외부에서 동기화해야 합니다.

요소는 열거형에서 선언된 순서에 따라 저장됩니다.

복사본에서 작동하는 안전 장치 반복자를 사용하므로 컬렉션을 반복할 때 컬렉션이 수정되는 경우 ConcurrentModificationException이 발생하지 않습니다.

참고) [https://www.baeldung.com/java-enumset](https://www.baeldung.com/java-enumset)

---

# 정리

- 열거형을 집합 형태로 사용할 땐 EnumSet을 사용하자.
- 유일한 단점이라면 자바 11까지는 불변 EnumSet을 만들 수 없다.
    - Immutable 하게 하려면, Collections.unmodifiableSet 으로 래핑해줘야한다.

---

### 참고

[https://blog.naver.com/dmswldla91/222821160643](https://blog.naver.com/dmswldla91/222821160643)