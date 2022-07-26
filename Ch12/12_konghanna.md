# 아이템 85: 자바 직렬화의 대안을 찾으라 

### 직렬화의 위험성
1. 공격 범위가 너무 넓고, 지속적으로 더 넓어져 방어하기 어려움
    - OutputInputStream의 readObject 메서드
      - 객체 그래프가 역직렬화될 때 호출됨, 거의 모든 타입의 객체를 만들어낼 수 있는 생성자 
      - 바이트 스트림을 역직렬화하는 과정에서 그 타입들 안의 모든 코드를 수행할 수 있음 -> 그 타입들의 코드 전체가 공격 범위에 포함됨  
2. 역직렬화 과정에서 잠재적으로 위험한 동작을 수행하는 메서드(가젯)가 호출될 수 있음 
    - 신중하게 제작한 바이트 스트림만 역직렬화해야 함
3. 역직렬화에 시간이 오래 걸리는 짧은 스트림(역직렬화 폭탄)을 역직렬화하면 서비스 거부 공격에 쉽게 노출될 수 있음 (Q)

### Cross-Platform Structured-Data Representation
- 직렬화 위험을 피하는 방법은 아무것도 역직렬화하지 않는 것 -> 크로스-플랫폼 구조화된 데이터 표현을 사용할 것
  - ex) JSON, 프로토콜 버퍼

### 직렬화를 꼭 사용해야 하는 경우
1. 신뢰할 수 없는 데이터는 절대 역직렬화하지 말 것
    - 특히 신뢰할 수 없는 발신원으로부터의 RMI는 수용하지 말 것  
2. 역직렬화한 데이터가 안전한지 확신할 수 없다면 객체 역직렬화 필터링(ObjectInputFilter)을 사용할 것
    - 객체 역직렬화 필터링: 데이터 스트림이 역직렬화되기 전에 필터를 설치하는 기능, 특정 클래스를 수용 or 거부할 수 있음
      - 기본적으로 모든 클래스를 거부하고 화이트리스트에 기록된 클래스만 수용하는 화이트리스트 방식 권장 cf) 스왓 - 화이트리스트를 자동으로 생성해주는 도구  

<br><br>

# 아이템 86: Serializable을 구현할지는 신중히 결정하라 

### Serializable 구현 시 문제점
1. 릴리스한 뒤 수정하기 어려움 
    - Serializable을 구현하면 직렬화 형태도 하나의 공개 API가 됨 -> 클래스의 private, package-private 인스턴스 필드들마저 API로 공개되어 캡슐화가 깨짐
    - 뒤늦게 클래스를 수정할 시 구버전 인스턴스로 직렬화, 신버전 클래스로 역직렬화하게 됨 -> 오류 발생 
    - serialVersionUID 
      - 직렬화된 클래스가 부여받는 고유 식별 번호, 명시되지 않으면 런타임에 자동으로 생성됨 
      - 클래스 멤버들에 따라 부여받는 번호로 클래스가 수정되면 값이 변함 -> 자동 생성되는 값에 의존할 경우 호환성이 깨져서 런타임 오류 발생 

2. 버그와 보안 구멍이 생길 위험이 높아짐
    - 직렬화: 생성자를 사용하는 기본 방식을 우회하는 객체 생성 기법
    - 역직렬화: 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자' -> 역직렬화 사용 시 불변식이 깨질 수 있고 허가되지 않은 접근에 쉽게 노출될 수 있음

3. 신버전을 릴리스할 때 테스트할 요소가 늘어남
    - 수정된 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지, 그 반대도 가능한지 테스트해야 함

### Serializable 구현 시 주의사항
1. 구현 여부는 가볍게 결정할 사안이 아님
    - Serializable 구현에 따르는 비용이 적지 않기 때문에 이득과 비용을 잘 따져봐야 함
      - 값 클래스(ex. BigInter, Instant), 컬렉션 클래스: 구현 O
      - 동작하는 객체를 표현한 클래스(ex. 스레드 풀): 구현 X

2. 상속용으로 설계된 클래스와 인터페이스는 대부분 구현하면 안됨
    - 클래스를 확장, 인터페이스를 구현하는 대상에게 부담을 지우게 됨
    - Serializable을 구현한 클래스만 지원하는 프레임워크를 사용하는 경우 이 원칙을 어길 수밖에 없음
      - ex) Throwable 클래스: 상속용으로 설계됨, but 서버가 RMI를 통해 클라이언트로 예외를 보내기 위해 Serializable을 구현
    - 직렬화와 확장이 모두 가능한 클래스를 작성한다면, 하위 클래스에서 finalize 메서드를 재정의하지 못하게 하여 불변식을 보장할 수 있음
      - 자신이 finalize 메서드를 재정의하고 final로 선언 
    - 인스턴스 필드 중 기본값으로 초기화되면 위배되는 불변식이 있다면 클래스에 readObjectNoData 메서드 추가 (스트림 데이터가 아니면 예외 발생) 
    
3. 내부 클래스는 직렬화를 구현하면 안됨 
    - 내부 클래스에 대한 기본 직렬화 형태가 분명하지 않음
      - 내부 클래스에는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가됨 -> 필드들이 어떻게 추가되는지 알 수가 없음
    - 정적 멤버 클래스는 Serializable을 구현해도 됨

<br><br>

# 아이템 87: 커스텀 직렬화 형태를 고려해보라 

- 클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면 현재의 구현에 종속될 수밖에 없음, 기본 직렬화 형태를 버릴 수 없게 됨 -> 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 기본 직렬화 형태를 사용할 것 

### 이상적인 직렬화 형태
- 기본 직렬화 형태는 객체가 포함된 데이터, 그 객체로부터 시작해 접근할 수 있는 모든 객체, 객체들이 연결된 위상까지 기술
- 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 함 
  - cf) 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태를 사용해도 무방 
- 기본 직렬화 형태가 적합하더라도 불변식 보장, 보안을 위해 readObject 메서드를 제공해야 하는 경우가 많음

### 적합하지 않은 기본 직렬화 형태를 사용할 시 문제점
```java
// 논리적: 문자열을 표현
// 물리적: 문자열들을 이중 연결 리스트로 연결
// -> 기본 직렬화 형태 사용 시 각 노드의 연결 정보까지 표현할 것 
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    // ... 
}
```
- 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 생기는 문제 
1. 공개 API가 현재의 내부 표현 방식에 영구히 묶이게 됨
    - 사용하지 않는 코드도 공개 API가 되어버리기 때문에 제거할 수 없게 됨
2. 너무 많은 공간을 차지할 수 있음
    - 연결 정보 등 내부 구현 정보가 포함되어 직렬화 형태가 너무 커짐 -> 디스크에 저장하거나 네트워크로 전송하는 속도가 느려짐
3. 시간이 너무 많이 걸릴 수 있음
    - 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 직접 순회해볼 수밖에 없음
4. 스택 오버플로를 발생시킴
    - 기본 직렬화 과정은 객체 그래프를 재귀 순회함 -> 스택 오버플로를 일으킬 수 있음

### 합리적인 직렬화 형태
- 단순히 리스트가 포함한 문자열의 개수, 문자열만 있으면 됨
  - 물리적인 상세 표현은 배제하고 논리적인 구성만 담는 것 
    - writeObject, readObject: 직렬화 형태 처리 
    - transient 한정자: 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는다는 표시 
    - 필드 모두가 transient로 선언되었더라도 writeObject, readObject 메서드가 각각 defaultWriteObject, defaultReadObject를 호출함 -> 향후 릴리스에서 transient가 아닌 인스턴스 필드가 추가되더라도 상호 호환됨

### 주의사항 
- 기본 직렬화 수용 여부와 상관없이 defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화됨
  - -> transient를 선언해도 되는 필드는 transient 한정자를 붙여야 함
  - 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략할 것 -> 커스텀 직렬화 형태를 사용한다면 대부분의 인스턴스 필드를 transient로 선언해야 함
- 기본 직렬화를 사용한다면 transient 필드들은 역직렬화될 때 기본값으로 초기화됨
  - 기본값을 사용하면 안되는 경우 readObject 메서드에서 defaultReadObject 메서드를 호출한 다음 원하는 값으로 복원할 것, 혹은 그 값을 처음 사용할 때 초기화할 것 

### 동기화 
- 기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 함
  - 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에 기본 직렬화를 사용하려면 writeObject도 synchronized로 선언해야 함

### SerialVersionUID
- 어떤 직렬화 형태를 택하든 직렬화 가능 클래스에 모두 직렬 버전 UID를 명시적으로 부여해야 함
  - 잠재적인 호환성 문제가 사라지고, 이 값을 자동 생성하기 위해 런타임에 복잡한 연산을 수행하지 않아도 됨
  - 꼭 고유할 필요는 없음
  - 구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 절대 수정하지 말 것

<br><br>

# 아이템 88: readObject 메서드는 방어적으로 작성하라 

- readObject 메서드는 매개변수로 바이트 스트림을 받는 생성자라고 할 수 있음 -> 보통의 생성자처럼 인수의 유효성을 검사하고, 필요하다면 매개변수를 방어적으로 복사해야 함
  - 보통 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해서 만들어지지만, 불변식을 깨뜨릴 의도로 생성된 바이트 스트림을 받으면 문제가 생김 -> 정상적인 생성자로 만들어낼 수 없는 객체를 생성할 수 있음

### readObject 작성 시 주의사항
- 객체를 역직렬화할 때는 클라이언트가 소유하면 안되는 객체 참조를 갖는 필드를 반드시 방어적으로 복사해야 함 
    - readObject 메서드가 defaultReadObject를 호출한 후  역직렬화된 객체의 유효성을 검사해야 함 -> 잘못된 역직렬화를 막기에 충분하지 X
      - 바이트 스트림 끝에 참조를 추가하면 가변적인 인스턴스를 만들어 내부 정보를 수정할 수 있기 때문 -> readObject 메서드가 방어적 복사를 해야 함
        - readObject에서는 불변 클래스 안의 모든 private 가변 요소들을 방어적으로 복사해야 함
        - 방어적 복사는 유효성 검사보다 먼저 수행해야 함
        - final 필드는 방어적 복사가 불가능하므로 final 한정자는 제거해야 함 
2. transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮으면 -> 기본 readObejct 메서드 사용
    - 아닌 경우 커스텀 readObejct 메서드를 만들어 모든 유효성 검사와 방어적 복사 수행
    - 또는, 직렬화 프록시 패턴을 사용할 것 -> 역직렬화를 안전하게 만드는 데 필요한 노력을 줄여줌
3. final이 아닌 직렬화 가능 클래스의 경우 생성자처럼 readObject 메서드도 재정의 가능한 메서드를 호출해서는 안됨
    - 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행되기 때문

<br><br>

# 아이템 89: 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라 
- 싱글턴으로 만들어진 클래스도 Serializable을 구현하게 되는 순간 싱글턴이 아니게 됨
  - 기본 직렬화를 쓰지 않고, 명시적인 readObject 메서드를 제공하더라도 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 됨

### readResolve 메서드
- readObject가 만들어낸 인스턴스를 대체하는 객체 참조를 반환함 -> 역직렬화된 객체가 아닌 기존의 인스턴스를 반환하여 싱글턴 속성을 유지할 수 있음
  - readResolve 메서드를 인스턴스 통제 목적으로 사용 시 모든 인스턴스 필드는 transient로 선언해야 함 -> 그렇지 않으면 역직렬화된 인스턴스를 공격할 수 있음
    - 한계: 싱글턴이 transient가 아닌 필드를 가지고 있으면 그 필드의 내용은 readResolve가 실행되기 전에 역직렬화됨 -> 스트림으로 역직렬화된 인스턴스의 참조를 훔쳐올 수 있음

### 대안: enum 방식 싱글턴 구현
- 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체가 존재하지 않음이 보장됨
  - 공격자가 AccessibleObject.setAccessible처럼 네이티브 코드를 수행하는 메서드를 악용하는 경우 제외 
- 컴파일타임에 어떤 인스턴스들이 있는지 알 수 없는 경우에는 열거 타입으로 표현하는 것이 불가능하기 때문에 readResolve 메서드를 사용해야 함

<br><br>

# 아이템 90: 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
- Serializable을 구현하면 정상 메커니즘인 생성자 의외의 방법으로 인스턴스를 생성할 수 있게 됨 -> 버그와 보안 문제가 일어날 위험이 커짐 -> 직렬화 프록시 패턴은 이 위험을 크게 줄여주는 기법임

### 직렬화 프록시 패턴
- 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언 
  - 중첩 클래스 = 바깥 클랙스의 직렬화 프록시
  - 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 함 -> 생성자는 인수로 넘어온 인스턴스의 데이터를 복사함 (일관성 검사, 방어적 복사 필요 X)
  - 바깥 클래스, 직렬화 프록시 모두 Serializable을 구현해야 함 
  - readResolve 메서드로 역직렬화 시 바깥 클래스의 인스턴스로 변환
    - 일반 인스턴스를 만들 때와 똑같은 방식으로 역직렬화된 인스턴스를 생성함 
    - 불변식 확인도 클래스의 정적 팩터리, 생성자로 확인

### 직렬화 프록시 패턴의 장점
1. 가짜 바이트 스트림 공격, 내부 필드 탈취 공격을 프록시 수준에서 차단
2. 필드를 final로 선언해도 되기 때문에 진정한 불변으로 만들 수 있음
3. 역직렬화 시 유효성 검사를 따로 수행하지 않아도 됨
4. 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동함 
    - ex) EnumSet - 원소 개수가 64 이하면 RegularEnumSet, 그 이상이면 JumboEnumSet 사용
      - 원소의 개수가 달라진져도 직렬화된 인스턴스 타입이 아닌 다른 타입의 인스턴스를 역직렬화할 수 있음 

### 직렬화 프록시 패턴의 한계
1. 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없음
2. 객체 그래프에 순환이 있는 클래스에는 적용할 수 없음
    - 이런 객체의 메서드를 직렬화 프록시의 readResolve에서 호출하는 경우 예외 발생. 직렬화 프록시만 가진 것이지 실제 객체는 아직 만들어지지 않았기 때문
3. 방어적 복사보다 속도가 느림
