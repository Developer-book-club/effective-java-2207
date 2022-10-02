더 많은 코드를 재사용하는게 좋은것처럼 예외도 마찬가지.

# 표준예외를 사용할때의 이점

- 우리의 API가 다른 사람이 익히고 사용하기 쉬워진다.
- 예외 클래스 수가 적을 수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.

# 자주 사용되는 예외

- **IllegalArgumentException**
    - 호출자가 인수로 부적절한 값을 넘길 때
    - (ex) 반복 횟수를 지정하는 매개변수에 음수를 건넬 때
- **IllegalStateException**
    - 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때
    - (ex) 제대로 초기화하지 않은 객체를 사용할 때
- **ConcurrentModificationException**
    - 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때
- **UnsupportedOperationException**
    - 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때
    - (ex) 원소를 넣을수만 있는 List 구현체에 누가 remove 메서드를 호출할 경우

해당 예외의 하위 타입도 있을 수 있다. (null일 경우 IllegarArgument 보단 NPE를 던지는 식)

## IllegalArgument와 State를 선택하는 방법

인수값이 무엇이었든 어차피 실패했을 거라면

? IllegalStateException

: IllegalArgumentException

# 재사용 금지

- Exception
- RuntimeException
- Throwable
- Error

직접 재사용하지 말자. 단순 추상 클래스라고 생각하. 여러 성격의 예외들을 포괄하는 클래스므로 안정적으로 테스트할 수 없다.

# 참고

- 예외는 직렬화가 가능하다. 이 사실만으로 나만의 예외를 새로 만들지 않아야 할 근거로 충분하다.