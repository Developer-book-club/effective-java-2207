# 3장 모든 객체의 공통 메서드
Object에서 final이 아닌 equals, hashCode, toString, clone, finalize는 모두 overriding을 염두해두고 설계된 것이라 재정의시 지켜야하는 규약이 명확히 정의되어 잘못 구현하면 규약 준수를 가정하는 HashMap, HashSet 등의 클래스를 오동작 하게 만들 수 있다.(Comparable.compareTo도 유사)
## 아이템 10. equals는 일반 규약을 지켜 재정의하라
1. 재정의하지 않는 것이 최선인 경우
    * 각 인스턴스가 본질적으로 고유 : 동작하는 개체의 경우(ex. Thread)
    * 인스턴스의 논리적 동치성(logical equality) 검사 불필요 : 원하지 않거나 불필요하거나
    * 상위 클래스에서 적절한 equals가 정의되어있음
    * 클래스가 private, package-private이고 equals 메서드를 호출할 일이 없음
    * 값이 같은 인스턴스가 둘 이상 만들어지지 않음이 보장되어있는 인스턴스 통제 클래스(ex. Enum) : 논리적 동치성과 객체 식별성이 동일한 의미
2. 재정의가 필요한 경우 : 물리적(객체 식별성 검사)인 비교가 아니라 논리적인 동치성을 확인해야하는데 상위 클래스의 equals가 그렇지 못할 경우
3. 아래의 재정의 규약을 잘 지키면 객체의 논리적 동치를 확인하기 원하는 프로그래머의 기대에 부흥하며 Map의 키나 Set의 원소로 사용 가능
    * 반사성(reflexivity) :x.equals(x)는 true
    * 대칭성(symmetry) : x.equals(y), y.equals(x) 결과가 동일해야함
    * 추이성(transitivity) : x.equals(y), y.equals(z) 둘다 true면 x.equals(z)도 true
    * 일관성(consistency) : x.equals(y)의 결과는 반복해서 호출해도 동일
    * null-아님 : null이 아닌 모든 객체 x의 x.equals(null) == false
4. 양질의 equals 메서드 구현 방법
    1. == 연산자로 입력이 자기참조인지 확인(성능 최적화용)
    2. instanceof 연산자로 타입 비교: 아니면 바로 false 반환
    3. 형변환(타입 비교를 했기때문에 100% 성공)
    4. 핵심 필드 하나씩 검사(하나라도 다르면 false 반환)
        * float, douvle 제외한 기본 타입은 == 연산자로 비교
        * 참조 필드는 equals로 비교
        * float, double 필드는 NaN, -0.0f, 특수한 부동소수값 때문에 compare 메소드로 비교(equals보다 성능 우위)
        * 다를 가능성이 더 크거나, 비교하는 비용이 싼 필드 먼저 비교(성능)
    5. 대칭성, 추이성, 일관성 단위 테스트 작성(혹은 AutoValue 사용)
    ```java
    // 예시
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
    ```
5. 주의사항
    * equals를 재정의할땐 hashCode도 반드시 재정의
    * 복잡하게 해결하려 하지말것 : 필드들의 동치성만 검사해도 equals의 규약을 어렵지 않게 지킬 수 있음, 별칭 비교하지 말 것
    * equals는 반드시 Object 타임을 매개변수로 받아야함 : 아니면 overriding이 아님
    * 구글 AutoValue 프레임워크 사용하면 테스트가 수월
## 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라 
* equals에서 사용되는 주요 정보가 변하지 않았다면, 애플리케이션 실행 시간 동안에는 hashCode는 항상 동일한 값을 반환해야 함
* equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 동일해야함
* equals가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 꼭 다를 필요는 없지만, 달라야 해시테이블의 성능이 좋아진다.
1. 모든 객체에 동일한 해시코드 반환 금지(해시테이블 성능 저하)
2. 좋은 hashCode를 작성하는 요령
    1. int 변수 result 선언 후 값 c로 초기화(c는 첫번째 핵심 필드로 계산)
    2. 핵심 필드 f 각각에 대해 다음 작업 수행
        a. f의 hashCode 계산
            * 기본 타입 : Type.hashCode(f)
            * 참조 타입이고 equals가 f의 equals를 재귀적으로 호출하면 hashCode도 재귀적으로 호출, 필드값이 null이면 hashcode = 0
            * 배열 : 핵심 원소 각각을 별도 필드처럼 계산        
        b. 2.a에서 계산한 해시코드 c로 result 갱신(result = 31 * result + c)
    3. result 반환
    ```java
    @Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
    ```
    * 31은 홀수이면서 소스이기 때문(짝수고 오버플로우가 발생하면 정보를 잃게되고 소수는 전통적으로 그리해왔다?)
    * 해시 충돌이 적은 방법을 꼭 써야하면 구아바의 com.google.common.hash.Hashing 참고
    * Object.Hash(vargs...)도 제공되지만 속도가 느리다.
    * 클래스가 불변이고 해시코드 계산 비용이 크면 캐싱 방식 고려 필요 : 처음 불릴때 계산하는 지연 초기화(lazy initialization) 전략
    ```java
    private int hashCode;    // 0으로 초기화
    
    @Override public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            reslut = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
    ```
    * 핵심 필드를 생략하면 해시테이블 성능이 떨어진다
    * 클라이언트가 이 값에 의지하지 않도록 생성 규칙 공표하지 말 것
## 아이템 12. toString을 항상 재정의하라
1. Object.toString()은 유의미한 문자열이 아니기때문에 모든 하위 클래스에서 이 메서드를 재정의하는것이 좋다.
2. println, printf, + 연산자, assert 구분 등에서 자동으로 호출된다.
3. 객체가 가진 주요 정보를 모두 포함하여 반환(이상적으로 객체 스스로를 완벽하게 설명하는 문자열)
4. 반환값 포맷을 문서화하면 표준적이고 명확하고 사람이 읽기 쉽지만, 포맷이 바뀔 경우 포맷을 파싱하여 사용하는 경우 문제가 될 수 있다. : 그렇게 사용하지 않도록 하기위해 반환값들을 모두 얻을 수 있는 api 제공 필요
5. 대부분의 경우 구글 AutoValue 프레임워크에서 자동 생성된걸 사용해도 무방(의미를 명확하기 위해서는 직접 구현)
## 아이템 13. clone 재정의는 주의해서 진행하라
## 아이템 14. Comparable을 구현할지 고려하라