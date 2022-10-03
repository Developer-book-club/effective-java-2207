자바 5에서 도입된 고수준의 동시성 유틸리티 덕분에 요새는 중요성이 그리 높진 않다.

그러니 **wait과 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.**

# 고수준 유틸리티

- 실행자 프레임 워크
- 동시성 컬렉션
- 동기화 장치(synchronizer)

## 동시성 컬렉션

List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다.

동기화를 각자 내부에서 수행한다. 그리하여 **동시성 컬렉션에서 동시성을 무력화하는건 불가능하고, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.**

### ConcurrentHashMap

ConcurrentHashMap은 get 같은 검색 기능에 최적화되었다. 동시성이 뛰어나고 속도도 무척 빠르다.

### BlockingQueue

take 메서드는 큐의 첫 원소를 꺼내고, 이때 만약 큐가 비었따면 새로운 원소가 추가될때까지 기다린다.

이런 특성 덕에 작업 큐로 쓰기 적합하다.

## 동기화 장치

스레드가 다른 스레드를 기다릴 수 있게 하여 서로 작업을 조율할 수 있게 해준다.

- CountDownLatch
- Semaphere
- CyclicBarrier
- Exchanger
- Phaser

### CountDownLatch

일회성 장벽으로, 하나 이상의 스레드가 또다른 하나이상의 스레드 작업이 끝날때까지 기다리게 한다.

생성자는 int 값을 받으며, 해당 값이 래치의 countDown 메서드를 몇번 호출해야 대기중인 스레드들을 꺠우는지 결정한다.

아래는 동시 실행 시간을 재는 간단한 프레임워크이다.

```java
public class ConcurrentTimer {
    private ConcurrentTimer() { } // 인스턴스 생성 불가

    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done  = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
                try {
                    start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                }
            });
        }

        ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```

### nanoTime

시간 간격을 잴때는 `System.currentTimeMillis` 가 아닌 `System.nanoTime` 을 사용하자.

후자가 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향받지 않는다.

## wait-notify 사용

레거시 코드를 다뤄야 할때는 사용해야 할 것이다.

### wait

- 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용
- 락 객체의 wait 메서드는 그 객체를 잠근 동기화 영역 안에서 호출해야한다.

```java
synchronized(obj) {
	while(<조건이 충족되지 않음>)
		obj.wait(); // 락을 놓고, 꺠어나면 다시 잡는다.

	... //조건이 충족되었을 때의 동작을 수행
}
```

- wait 메서드 사용시 반드시 대기 반복문(wait loop) 관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 마라.
    - 해당 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.

추가적으로 **notify와 notifyAll 중 언제나 notifyAll을 사용하는게 합리적이고 안전**하다.

# 정리

- 코드를 새로 작성한다면, concurrent를 사용하고, 레거시를 유지보수 해야할때만 wait-notify를 사용하자.
- wait은 항상 while문 안에서 호출하자
- 일반적으로 notify 대신 notifyAll을 사용하자