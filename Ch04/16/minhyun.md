```java
class Point {
	public double x; //절대 안된다!!!!!
	public double y; //절대 안된다!!!!!
}
```

데이터 필드에 직접 접근할 수 있어 캡슐화를 깨는 위와 같은 코드는 절대로 쓰지말자.

```java
class Point {
	private double x;
	private double y;

	public double getX() {
		return x;
	}
	...
}
```

**패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공하여 내부 표현 방식을 마음대로 바꿀수 없도록 하자.**

> 자바 플랫폼 라이브러리에서도 규칙을 어긴 사례가 있다. `Point` 와 `Dimension` 클래스이며, 해당 클래스의 심각한 성능문제는 오늘날까지도 해결되지 못했다.
> 

# 정리

- public 클래스는 절대 가변 필드를 직접 노출해선 안된다.
- package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.