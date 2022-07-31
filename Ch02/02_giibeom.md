# 목차
- [item-01. 생성자 대신 정적 팩터리 메서드를 고려하라](#item-01.-생성자-대신-정적-팩터리-메서드를-고려하라)
- [item-02. 생성자에 매개변수가 많다면 빌더를 고려하라](#item-02-생성자에-매개변수가-많다면-빌더를-고려하라)
- [item-03. private 생성자나 열거 타입으로 싱글턴임을 보증하라](#item-03-private-생성자나-열거-타입으로-싱글턴임을-보증하라)




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

<br>
<br>


# [item-02] 생성자에 매개변수가 많다면 빌더를 고려하라

- 정적 팩터리와 생성자는 동일하게 선택적 매개변수가 많을 때 적절히 대응하기 어렵습니다
- 매개변수가 많을때의 생성자, 정적 팩터리의 모습을 알아봅시다

<br>

## 1. 점층적 생성자 패턴(telescoping constructor pattern)

- 필수 매개변수만 받는 생성자와 필수 매개변수 + 선택 매개변수 1개, 2개, 3개….. 식으로 생성자를 늘려나가는 방식입니다
- 예제는 아래와 같습니다

    ```java
    public class NutritionFacts {
        private final int servingSize;  // (mL, 1회 제공량)     필수
        private final int servings;     // (회, 총 n회 제공량)  필수
        private final int calories;     // (1회 제공량당)       선택
        private final int fat;          // (g/1회 제공량)       선택
        private final int sodium;       // (mg/1회 제공량)      선택
        private final int carbohydrate; // (g/1회 제공량)       선택
    
    // 필수 파라미터 생성자
        public NutritionFacts(int servingSize, int servings) {
            this(servingSize, servings, 0);
        }
    
    // 필수 파라미터 + 칼로리 생성자
        public NutritionFacts(int servingSize, int servings,
                              int calories) {
            this(servingSize, servings, calories, 0);
        }
    
    // 필수 파라미터 + 칼로리 + 지방 생성자
        public NutritionFacts(int servingSize, int servings,
                              int calories, int fat) {
            this(servingSize, servings, calories, fat, 0);
        }
    
    // 모든 파라미터 생성자
        public NutritionFacts(int servingSize, int servings,
                              int calories, int fat, int sodium) {
            this(servingSize, servings, calories, fat, sodium, 0);
        }
        public NutritionFacts(int servingSize, int servings,
                              int calories, int fat, int sodium, int carbohydrate) {
            this.servingSize  = servingSize;
            this.servings     = servings;
            this.calories     = calories;
            this.fat          = fat;
            this.sodium       = sodium;
            this.carbohydrate = carbohydrate;
        }   
    }
    ```

<br>

### 점층적 생성자 패턴의 단점

- 점층적 생성자 패턴은 매개변수 개수가 많아질수록 클라이언트 코드를 작성하거나 읽기 어렵습니다 → 가독성 ↓
- 위의 예시를 생성자를 통해 인스턴스를 생성할 때 어느 자리에 무슨 인자를 넣어야 할지 파악하기 어렵습니다

    ```java
    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
             // 인자의 240은 뭐고 8은 뭐고 나머지도 다 무슨 값을 넣어야하나..
    }
    ```

<br>

## 2. 자바빈즈 패턴(JavBeans pattern)

- 자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만든 후 세터(setter) 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식입니다
- 예제는 아래와 같습니다

    ```java
    public class NutritionFacts {
        // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
        private int servingSize  = -1; // 필수; 기본값 없음
        private int servings     = -1; // 필수; 기본값 없음
        private int calories     = 0;
        private int fat          = 0;
        private int sodium       = 0;
        private int carbohydrate = 0;
    
    		// 기본 생성자
        public NutritionFacts() { }
    
        // Setters
        public void setServingSize(int val)  { servingSize = val; }
        public void setServings(int val)     { servings = val; }
        public void setCalories(int val)     { calories = val; }
        public void setFat(int val)          { fat = val; }
        public void setSodium(int val)       { sodium = val; }
        public void setCarbohydrate(int val) { carbohydrate = val; }
    }
    ```

<br>

### 자바빈즈 패턴의 장점

- 점층적 생성자 패턴의 단점은 보완이 됐습니다 → 어떤 매개변수인지 확인 가능
- 코드가 길어졌지만 인스턴스를 만들기 쉽고 가독성이 좋아져 코드가 읽기 쉬워집니다

    ```java
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
    ```

<br>

### 자바빈즈 패턴의 단점

- 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓입니다
- 따라서 스레드 안정성을 얻으려면 추가 작업이 필요합니다
- 이러한 단점을 완화하고자 freezing 기법을 사용하지만 다루기 어려워 실전에서 거의 사용되지 않습니다
    - freezing: 생성이 끝난 객체를 수동으로 얼리고(freezing) 얼리기 전에는 객체 사용 불가

<br>

## 3. 빌더 패턴

- 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비한 방식입니다
1. 필수 매개변수만으로 생성자 or 정적 팩터리를 호출해 빌더 객체를 얻습니다
2. 그리고 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정합니다
3. 매개변수 설정이 끝났을 경우 build() 메서드를 통해 객체를 반환합니다

    ```java
    NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100).sodium(35).carbohydrate(27).build();
    ```

    ```java
    public class NutritionFacts {
        private final int servingSize;
        private final int servings;
        private final int calories;
        private final int fat;
        private final int sodium;
        private final int carbohydrate;
    
        public static class Builder {
            // 필수 매개변수
            private final int servingSize;
            private final int servings;
    
            // 선택 매개변수 - 기본값으로 초기화한다.
            private int calories      = 0;
            private int fat           = 0;
            private int sodium        = 0;
            private int carbohydrate  = 0;
    
            public Builder(int servingSize, int servings) {
                this.servingSize = servingSize;
                this.servings    = servings;
            }
    
            public Builder calories(int val)
            { calories = val;      return this; }
            public Builder fat(int val)
            { fat = val;           return this; }
            public Builder sodium(int val)
            { sodium = val;        return this; }
            public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }
    
            public NutritionFacts build() {
                return new NutritionFacts(this);
            }
        }
    
        private NutritionFacts(Builder builder) {
            servingSize  = builder.servingSize;
            servings     = builder.servings;
            calories     = builder.calories;
            fat          = builder.fat;
            sodium       = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }
    ```

<br>

### 빌더 패턴의 장점

- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋습니다
- 빌더를 이용하면 가변인수 매개변수를 여러 개 사용할 수 있습니다
- 빌더 하나로 여러 객체를 순회하면서 만들 수 있고 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수 있는 등 상당히 유연합니다
- 객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수도 있습니다

<br>

### 빌더 패턴의 단점

- 객체를 만들려면 그에 앞서 빌더부터 만들어야 됩니다
    - 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있습니다
- 점층적 생성자 패턴보다는 코드가 장황합니다 (클라이언트의 코드는 간결합니다)
    - 따라서 매개변수가 4개 이상(?)은 되어야 값어치를 합니다
    - 또한 매개변수가 많아지거나 늘어날 가능성이 있을 경우 빌더로 시작하는걸 추천합니다

<br>

### 핵심 정리

- 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 것을 추천합니다
- 특히 선택적 매개변수가 대부분이거나 매개변수끼리의 타입이 같다면 더욱 빌더 패턴을 추천합니다
- 빌더 패턴은 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하며, 쓰레드 안전성으로부터 자바빈즈보다 훨씬 안전합니다


<br>
<br>

# [item-03] private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말합니다
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어렵다는 단점이 있습니다
    - 싱글턴이 인스턴스를 구현한게 아니라면 가짜(mock) 구현으로 대체할 수 없기 때문입니다

<br>

## 싱글턴을 만드는 방식

- 싱글턴을 만드는 방식은 두가지입니다
- 두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해 둡니다

<br>

### 1. final 필드 방식

- public static 멤버가 final 필드인 방식입니다
- 아래는 예제입니다

    ```java
    // 싱글턴 객체
    
    public class Singleton {
    
    // 참조변수 instance로만 접근하게 하여 하나의 인스턴스로만 공유해서 사용합니다
        public static final Singleton instance = new Singleton();
    
    // Singleton 객체의 생성자는 private으로 막아놓습니다
        private Singleton() {
        }
    }
    ```
    
    ```java
    public class App {
    
        public static void main(String[] args) {
                    Singleton singleton1 = new Singleton(); // 컴파일 에러
    
                // 오로지 하나의 인스턴스로만 사용할 수 있습니다
            Singleton singleton2 = Singleton.instance;
        }
    }
    ```

- 다만 권한이 있는 클라이언트에서 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 경우 싱글턴이 뚫릴 수 있습니다

<br>

#### 리플렉션 API란?

- 리플렉션 API는 구체적인 클래스 타입을 알지 못해도 해당 클래스의 정보(메서드, 타입, 변수 등)에 접근할 수 있게 도와주는 자바 API입니다
- 간단하게만 예제를 작성해 보겠습니다

    ```java
    public class App {
    
        public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    
    // 위 예제에서 instance 를 통한 단일 인스턴스를 얻어왔다
            Singleton singleton1 = Singleton.instance;
    
            Class<?> singletonClass = Class.forName("Template.Singleton");
            Constructor<?> constructor = singletonClass.getDeclaredConstructor();
            constructor.setAccessible(true);
    // 이렇게 하면 private 생성자를 호출 할 수 있다 -> 싱글턴이 깨지게 된다
            Singleton singleton2 = (Singleton) constructor.newInstance();
    
    				System.out.println("singleton1 주소값: " + singleton1);
            System.out.println("singleton2 주소값: " + singleton2);
        }
    }
    
    /*
        [출력 결과]
        singleton1 주소값: Template.Singleton@1554909b
        singleton2 주소값: Template.Singleton@6bf256fa
    */
    ```

<br>

#### final 필드 방식 장점

- 해당 클래스가 싱글턴임이 API에 명백히 드러납니다
- 간결합니다

<br>

### static 팩터리 메서드 방식

- final 방식과 다르게 유일한 인스턴스를 갖고있는 참조변수 instance를 private으로 설정합니다
- instance를 반환하는 정적 팩터리 메서드(public static)를 선언합니다
- 아래는 예제입니다

    ```java
    public class Singleton {
        private static final Singleton instance = new Singleton();
    
        private Singleton() {
        }
    
        public static Singleton getInstance() {
            return instance;
    			
    // 이렇게 한다면 클라이언트 코드를 바꾸지 않고 싱글턴이 아니게 변경할 수 있습니다
    //			return new Singleton();
        }
    }
    ```

<br>

#### static 팩터리 메서드 방식 장점

- 원한다면 API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있습니다
    - 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있습니다
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있습니다
    - `ex) Supplier<Singleton> s2supplier = Singleton::getInstance;`
- 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있습니다

<br>

## 싱글턴 클래스 직렬화

- 위 1, 2번 방식을 통해 싱글턴 클래스 직렬화를 하려면 단순히 Serializable 선언으로는 부족합니다
- 모든 인스턴스 필드를 일시적이라고 선언하는 transient를 추가 (직렬화 하지 않겠다는 뜻)하고, readResolve 메서드를 제공해야 합니다
    - 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화 할 때마다 새로운 인스턴스가 만들어집니다

    ```java
    private Object readResolve() {
        return INSTANCE;
    }
    ```

<br>

## Enum 방식

- 원소가 하나인 열거타입을 선언하는 방식

    ```java
    public enum Elvis {
    	INSTANCE;
    }
    ```

- 직렬화/역직렬화 상황이나 리플렉션 공격에도 제 2의 인스턴스가 생성되는 일을 완벽히 막아줍니다
- 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법입니다