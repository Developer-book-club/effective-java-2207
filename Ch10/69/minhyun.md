**예외는 정말 오직 예외상황에서만 써야한다. 일상적인 제어 흐름용으로 쓰여서는 안된다.**

잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야한.

‘상태 의존적’ 메서드를 제공하는 클래스는 ‘상태 검사’ 메서드도 함께 제공하.

(ex) Iterator 인터페이스 → next, hasNext

# 세가지 중 선택 지침

- **옵셔널이나 특정 값 선택**
    - 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있는 경우
        - 상태 검사 메서드와 상태 의존 메서드 호출 사이에 객체의 상태가 변할 수 있기 때문
    - 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업일부를 중복 수행할 경우

저 위 경우를 제외하곤 상태 검사 메서드 방식이 조금 더 낫다.

- 가독성이 살짝 더 좋다
- 잘못 사용했을때 발견하기 쉽다
- 상태 검사 메서드 호출을 잊었다면 상태 의존 메서드가 예외를 던져 버그를 드러내준다