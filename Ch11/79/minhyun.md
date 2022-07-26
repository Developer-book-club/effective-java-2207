# 정확성 관점

**과도한 동기화의 안좋은 효과**

- 성능이 떨어진다
- 교착상태에 빠진다
- 예측할 수 없는 동작을 낳을 수 있다

**응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에선 제어를 절대 클라이언트에 양도하면 안된다.**

- 동기화된 영역 안에서 재정의 할 수 있는 메서드 호출 금지
- 클라이언트가 넘겨준 함수객체 호출 금지

왜냐면 외부에서 온 메서드가 어떤 일을 할 지 모르고, 통제할 수도 없기 때문이다. (외계인 메서드)

재진입 가능한 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 교착상태가 될 생황을 안전 실패(데이터 훼손) 로 변모시킬 수 있다.

이 경우 두가지 방법이 있다.

1. 외계인 메서드 호출을 동기화 블록 바깥으로 옮기는 것
2. 자바의 동시성 컬렉션 라이브러리를 사용하는 것

메서드 바깥에서 호출되는 외계인 메서드는 열린 호출(open call)이라고 하는데, 실패 방지 효과 외에도 동시성 효율을 크게 개선해 준다.

→ 동기화 영역 내부에서 호출되면 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야한다.

**일단 동기화의 기본 규칙은 ‘동기화 영역에서는 가능한 한 일을 적게 하라’ 라는 것이다.**

# 성능 관점

요새는 동기화의 비용은 락을 얻는데 드는 CPU 시간이 아니다. 바로 **경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연 시간이 진짜 비용이다.**

## 동기화되는 가변 클래스를 작성할 때

아래 두개의 선택지중 하나를 따르라.

1. 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화 하게 하라.
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들어라.
    
    단, 클라이언트가 외부 객체 전체에 락을 거는것보다 동시성을 월등히 개선할 수 있을 때만 선택하라.
    

`java.util` 은 첫번째 방식을 취했고, `java.util.concurrent` 는 두번째 방식을 취했다.

### 클래스 내부 동기화

- 락 분할(lock splitting)
- 락 스트라이핑 (lock striping)
- 비차단 동시성 제어 (nonblocking concurrency control)

등을 통해 동시서을 높여줄 수 있다.

# 정리

- 교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자.
- 동기화 영역 안에서의 작업은 최소한으로 줄이자.
- 멀티코어 세상인 지금은 과도한 동기화를 피하는게 과거 어느 때보다 중요하다.