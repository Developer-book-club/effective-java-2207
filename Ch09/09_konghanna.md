# 아이템 57: 지역변수의 범위를 최소화하라
- 지역변수의 유효 범위를 최소화하면 코드 가독성, 유지보수성이 높아지고 오류 가능성이 낮아짐

### 지역변수의 범위를 줄이는 방법
1. 지역변수는 사용할 때 선언하고 초기화할 것 
    - 예외: try-catch문
      - 변수 값을 try 블록 바깥에서도 사용해야 한다면 try 블록 앞에서 선언해야 함
2. 반복문은 while문보다 for문을 사용할 것 
    - while문 사용시 -> 변수가 반복문 밖에 선언되어 이전 while문에서 쓰였던 변수의 유효 범위가 새로운 while문까지 확장되기도 함 
    - for문 사용시 -> 변수의 유효 범위가 for문의 종료와 함께 끝남 (변수 유효 범위 = for문 범위)
3. 메서드를 작게 유지하고 한 가지 기능에 집중할 것
    - 여러 가지 기능을 처리한다면 그중 한 기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것임 -> 메서드를 기능별로 쪼갤 것

<br><br>

# 아이템 58: 전통적인 for 문보다는 for-each 문을 사용하라

### for문
- 반복자와 인덱스 변수를 사용함 -> 원소들만 얻으면 됨 
  - 코드가 지저분하고 사용되는 요소가 많아 오류 발생 가능성이 높음

### for-each문 (향상된 for문)
- 반복자와 인덱스 변수를 사용하지 않음
  - 코드가 깔끔하고 오류 발생 가능성이 낮음
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있음
```java
// 2중 for each
for (Suit suit : suits) {
    for (Rank rank : ranks) {
        deck.add(new Card(suit, rank));
    }
}
```
- 컬렉션을 중첩해 순회하는 경우 더욱 효과적
- 컬렉션, 배열, Iterable 인터페이스를 구현한 객체라면 무엇이든 순회 가능

### for-each문을 사용할 수 없는 상황
1. 파괴적인 필터링: 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 함
    - cf) Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하지 않아도 됨(자바8~)
2. 변형: 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 함
3. 병렬 반복: 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 함

<br><br>

# 아이템 59: 라이브러리를 익히고 사용하라

### 표준 라이브러리의 이점
1. 그 코드를 작성한 전문가의 지식과 경험을 활용할 수 있음
2. 핵심적인 일과 관련없는 시간 소비가 줄어듦
3. 따로 노력하지 않아도 성능이 지속해서 개선됨
4. 기능이 점점 많아짐
5. 표준 라이브러리를 활용한 코드는 많은 사람들에게 낯익은 코드가 됨

### 주요 라이브러리
- java.lang, java.util, java.io와 그 하위 패키지들
- 컬렉션 프레임워크, 스트림 라이브러리, java.util.concurrent의 동시성 기능 

<br><br>

# 아이템 60: 정확한 답이 필요하다면 float와 double은 피하라

### float, double 타입
- 과학, 공학 계산용, 넓은 범위의 수를 빠르게 정밀한 근사치로 계산하도록 설계됨 
- 0.1 또는 10의 음의 거듭 제곱 등을 표현할 수 없음
- 정확한 결과가 필요할 때는 사용하면 안 됨 ex) 금융 관련 계산

### BigDecimal, int, long 타입
- 금융 계산에 적합함
- BigDecimal
  - 기본 타입보다 쓰기가 불편하고 느림 -> 불편함, 성능 저하를 신경 쓰지 않을 경우 사용
  - 다양한 반올림 모드 제공 -> 반올림을 수행해야 하는 비즈니스 계산에서 편리함 
  - 열여덟 자리가 넘어가는 숫자 -> BigDecimal
- int, long
  - 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 함 -> 성능이 중요하고, 숫자가 너무 크지 않은 경우 사용
  - 아홉 자리로 표현되는 숫자 -> int
  - 열여덟 자리로 표현되는 숫자 -> long

<br><br>

# 아이템 61: 박싱된 기본 타입보다는 기본 타입을 사용하라

- 자바의 데이터 타입 : 기본 타입 / 참조 타입(String, List 등) 
  - 박싱된 기본 타입: 각각의 기본 타입에 대응하는 참조타입
    - int - Integer / double - Double / boolean - Boolean
- 오토박싱과 오토언박싱 때문에 두 타입을 크게 구분하지 않고 사용할 수 있으나 두 타입 간 차이가 사라지는 것은 아님

### 기본 타입 vs 박싱된 기본 타입
1. 값만 가지고 있음 vs 값+식별성을 가짐 
    - 박싱된 기본 타입의 두 인스턴스는 값이 같아도 서로 다르다고 식별될 수 있음 
      - 박싱된 기본 타입에 == 연산자를 사용하면 잘못된 결과를 반환함
2. 값이 언제나 유효함 vs 유효하지 않은 값(null)을 가질 수 있음
3. 기본 타입이 시간과 메모리 사용면에서 더 효율적
4. 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 해제됨

### 박싱된 기본 타입을 사용하는 경우
1. 컬렉션의 원소, 키, 값으로 사용
    - 컬렉션은 기본 타입을 담을 수 없기 때문
2. 매개변수화 타입, 매개변수화 메서드의 타입 매개변수인 경우 
    - 자바가 타입 매개변수로 기본 타입을 지원하지 않기 때문
    - ex) `ThreadLocal<Integer>`
3. 리플렉션을 통해 메서드를 호출할 때

<br><br>

# 아이템 62: 다른 타입이 적절하다면 문자열 사용을 피하라

- 문자열은 텍스트를 표현하도록 설계되었으나 다른 의도로 사용되기도 함

### 문자열의 특징
1. 문자열은 다른 값 타입을 대신하기에 적합하지 않음
2. 문자열은 열거 타입을 대신하기에 적합하지 않음
3. 문자열은 혼합 타입을 대신하기에 적합하지 않음
    ```java
    String compoundKey = className + "#" + i.next();
    ```
    - 두 요소 중 하나에 문자 #이 포함된다면 오류 발생 
    - String이 제공하는 기능해만 의존해야 함
    - -> private 정적 멤버 클래스로 선언해야 함
4. 문자열은 권한을 표현하기에 적합하지 않음 (Q)
    ```java
    // ex) 문자열로 권한 구분
    public class ThreadLocal {
        private ThreadLocal() { } // 객체 생성 불가

        // 현 스레드의 값을 키로 구분해 저장한다.
        public static void set(String key, Object value);

        // 키가 가리키는 현재 스레드의 값을 반환한다.
        public static Object get(String key);
    }
    ```
    - 문제: 스레드를 구분하는 문자열 키가 전역 이름공간에서 공유됨
    - 각 클라이언트가 고유한 키를 제공해야 함 

    ```java
    // 리팩터링 후 - key 클래스로 권한 구분
    public class ThreadLocal {
        private ThreadLocal() { } // 객체 생성 불가

        public static class Key { // 권한
            Key() { }
        }

        public static Key getKey() {
            return new Key();
        }

        public static void set(Key key, Object value);
        public static Object get(Key key);
    }
    ```
    - 문자열 대신 key 클래스로 권한 구분하여 문제점 해결
    - 개선점: set과 get static 메서드는 key 클래스의 인스턴스 메서드로 변경
      - key는 스레드 지역변수를 구분하는 용도가 아니라 그 자체가 스레드 지역변수가 됨
     ```java
    // ThreadLocal의 역할이 사라졌으므로 key를 ThreadLocal로 변경
    public class ThreadLocal {
        private ThreadLocal();
        public void Set(Object value);
        public Object get();
    }

    // 타입 안전성을 위해 ThreadLocal을 매개변수화 타입으로 선언
    public class ThreadLocal<T> {
        private ThreadLocal();
        public void Set(T value);
        public T get();
    }
    ```     

<br><br>

# 아이템 63: 문자열 연결은 느리니 주의하라
- 문자열 연결 연산자(+)로 문자열 n개를 잇는 시간은 n²에 비례
- 문자열은 불변이기 때문에 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하므로 성능 저하를 피할 수 없음
- StringBuilder의 append 메서드을 사용하면 성능을 개선할 수 있음

<br><br>

# 아이템 64: 객체는 인터페이스를 사용해 참조하라
1. 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언할 것
    - -> 구현 클래스 교체가 용이함 (프로그램의 유연성 제고)
    - but, 구현체가 인터페이스의 일반 규약 이외의 특별한 기능을 제공한다면 주의해야함 
      - ex) LinkedHashSet의 순서 정책을 가정하고 사용했다면 HashSet(순회 순서 보장 x)으로 바꿀 경우 문제가 될 수 있음
2. 적합한 인터페이스가 없다면 당연히 클래스로 참조할 것
    - 적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인(상위의) 클래스를 타입으로 사용
      - ex1) 값 클래스(String, BigInteger 같은): 값 클래스는 여러 가지로 구현되지 않기 때문에 final인 경우가 많고, 상응하는 인터페이스가 존재하는 경우가 거의 없음
      - ex2) 클래스 기반으로 작성된 프레임워크가 제공하는 객체들(OutputStream 같은 java.io 패키지의 여러 클래스들): 특정 구현 클래스보다는 기반 클래스(보통은 추상 클래스)를 참조하는 것이 좋음
      - ex3) 인터페이스에는 없는 특별 메서드를 제공하는 클래스들(PriorityQueue 클래스 - Queue 인터페이스에 없는 comparator 메서드 제공)

<br><br>

# 아이템 65: 리플렉션보다는 인터페이스를 사용하라

<br><br>

# 아이템 66: 네이티브 메서드는 신중히 사용하라
- 자바 네이티브 인터페이스(JNI): 자바 프로그램이 네이티브 메서드를 호출하는 기술
- 네이티브 메서드: C, C++ 같은 네이티브 프로그래밍 언어로 작성한 메서드

### 네이티브 메서드를 사용하는 경우
1. 레지스트리 같은 플랫폼 특화 기능을 사용할 때
    - 자바가 성숙해지면서 하부 플랫폼의 기능들을 흡수하고 있기 때문에 네이티브 메서드를 사용할 필요가 줄어들고 있음 
      - ex) process API(자바9~): OS 프로세스에 접근할 수 있음
    - but, 대체할 만한 자바 라이브러리가 없다면 네이티브 라이브러리를 사용해야 함  
2. 네이티브 코드로 작성된 기존 라이브러리를 사용할 때
    - ex) 레거시 데이터를 사용하는 레거시 라이브러리 
3. 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역을 따로 네이티브 언어로 작성할 때
    - 권장하지 않음
    - cf) 고성능의 다중 정밀 연산이 필요할 경우에는 네이티브 라이브러리인 GMP를 사용하는 것을 고려해볼 것

### 네이티브 메서드의 단점
1. 네이티브 언어가 안전하지 않으므로 네이티브 메서드도 메모리 훼손 오류로부터 안전하지 않음
2. 자바와 비교했을 때 플랫폼 종속성은 높고, 이식성은 낮고, 디버깅도 어려움
3. 속도가 더 낮아질 수도 있으며 가비지 컬렉터가 네이티브 메모리는 자동 회수, 추적하지 못함
4. 네이티브 코드와 자바 코드의 경계를 넘나들 때마다 비용이 발생함
5. 네이티브 메서드와 자바 코드 사이의 접착 코드를 작성하기가 어려움 (귀찮고 가독성이 떨어짐)

<br><br>

# 아이템 67: 최적화는 신중히 하라
추후 정리 

<br><br>

# 아이템 68: 일반적으로 통용되는 명명 규칙을 따르라
추후 정리
