API 설계자의 목소리를 흘려버리지 말자.

(ex) catch 블록에서 아무 일도 하지 않는 식..

**굳이 예외를 무시하기로 했다면 catch블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓자.**

```java
try {
	...
} catch (TimeoutException | ExcutionException ignored) {
	// 기본값을 사용한다. (색상 수를 최소화 하면 좋지만, 필수는 아니다)
}
```
