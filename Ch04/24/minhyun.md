# 중첩클래스

**다른 클래스 안에 정의된 클래스**이다. 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

## 종류

- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

첫번째를 제외한 나머지는 내부 클래스에 해당한다.

## 1. 정적 멤버 클래스

바깥 클래스와 함꼐 쓰일때만 유용한 public 도우미 클래스로 쓰인다.

### 특징

- 다른 클래스 안에 선언된다
- 바깥 클래스의 private 멤버에도 접근할 수 있다.
- 그 외엔 일반 클래스와 동일

*(ex) 계산기가 지원하는 연산 종류를 정의하는 열거 타입*

## 2. 비정적 멤버 클래스

정적 멤버 클래스와 구문상 차이는 `static` 의 유무지만 의미상 차이는 꽤 크다.

### 특징

- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결
    - 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 `this` 를 사용해 바깥 인스턴스의 메서드를 호출하거나, 참조를 가져올 수 있다.
        - 정규화된 `this` → `클래스명.this` 형태
- 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없음.
    - 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들 것
- 비정적 멤버 클래스와 바깥 인스턴스는 멤버 클래스가 인스턴스화 될 때 확립되며, 더 이상 변경할 수 없다.
    - 드물게 `바깥 인스턴스의 클래스.new Member Class(args)` 를 호출해 수동으로 만들기도 하나, 성능면에서 별로이다.
- 어댑터를 정의할 때 자주 쓰인다.
    - 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것
    - *(ex) Map 인터페이스의 구현체 → 자신의 반복자 구현*

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여 정적 멤버 클래스로 만들자.**

→ 이 참조를 저장하려면 시간과 공간이 소비되며, 가비지 컬렉션이 바깥 클래스의 인스턴스르 ㄹ수거하지 못해 메모리 누수가 생길 수 있기 때문.

### private 정적 멤버 클래스

바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 사용.

**예시**

모든 Entry가 Map과 연관되어 있지만 Entry의 메서드들(ㅎetKey, getValue, setValue)은 맵을 직접 사용하지 않는다.

그래서 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비라 private 정적 멤버 클래스가 가장 알맞다.

## 3. 익명 클래스

### 특징

- 멤버와 달리 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 코드의 어디서든 만들 수 있다.
- 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.
- 응용하는데 제약이 많다.
    - 선언한 지점에서만 인스턴스 생성 가능
    - 여러 인터페이스 구현 X
    - 인터페이스 구현하는 동시에 다른 클래스 상속 X
    - 사용하는 클라이언트는 익명 클래스가 상위 타입에서 상속한 멤버 외에 호출할 수 없음.
    - 짧지 않으면 가독성이 떨어짐.
- 람다를 지원하기 전 자주 사용했다.
- 정적 팩터리 메서드를 구현할 때 쓰인다.

## 4. 지역 클래스

4가지 중첩 클래스 중 가장 드물게 사용된다.

### 특징

- 지역 변수를 선언할 수 있는 곳이면 어디든 선언할 수 있다.
- 유효 범위도 지역변수와 같다.
- 멤버 클래스처럼 이름이 있고 반복하여 사용가능
- 익명클래스처럼 비정적 문맥에서 사용될때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버는 가질 수 없으며, 가독성을 위해 짧게 작성해야 한다.

# 정리

- 중첩클래스에는 네가지가 있으며 각각 쓰임이 다르다.
    - 메서드 밖에서도 사용해아하거나 메서드 안에 정의하기에 너무 길다 → 멤버 클래스
        - 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면?
            - 네 → 비정적
            - 아니요 → 정적
    - 중첩 클래스가 한 메서드 안에서만 쓰이고, 인스턴스 생성하는 지점이 단 한곳이며 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 존재한다면?
        - 네 → 익명 클래스
        - 아니오 → 지역 클래스