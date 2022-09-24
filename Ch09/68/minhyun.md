# 명명규칙

- 철자
- 문법

# 철자

패키지, 클래스, 인터페이스, 메서드, 필드, 타입변수의 이름을 다룬다.

반드시 따르자. 이 규칙을 어긴 API는 사용하기 어렵고, 유지보수하기 어렵다.

## 규칙

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6fb3b138-44b6-4329-9675-2996597df136/Untitled.png)

### 패키지와 모듈 이름

- 각 요소를 점(.)으로 구분하여 계층적으로 짓는다.
- 요소들은 소문자 알파벳 혹은 (드물게) 숫자로 이뤄진다.
- 조직 바깥에서도 사용될 패키지면 인터넷 도메인 이름을 역순으로 사용한다. (ex) com.google
- 패키지 이름의 나머지는 패키지를 설명하는 8자 이하의 짧은 단어로 한다.

### 클래스와 인터페이스 이름

- 각 단어는 대문자로 시작한다.

### 메서드와 필드 이름

- 첫 글자를 소문자로 쓴다는 점만 빼면 클래스 명명규칙과 같.

### 상수 필드 (static final 필드의 타입이나, 불변 참조타입) 이름

- 상수 필드를 구성하는 단어는 모두 대문자로 쓴다.
- 단어 사이는 밑줄로 구분한다.

### 지역 변수 이름

- 다른 멤버와 비슷한 명명 규칙이 적용된다.

### 타입 매개변수 이름

- T: 임의의 타입
- E: 컬렉션 원소의 타입
- K: 맵의 키
- V: 맵의 값
- X: 예외
- R: 메서드의 반환 타입
- T, U, V, T1, T2, T3: 임의 타입의 시퀀스

# 문법

## 규칙

### 객체를 생성할 수 있는 클래스의 이름

- 단수 명사나 명사구 사용
    - (ex) Thread, PriorityQueue …

### 객체를 생성할 수 없는 클래스

- 보통 복수형 명사 사용
    - (ex) Collectors, Collections …

### 인터페이스 이름

- 클래스와 똑같이 짓거나 ‘able’ 혹은 ‘ible’로 끝나는 형용사로 짓는다.
    - (ex) Runnable, Iterable, Accessible …

### 애너테이션

- 딱히 규칙은 없다.

### 메서드

- 동사나 (목적어를 포함한) 동사구로 짓는다.
    - (ex) append, drawImage …
- boolean 값을 반환하는 메서드라면 보통 is나 has로 시작하고 명사나 명사, 형용사로 기능하는 단어나 구로 끝나도록 짓는다.
    - (ex) isDigit, isProbablePrime, hasSiblings …
- 인스턴스의 속성을 반환하는 메서드면 명사 혹은 get으로 시작하는 동사구로 짓는다.
    - (ex) size, getTime …

### 객체의 타입을 바꾸거나, 다른 타입의 또다른 객체를 반환하는 인스턴스 메서드 이름

- toType 형태로 짓는다.
    - (ex) toString, toArray …

### 객체의 내용을 다른 뷰로 보여주는 메서드

- asType 형태로 짓는다.
    - (ex) asList …

### 객체의 값을 기본 타입값으로 반환하는 메서드

- typeValue 형태로 짓는다.
    - (ex) intValue …

### 정적 팩터리

- from
- of
- valueOf
- instance
- getInstance
- newInstance
- getType
- newType