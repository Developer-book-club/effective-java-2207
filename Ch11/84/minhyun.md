정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다.

견고하고 빠르고 이식성 좋은 프로그램을 작성하는 가장 좋은 방법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다.

→ 스레드 스케줄러가 고민할 거리가 줄어든다.

실행 가능한 스레드 수를 적게 유지하는 주요 기법은 작업 완료 후 다음 일거리가 생길 때 까지 대기하도록 하는 것이다.

→ 스레드는 당장 처리해야 할 작업이 없다면 실행돼서는 안된다.

# 정리

- 스레드 스케줄러에 프로그램 동작을 기대지 말자. 견고성과 이식성이 모두 해쳐진다.
- 스레드 우선순위는 잘 동작하는 서비스 품질을 높이기 위해 드물게 쓰일 수 있지만, 간신히 동작하는 서비스를 ‘고치는 용도’로 사용해선 절대 안된다.