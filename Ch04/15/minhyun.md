# 캡슐화

잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.

## 장점

- 시스템 개발 속도를 높인다.
  - 여러 컴포넌트를 병렬로 개발할 수 있기 때문
- 시스템 관리 비용을 낮춘다.
  - 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 교체 부담도 적다.
- 성능 최적화에 도움을 준다.
  - 다른 컴포넌트에 영향을 주지않고 해당 컴포넌트만 최적화 할 수 있기 때문
- 소프트웨어 재사용성을 높인다.
  - 독자적으로 동작하는 컴포넌트라면 낯선 환경에서도 유용하게 쓰일 가능성이 크다.
- 큰 시스템을 제작하는 난이도를 낮춰준다.
  - 전체가 완성되지 않았어도, 개별 컴포넌트의 동작을 검증할 수 있기 때문

## 핵심

요소를 선언하는 위치와 접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심이다.

## 기본 원칙

- 항상 가장 낮은 접근 수준을 부여해야 한다.
  - 패키지 외부에서 쓸 이유가 없다면 package-private로 선언하자.
- 한 클래스에서 사용하는 private 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 private static으로 중첩시키자 (아이템 24)
  - private static으로 중첩시키면 바깥 클래스 하나에서만 접근할 수 있다.
- public일 필요가 없는 클래스의 접근 수준을 private 톱레벨 클래스로 좁혀야한다.
  - private 톱레벨 클래스는 내부 구현에 속하기 때문

### 접근 범위

- **private**
  - 멤버를 선언한 톱레벨 클래스에서만 접근 할 수 있다.
- **package-private**
  - 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다. (접근 제한자를 명시하지 않았을때 default 적용)
- **protected**
  - 선언한 클래스의 하위 클래스에서도 접근할 수 있다.
- **public**
  - 모든 곳에서 접근할 수 있다.

책내에서는 package-private 얘기가 많이나오는데 사실상 실무에서 거의 사용하지 않는다. 무조건 private로 설정하는것으로 이해하자. (본인도 요약할때 그냥 private로 쓸것임)

### 규칙

1. 공개 API를 잘 설계한 후, 그 외 모든 멤버는 private로 만든다.
2. 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여 package-private로 푼다.
   - 이때 권한을 푸는 빈도가 높다면 컴포넌트를 더 분해해야하는지 고민할 것
   - 보통은 공개 API에 영향을 주지 않으나, `Serializable` 을 구현한 클래스에서 의도치 않게 공개 API가 될 수 있다.
3. 리스코프 치환 원칙에 따라 클래스는 인터페이스가 정의한 모든 메서드를 public으로 선언해야 한다.
   - 리스코프 치환 원칙이란, 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 규칙이다.
4. 테스트만을 위해 클래스, 인터페이, 멤버를 공개 API로 만들어서는 안된다.
5. public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.
   - public 가변 필드를 갖는 클래스는 일반적으로 스레드에 안전하지 않다.
   - 구성요소의 상수라면 `public static final` 필드로 공개해도 된다. 관례상 대문자 알파벳에 각 단어 사이에 언더바를 넣는 컨벤션을 사용한다. 무조건 기본 타입값이나 불변 객체를 참조해야 한다. 또한 접근자 메서드를 제공하면 안된다.
     - 길이가 0이 아닌 배열은 모두 변경 가능함

### 배열의 상수화 문제점과 해결

배열의 경우 필드의 참조를 반환하기 때문에 아무리 final을 걸었다 하더라도 값 수정이 가능하다. 이러한 부분을 해결해보자.

```java
//문제가 있는 코드
public static final Thing[] VALUES = { ... };

//해결법 1 (public 불변 리스트를 추가하는 방식)
private static final Thing[] SOLUTION_VALUES1 = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

//해결법 2 (복사본을 반환하는 public 메서드)
private static final Thing[] SOLUTION_VALUES2 = { ... };
public static final Thing[] values() {
	return PRIVATE_VALUES.clone();
}
```

- 첫번째 방식의 경우 `Arrays.asList()` 의 경우 불변 처리가 되는데 해당 글은 [본인의 블로그글](<https://mintheon.com/devlog/2021/04/13/List.add()%EC%8B%9C-UnsupportedOperationException-%EB%B0%9C%EC%83%9D-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0/>) 을 참고하자.
- 추가적으로 `unmodifiableList` 도 불변 배열을 만들어 준다. 해당 부분은 [여기](https://lts0606.tistory.com/133)를 참고.

개인적으로 두번 감싼 이유가 조금 궁금함.

# 모듈

자바9에서는 모듈 시스템이라는 개념이 도입되었다. 모듈은 자신에 속하는 패키지 중 공개(export)할 것들을 (관례상module-info.java 파일에) 선언한다.

public, protected라도 해당 패키지를 공개하지 않았따면 모듈 외부에서는 접근할 수 없다. (내부는 노상관)

→ 암묵적으로 기존 접근 수준 외에 추가되었다.

패키지들을 모듈 단위로 묶고, 모듈 선언에 패키지들의 모든 의존성을 명시한다. 그후 소스트리를 재배치하고 모듈 안으로부터 일반 패키지로의 모든 접근에 특별한 조치를 취한다.

최근 많이 사용하고 있는 멀티 모듈을 참고할 것 → [멀티모듈 설계 이야기](https://techblog.woowahan.com/2637/)

# 정리

- 프로그램 요소의 접근성은 가능한 최소한으로 하라.
- public 클래스는 상수용 필드외에는 어떤 public 필드도 가져선 안된다. 또한 상수 객체가 참조하는 객체가 불변인지 확인하라.