![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1b2a1ee3-457f-44ba-8722-ad00265c62b0/Untitled.png)

- **성능 때문에 견고한 구조를 희생하지 말자. 빠른 프로그램보단 좋은 프로그램을 작성하라.**
    - 좋은 프로그램은 개별 구성요소의 내부를 독립적으로 설계할 수 있고 시스템의 나머지에 영향을 줒 ㅣ않고 각 요소를 다시 설계할 수 있다.
- **성능을 제한하는 설계를 피하라.**
    - 완성후 변경하기 힘든 설계요소는 컴포넌트 혹은 외부 시스템과의 소통 방식이다.
- **API를 설계할 때 성능에 주는 영향을 고려하라.**
    - 가변으로 만들 경우 방어적 복사를 수없이 유발할 수 있다.
        - 컴포지션으로 해결할 수 있음에도 상속방식으로 설계한 클래스는 성능 제약도 물려받는다.
    - 인터페이스가 있는데 구현타입을 굳이 사용하지 말자.
        - 특정 구현체에 종속되게 하여 튜닝이 힘들어진다.

# 즉..

1. 하지마라
2. (전문가 한정) 아직 하지마라
3. 각각의 최적화 시도 전후로 성능을 측정하라

시도한 최적화 기법이 눈에 띄게 성능을 높이는 경우가 별로 없고, 심지어 더 나빠지게 할 때도 있다.

# 프로파일링 도구

개별 메서드의 소비 시간과 호출 횟수 같은 런타임 정보를 제공하여 어디에 최적화를 집중해야하는지 찾는데 도움을 준다.

(근데 써보니까 확실히 문제가 있는부분이 아니면 찾기가 어렵더라)

# 자바의 최적화

자바는 프로그래머가 작성하는 코드와 CPU에서 수행하는 명령 사이의 ‘추상화 격차’가 커서 최적화로 인한 성능 변화를 일정하게 예측하기가 어렵다.

# 정리

- 빠른 프로그램을 작성하려 안달하지 말자. 좋은 프로그램을 작성하다보면 성능은 따라온다.
- 시스템 구현을 완료했다면 성능을 측정하, 충분히 빠르면 그것으로 끝이다.
    - 그렇지 않다면 프로파일러를 사용하여 원인 지점을 찾아 최적화를 수행하라.