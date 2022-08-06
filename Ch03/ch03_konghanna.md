# 아이템 10: equals는 일반 규약을 지켜 재정의하라

- equals 메소드는 아예 재정의하지 않는 것이 최선

### equals 메소드를 재정의하면 안되는 경우
- 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스인 경우 
    - Thread 클래스: Object의 equals 메소드 사용 
- 논리적 동치성을 검사할 필요가 없는 경우 
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞는 경우 
- private이거나 package-private이라 equals 메소드를 호출할 일이 없는 클래스의 경우 
    - equals를 오버라이딩해서 호출되지 않도록 막아야 함 (예외를 던지게)

### equals 메소드를 재정의해야 하는 경우
- 객체 식별성(두 객체가 물리적으로 같은지)이 아니라 논리적 동치성을 확인하기에 상위 클래스의 equals가 적합하지 않을 때 (주로 값 클래스일 경우)   
    - 논리적 동치성을 확인하는 경우 값을 비교할 수 있을 뿐만 아니라 인스턴스를 Map의 키와 Set의 원소로 사용할 수 있게 됨 
     - 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스의 경우 equals를 재정의하지 않아도 됨 (ex. Enum 타입 클래스)  

 ### equals 메소드의 일반 규약
- equals는 동치관계(집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산)를 구현하며 다음의 조건을 만족함
   - 반사성: null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true 
   - 대칭성: null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true 
   - 추이성: null이 아닌 모든 참조 값 x,y,z에 대해 x.equals(y), y.equals(z)가 true면 x.equals(z)도 true
   - 일관성: null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true거나 항상 false  
   - null-아님: null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false

 ### equals 메소드 구현 방법
- == 연산자를 사용해 입력이 자기 자신의 참조인지 확인 
- instanceof 연산자로 입력이 올바른 타입인지 확인
- 입력을 올바른 타입으로 형변환함. 위에서 instanceof 검사를 했기 때문에 100% 성공
- 입력된 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사 
    - float, double을 제외한 기본 타입 필드는 == 연산자로 비교 
    - float, double 필드는 정적 메소드인 Float.compare, Double.compare로 비교
    - 참조 타입 필드는 equlas 메소드로 비교

<br><br>

# 아이템 11: equals를 재정의하려거든 hashCode도 재정의하라

- equals 메소드를 재정의한 클래스 모두에서 hashCode도 재정의해야 함 <br>
-> hashCode를 재정의하지 않으면 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으키게 됨

### hashCode 메소드의 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 함 (단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
- equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 함 
- equals가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없음. (단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아짐)
<br>

- hashCode 재정의를 잘못해서 두 번째 규약을 어기면 문제가 발생함 -> 논리적으로 같은 객체는 같은 해시코드를 반환해야 함

### 좋은 hashCode 메소드를 작성하는 방법
- 좋은 해시 함수: 서로 다른 인스턴스에 다른 해시코드를 반환해야 함 (세 번째 규약이 요구하는 속성)
    - 이상적인 해시 함수: 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 함
1. int 변수 result를 선언한 후 값 c(2.a 방식으로 계산한 해시코드)로 초기화 함
2. 해당 객체의 나머지 핵심 필드 각각에 대해 다음 작업을 수행함 <br>
   a. 해당 필드의 해시코드 c를 계산 <br>
      &nbsp; 1) 기본 타입 필드: Type.hashCode 사용 <br>
      &nbsp; 2) 참조 타입 필드: <br>
            &nbsp;&nbsp; - 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashcode를 재귀적으로 호출함. <br>
      &nbsp; 3) 배열 타입 필드: <br>
           &nbsp;&nbsp;  - 핵심 원소 각각을 별도 필드처럼 다루고 이상의 규칙을 재귀적으로 적용하여 각 핵심 원소의 해시코드를 계산한 후 2.b 방식으로 갱신 <br>
           &nbsp;&nbsp;  - 모든 원소가 핵심 원소라면 Arrays.hashcode 사용 <br>
    b. 계산한 해시코드 c로 result 갱신 <br>
      &nbsp;&nbsp;  result = 31 * result + c;  <br>
3. result 반환 
- equals 비교에 사용되지 않는 필드는 반드시 제외해야 함

### Objects 클래스의 hash 
- 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메소드

### 해시코드를 지연 초기화하는 방법
- 클래스가 불변이고 해시코드를 계산하는 비용이 너무 크다면 매번 새로 계산하는 것보다는 캐싱하는 방식을 고려해야 함
- 해시의 키로 잘 사용되지 않는 객체의 경우 hashCode가 처음 불릴 때 계산하는 지연 초기화 전략 사용 <br>
    - 필드를 지연 초기화하려면 스레드 안전성을 고려해야 함 
    - hashCode 필드의 초기값은 흔히 생성되는 객체의 해시코드와는 달라야 함
    
<br><br>

# 아이템 12: toString을 항상 재정의하라

- Object의 기본 toString 메소드: 클래스 이름@16진수로 표시한 해시코드를 반환함

### toString의 규약
  - 간결하고 읽기 쉬운 형태의 유익한 정보를 반환해야 함 <br> 
     -> 모든 하위 클래스에서 메소드를 재정의해야 함

 ### toString 재정의 방법
   1. 객체가 가진 주요 정보 모두를 반환하는 게 좋음
   2. 반환값의 포맷을 명시하든 아니든 의도를 명확하게 해야 함
   3. 반환값에 포함된 정보를 얻어올 수 있는 API를 제공해야 함 <br>
       -> 없으면 이 정보를 얻기 위해 toString의 반환값을 파싱할 수 밖에 없음        

### toString를 재정의하지 않아도 되는 경우
   1. 정적 유틸리티 클래스
   2. 대부분의 enum 타입
   3. 상위 클래스에서 이미 알맞게 재정의한 경우
      cf. 하위 클래스에서 공유해야 할 문자열 표현이 있는 추상 클래스: toString을 재정의해줘야 함 ex) 추상 컬렉션 클래스

<br><br>

# 아이템 13: clone 재정의는 주의해서 진행하라

### Cloneable 인터페이스
- Cloneable: 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스지만 의도한 목적을 이루지 못함 <br>
-> clone 메소드가 선언된 곳이 Cloneable이 아닌 Object 클래스고 접근 지정자도 protected이기 때문 <br>
-> Cloneable을 구현하는 것만으로 clone 메소드를 호출할 수 없음
- 메소드도 하나 없지만 Object의 protected 메소드인 clone의 동작 방식을 결정함
  - Cloneable을 구현한 클래스의 인스턴스에서 clone 메소드를 호출하면 그 객체의 필드들을 모두 복사한 객체를 반환하고, 그렇지 않은 클래스의 인스턴스에서 호출하면 예외를 던짐
   -> 인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위지만, Cloneable의 경우에는 상위 클래스에 정의된 protected 메소드의 동작 방식을 변경한 것

### clone 메소드의 일반 규약 
1. x.clone() != x
2. x.clone().getClass() = x.getClass()  
3. x.clone().equlas(x)                              
- 2, 3은 일반적으로 참이지만 필수는 X
- 생성자 연쇄와 비슷한 메커니즘
    - 하위 클래스에서 clone 메소드가 제대로 동작하지 않을 수 있음 <br>
    ex) 클래스 B가 클래스 A를 상속할 때
        -  클래스 A의 clone이 자신의 생성자로 생성한 객체를 반환한다면 B의 clone도 A타입 객체를 반환하게 됨 <br>
        -> super.clone을 연쇄적으로 호출하도록 구현하면 하위 클래스의 객체가 만들어짐

Q. 79p~85p

### clone 방식보다 더 나은 객체 복사 방식: 복사 생성자와 복사 팩터리
- 복사 생성자: 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자
- 복사 팩터리: 복사 생성자를 모방한 정적 팩터리 
- 배열을 제외하고는 clone 메소드 방식이 아닌 복사 생성자, 복사 팩터리 방식을 사용하는 것이 권장됨
```java
    // 복사 생성자
    public Yum(yum yum) {    }

    // 복사 팩터리
    public static Yum newInstance(yum yum) {    }
```

<br><br>

# 아이템 14: Comparable을 구현할지 고려하라

### Comparable 인터페이스의 compareTo 메소드 vs equals
- compareTo는 단순 동치성 비교뿐만 아니라 순서까지 비교할 수 있으며 제네릭함
- Comparable을 구현했다는 것은 그 클래스의 인스턴스들에 자연적인 순서가 있음을 의미함 <br>
-> 순서가 명확한 값 클래스 작성 시 Comparable 인터페이스를 구현해야 함 
- compareTo를 활용하는 클래스 <br>
  ex) 정렬된 컬렉션: TreeSet, TreeMap <br>
         검색과 정렬 알고리즘 활용하는 유틸리티 클래스: Collections, Arrays

 ### compareTo 메소드의 일반 규약
- 이 객체와 주어진 객체의 순서 비교
   - 이 객체가 주어진 객체보다 작으면 음의 정수를, 이 객체와 주어진 객체가 같으면 0을, 이 객체가 주어진 객체보다 크면 양의 정수를 반환함
   - 이 객체와 비교할 수 없는 타입의 객체가 주어지면 예외를 던짐 
- sgn(표현식) 표기
   - 대칭성: Comparable을 구현한 클래스는 모든 x, y에 대해 sng(x.compareTo(y)) == -sgn(y.compareTo(x)) 
   - 추이성: Comparable을 구현한 클래스는 x.compareTo(y) > 0 && y.compareTo(z) >0 이면, x.compareTo(z) >0 
   - 반사성: Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0 이면, sgn(x.compareTo(z)) == sgn.(y.compareTo(z)) 
   - eqauls를 수행한 결과와의 일관성: (필수는X) (x.compareTo(y) == 0) == (x.equals(y)) 여야 함
       - Comparable을 구현하고 이 권고를 지키지 않은 클래스에는 그 사실을 명시해야 함
       - cf) BigDecimal 클래스: 충족 X 

Q. 89p 세번째 문단??

### compareTo 메소드 작성 요령
- Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 입력 인수의 타입을 확인하거나 형변환할 필요가 없음 <br>
  -> 인수의 타입이 컴파일 타임에 정해짐, 인수 타입이 잘못됐다면 컴파일 시점에 오류가 발생
- 각 필드가 동치인지 비교하는 것이 아니라 그 순서를 비교함
- 객체 참조 필드를 비교하려면 compareTo 메소드를 재귀적으로 호출
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(comparator)를 대신 사용
    - 직접 만들거나 자바가 제공하는 것 중에 골라 쓰면 됨
- 기본 정수 타입 필드를 비교할 때 <, >를 사용하지 않고 compare를 사용 
- 가장 핵심적인 필드부터 비교해야 함

### comparator 인터페이스 (Q) 
- 자바 8부터 Comparator 인터페이스가 비교자 생성 메소드를 통해 메소드 연쇄 방식으로 비교자를 생성할 수 있게 되었음 

```java
    private static final Compartor<PhoneNumber> COMPARATOR =  
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum)
  
    public int compareTo(PhoneNumber pn){
        return COMPARATOR.compare(this, pn);
    } 
```
