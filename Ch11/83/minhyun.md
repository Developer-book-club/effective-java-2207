지연초기화란 필드의 초기화 시점을 그 값이 청므 필요할 때까지 늦추는 기법이다.

그리하여 값이 전혀 쓰이지 않으면 초기화도 일어나지 않는다.

모든 최적화와 마찬가지로 지연 초기화도 “필요할때까지는 하지말아라” 

지연 초기화는 양날의 검인데, 인스턴스 생성시의 초기화 비용은 줄지만 지연 초기화하는 필드에 접근하는 비용이 커진다.

결국 성능이 느려질 수 있다.

# 지연 초기화가 필요할 때

그 필드를 사용하는 인스턴스의 비율이 낮지만, 필드를 초기화하는 비용이 클때는 효과기 있을 것이다.

대신 정말 그런지 알 수 있는 유일한 방법은 지연초기화 적용 전후의 성능을 측정해보는 것이다.

대부분의 상황에서는 일반적인 초기화가 지연 초기화보다 낫다.

## 지연 초기화가 초기화 순환성을 깨뜨릴 것 같다면 synchronized를 단 접근자를 사용하자.

```java
// 일반적인 초기화
private final FieldType field = computeFieldValue();

// 인스턴스 필드의 지연 초기화 - synchronized 접근자 방식
private FieldType field;

private synchronized FieldType getField() {
	if (field == null)
		field = computeFieldValue();

	return field;
}
```

## 성능때문에 정적필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자.

```java
// 지연초기화 홀더 클래스 관용구
private static class FieldHolder {
	static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

- 필드에 접근하며 동기화를 전혀 하지 않아 성능이 느려질거리가 전혀 없다.
- 오직 클래스를 초기화할때만 필드 접근을 동기화한다

## 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하자.

```java
private volatile FieldType field;

private FieldType getField() {
	FieldType result = field;
	if(result != null) { //첫번째 검사 (락 사용 안함)
		return result;
	}

	synchronized(this) {
		if(field == null) //두번째 검사 (락 사용)
			field = computeFieldValue();
		return field;
	}
}
```

- 초기화된 필드에 접근할 떄의 동기화 비용을 없애준다.
- 두번째 검사에서도 필드가 초기화되지 않았을때만 필드를 초기화한다.
- 필드가 초기화 된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile 로 선언한다.

# 정리

- 대부분 필드는 지연시키지 않고 곧바로 초기화 해야한다.