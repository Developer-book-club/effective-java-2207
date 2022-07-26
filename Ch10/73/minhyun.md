수행하려는 일과 관련 없어 보이는 예외가 튀어나오면 당황스러울 것이다.

→ 메서드가 저수준 예외를 처리하지 않고 전파해버릴때 종종 일어난다.

**상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던지자. (예외 번역)**

# 예외 번역

```java
try {
	... // 저수준 추상화 이용
} catch (LowerLevelException e) {
	// 추상화 수준에 맞게 번역
	throw new HigherLevelException(...);
}
```

- 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하면 좋다.
    - 예외 연쇄란 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식

## 예외 연쇄

```java
try {
	... // 저수준 추상화 이용
} catch (LowerLevelException cause) {
	// 저수준 예외를 고수준 예외에 실어 보낸다.
	throw new HigherLevelException(cause);
}
```

```java
// 예외 연쇄용 생성자
Class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

- 별도의 접근자 메서드(Throwable의 getCause 메서드)를 통해 필요하면 언제든 꺼내볼 수 있다.

# 그래서

- 가능하다면 아래 계층에서는 예외가 발생하지 않도록 하는것이 최선이다.
    - 상위계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전 미리 검사하는 방법이 있다.
- 아래 계층에서 예외를 피할 수 없다면 상위 계층에서 그 예외를 조용히 처리하여 API 호출까지 전파하지 않는 방법이 있다.
    - 이경우 로깅 기능을 활용하여 기록해두도록 한다.

# 정리

- 아래 계층 예외를 예방하거나 스스로 처리할 수 없고, 상위 계층에 그대로 노출하기 곤란하면 예외 번역을 사용하자.
- 예외 연쇄를 사용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지며 근본 원인도 함께 알려주면 좋다.