
# [item-01] 생성자 대신 정적 팩터리 메서드를 고려하라

## 클라이언트가 클래스의 인스턴스를 얻는 방법

1. public 생성자
2. 정적 팩터리 메서드(static factory method)

<br>

## 정적 팩터리 메서드란?

- 자바에서 객체 생성할 때 흔히 사용하는 생성자가 아닌 정적 메서드(static method)로 객체 생성을 할 수 있게 도와주는 것이 정적 팩터리 메서드입니다
1. 아래는 일반적인 생성자를 통한 객체 생성 방법입니다

    ```java
    // 카드 객체
    public class Card {
        String pattern;    // 무늬
        int number;     // 숫자
    
        public Card(String pattern, int number) {
            this.pattern = pattern;
            this.number = number;
        }
    }
    
    // 인스턴스 생성
    public class App {
        public static void main(String[] args) {
            Card heart7= new Card("heart", 7);
        }
    }
    ```

    - 카드 객체를 생성할 때 인자에 무엇을 집어넣어야 되는지 정확히 알 수 없습니다

<br>

2. 아래는 정적 팩터리 메서드를 통한 객체 생성 방법입니다

    ```java
    // 카드 객체
    public class Card {
        String pattern;    // 무늬
        int number;     // 숫자
    
    // 정적 팩터리 메서드
        public static Card createWithPatternAndNumber(String pattern, int number) {
            Card card = new Card();
            card.pattern = pattern;
            card.number = number;
            return card;
        }
    }
    
    // 인스턴스 생성
    public class App {
        public static void main(String[] args) {
            Card heart7 = Card.createWithPatternAndNumber("heart", 7);
        }
    }
    ```

    - 정적 팩터리 메서드를 통해 객체를 생성할 경우 어떤 인자를 집어넣는지 메서드 명으로 명확하게 알 수 있는 것을 볼 수 있습니다

<br>

## 정적 팩터리 메서드 장점

### 1. 이름을 가질 수 있다

- 이름만 잘 지으면 반환 될 객체의 특성을 쉽게 묘사할 수 있습니다

    ```markdown
    💡 예시를 들어보자
    
    소수인 값의 BigInteger를 반환하는 경우입니다!
    어떤 것이 더 명확한가요?
    1. BigInteger(int, int, Random)
    2. BigInteger.probablePrime(int, Random)
    
    반환 될 객체가 비교적 명확한 2번은 정적 팩터리 메서드입니다!
    ```

- 하나의 시그니처로는 하나의 생성자만 만들 수 있습니다

  `시그니처 = 메서드 명 + 메서드 매개변수의 타입들`

    - 따라서 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주면 됩니다

<br>

### 2. 호출될 때마다 인스턴스를 꼭 새로 생성하지는 않아도 된다

- 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 방식이 가능합니다 → 불필요한 객체 생성을 피할 수 있음

    ```java
    // 💡 예시를 들어보자
    
    // Boolean 클래스의 valueof 메서드는 객체를 아예 생성하지 않습니다
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
    ```

<br>

#### 인스턴스 통제 클래스

- 반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 인스턴스를 통제할 수 있습니다 → 언제 어느 인스턴스를 살아 있게 할지 통제 가능

<br>

#### 인스턴스를 통제한다면 어떤 장점이 있을까?

1. 클래스를 싱글턴으로 만들 수 있다
2. 인스턴스화 불가로 만들 수도 있다
3. 인스턴스가 단 하나뿐임을 보장할 수 있다

<br>

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다

- 반환할 객체의 클래스를 자유롭게 선택할 수 있게 합니다 → 엄청난 유연성을 보장
    - API 개발에 응용할 경우 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있습니다
- 쉽게 말해 반환 타입을 인터페이스만 노출하고 실제 반환값은 인터페이스의 구현체를 반환합니다
    - 클라이언트 입장에서는 인터페이스만 보고 개발하면 되는 편리함이 있습니다

<br>

#### Java 8 이전

- 인터페이스에 정적 메서드(static method)를 선언할 수 없었습니다
- 따라서 인스턴스화 불가인 동반 클래스를 만들어 그 안에 원하는 인터페이스를 반환하는 정적 메서드를 정의하는 관례가 있었습니다
    - `ex) 자바 컬렉션 프레임워크는 구현체 대부분이 단 하나의 인스턴스화 불가 클래스인 java.util.Collections에서 정적 팩터리 메서드를 틍해 얻도록 구현되어 있습니다`

<br>

#### Java 8 이후

- 인터페이스가 정적 메서드를 가질 수 있게 제한이 풀려 public 정적 멤버들을 인터페이스 자체에 두면 됩니다
- 다만 인터페이스에는 public 정적 멤버만 허용하기 때문에, 정적 메서드들을 구현하기 위한 코드 중 많은 부분은 여전히 별도의 package-private 클래스에 두어야 합니다
    - `접근제어자: private < package-private(default) < protected < public`

<br>

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없습니다
- 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없습니다 → 알 필요가 없어서 숨겨놓은 것
- 단지 해당 클래스의 하위 클래스이기만 하면 되는 것입니다

<br>

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

- 3, 4번과 비슷한 개념으로 이러한 유연함은 서비스 제공자 프레임워크의 근본입니다
- 서비스 제공자 프레임워크에서의 제공자(provider)는 **서비스 구현체**로써 이 구현체들을 클라이언트에게 제공하는 역할을 프레임워크가 통제합니다 → 클라이언트와 구현체 간의 분리 가능

<br>

#### 서비스 제공자 프레임워크

- 3개(+1개)의 핵심 컴포넌트로 이루어져 있습니다
1. 서비스 인터페이스
    - 구현체의 동작을 정의
2. 제공자 등록 API
    - 제공자가 구현체를 등록 할 때 사용
3. 서비스 접근 API  → `유연한 정적 팩터리의 실체`
    - 클라이언트가 서비스의 인스턴스를 얻을 때 사용
    - 클라이언트에서 원하는 구현체의 조건을 명시 가능
        - 조건 명시 안할 시 기본 구현체 반환 or 지원하는 구현체들을 하나씩 돌아가며 반환
4. (종종 사용) 서비스 제공자 인터페이스
    - 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명
    - 없다면 각 구현체를 인스턴스로 만들 때 리플렉션을 사용해야 함


```markdown
💡 예시를 들어보자

예시1) JDBC
Connection: 서비스 인터페이스 역할
DriverManager.registerDriver: 제공자 등록 API
DriverManager.getConnection: 서비스 접근 API
Driver: 서비스 제공자 인터페이스

예시2) 브리지 패턴
예시3) 의존 객체 주입(DI, 의존성 주입) -> 강력한 서비스 제공자
```

<br>

## 정적 팩터리 메서드 단점

### 1. public, protected 생성자 없이 정적 팩터리 메서드만 제공하는 클래스는 상속할 수 없다 (하위 클래스를 만들 수 없다)

- Collection 프레임워크의 구현 클래스들은 상속할 수 없다
- 오히려 상속보다 컴보지션을 사용하도록 유도하고, 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 장점으로 받아들일 수도 있다

<br>

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

- 생성자처럼 API 설명에 명확히 드러나지 않습니다
- 따라서 API 문서를 잘 써놓고 메서드 이름도 널리 알려진 규약을 따라 지어야 합니다

<br>

## 흔히 사용되는 정적 팩터리 메서드 명명규칙

- from
    - 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
    - `Date d = Date.from(instant);`
- of
    - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
    - `Set<Rank> faceCards = EnumSet.of(Jack, Queen, King);`
- valueOf
    - from과 of의 더 자세한 버전
    - `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
- instance or getInstance
    - 매개변수를 받으면 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않음
    - `StackWalker luke = StackWalker.getInstance(options);`
- create or newInstance
    - 위의 instance or getInstance와 동일하지만, 매번 새로운 인스턴스를 생성해 반환함을 보장함
    - Object newArray = Array.newInstance(classObject, arrayLen);
- getType
    - getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용
    - ‘Type’은 팩터리 메서드가 반환할 객체 타입
    - `FileStore fs = Files.getFileStore(path)`
- newType
    - newInstance와 같으며, getType과 동일한 상황에서 사용
    - ‘Type’은 팩터리 메서드가 반환할 객체 타입
    - `BufferedReader br = Files.newBufferedReader(path);`
- type
    - getType과 newType의 간결한 버전
    - `List<Complaint> litany = Collections.list(legacyLitany);`

<br>

## 핵심 정리

- 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상황에 맞게 사용하는 것이 좋습니다
- 하지만 각자의 장단점이 있고 각자의 쓰임새가 있다 하더라도 정적 팩터리를 사용하는게 유리한 경우가 더 많습니다
- 따라서 무작정 public 생성자를 제공하던 습관은 고쳐야됩니다