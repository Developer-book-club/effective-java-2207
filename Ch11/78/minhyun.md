`synchronized` 키워드는 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.

단순히 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도만이 아니다.

동기화는 일관성이 꺠진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

**즉, 동기화는 배타석 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다는 것이다.**

# 스레드를 올바르게 멈추는 방법

기존 `Thread.stop` 메서드를 사용하지 말자. 이미 deprecate 되었다.

## 잘못된 코드

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

1초후에 종료될 것이라 생각하지만 위 코드는 계속되어 실행된다.

→ 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 볼수 있을지 보증할 수 없다.

동기화 하지 않으면 가상 머신이 다음과 같이 최적화를 수행할 수 있기 때문이다. (호이스팅 기법)

```java
//원래 코드
while (!stopRequested)
	i++;

//최적화한 코드
if (!stopRequested)
	while(true)
		i++;
```

## 올바른 코드 (synchronized 사용)

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

위와 같이 ‘stopRequested’ 필드를 동기화해 접근하면 문제가 해결된다.

쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다.

## 다른 방법 (volatile 사용)

**필드를 `volatile` 로 선언하는 것**이다. 동기화 비용이 크지 않지만 속도가 더 빠르다. 이 경우 동기화를 생략해도 된다.

해당 한정자는 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

```java
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

### volitile 은 주의해서 사용하자.

고유한 값을 반환받는 기능이 있다면 해당 한정자 사용 대신 동기화를 제대로 시키자.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/17818e07-d588-42ca-81a3-b1d7307ea0a2/Untitled.png)

위 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다. 하지만 증가 연산자(++)가 문제가 될 수 있다. 실제로 해당 필드에 두번 접근하게 된다.(값을 읽을때, 증가시킨후 새로운 값을 저장할 때)

어떤 스레드가 이 두 접근 사이를 비집고 들어와 똑같은 값을 반환할 수 있기 때문이다.

→ generated… 메서드에 `synchronized` 를 붙이면 해결된다. 이후 volatile을 제거하자.

### atomicLong 사용

`java.util.concurrent.atomic` 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨있다.

`volatile`은 동기화의 두 효과중 통신쪽만 지원하지만, 이 패키지는 원자성(배타적 실행)까지 지원한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d200447-dbd2-4864-b5b4-90602a0c0176/Untitled.png)

# 그래서

결국 제일 좋은 방법은 가변 데이터를 공유하지 않는 것이다. 불변데이터만 공유하거나 아무것도 공유하지 말자.

**즉, 가변 데이터는 단일 스레드에서만 쓰자.**

사실상 불변객체를 다른 스레드에 건네는 행위를 ‘안전 발행’ 이라 하는데, 방법은 많다.

클래스 초기화 과정에서 객체를 정적필드, volatile 필드, final 필드 혹은 보통의 락을 통해 접근하는 필드에 저장하거나, 동시성 컬렉션에 저장하는 방법이 있다.

---

# 정리

- 여러 스레드가 가변 데이터를 공유한다면 읽고 쓰는 동작은 반드시 동기화 해야한다.