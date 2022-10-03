`java.util.concurrent` 패키지는 실행자 프레임워크라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고있다.

```java
// 작업 큐를 생성
ExecutorService.exec = Executors.newSingleThreadExecutor();

// 실행할 태스크를 넘김
exec.execute(runnable);

// 종료함
exec.shutdown();
```

아이템 49에서 나왔떤 코드보다 모든 면에서 뛰어난 작업큐를 생성하는 코드이다.

# 실행자 서비스의 주요 기능

- 특정 태스크가 완료되기를 기다린다.
- 태스크 모음 중 아무것 하나 혹은 모든 태스크가 완료되기를 기다린다.
- 실행자 서비스가 종료하길 기다린다.
- 완료된 태스크들의 결과를 차례로 받는다.
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다.

큐를 둘 이상의 스레드가 처리하게 하고 싶다면 다른 정적팩터리를 이용하여 다른 종류의 실행자 서비스(스레드풀)를 생성하면 된다. (스레드 풀의 개수는 유동적으로 설정할 수 있다)

# 스레드 풀

- Executors.newCachedThreadPool
    - 작은 프로그램이나 가벼운 서버에 추천
    - 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임되어 실행된다. 가용된 스레드가 없다면 새로 하나를 생성하기 때문에 무거운 프로덕션 서버에는 좋지 못하다.
- Executors.newFixedThreadPool
    - 무거운 서버에 추천.
    - 스레드 개수를 고정한다.

대신 작업큐를 손수 만드는 일과, 스레드를 직접 다루는 것은 일반적으로 삼가야 한다.

# 실행자 프레임워크

작업 단위와 실행 메커니즘이 분리된다.

## 태스크

작업 단위를 나타내는 핵심 추상 개념이며 아래와 같이 두가지가 있다.

1. Runnable
2. Callable

이런 태스크를 수행하는 일반적인 메커니즘이 실행자 서비스다.

태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행정책을 선택할 수 있고, 생각이 바뀌면 언제든 변경할 수 있다.

### 포크조인 태스크

포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다. 모든 서비스를 바쁘게 움직여 CPUfㅡㄹ 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다.

1. ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있다.
2. ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수 있다.