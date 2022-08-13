[기존 블로그 글](https://mintheon.com/devlog/2021/08/15/%EC%B6%94%EC%83%81%ED%81%B4%EB%9E%98%EC%8A%A4%EC%99%80-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EC%9D%98-%EC%B0%A8%EC%9D%B4/)

# 추상클래스와 인터페이스의 차이

간단한 부분은 표로 아래의 표로 확인하고, 디테일한 부분은 따로 설명한다.

|  | 추상클래스 | 인터페이스 |
| --- | --- | --- |
| 구현 클래스 | 반드시 추상 클래스의 하위 클래스가 되어야 한다. | 선언된 메서드를 모두 정의하고 일반 규약을 잘 지켰다면 어떤 클래스를 상속했든 같은 타입으로 취급된다. |
| 상속 | 단일 상속만 지원하여 확장하기 어렵다. | 인터페이스는 복수로 사용 가능하다. |
| 추가 | 기존 클래스에 새로운 추상 클래스를 끼워넣기 어렵다. | 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다. |
| 믹스인 | 정의할 수 없다. 기존 클래스에 덧씌울 수 없기 때문이다. | 안성맞춤이다.  |
| 계층 구조 | 가능한 조합 전부를 각 클래스로 정의해주어야 하여 조합 폭발 현상이 생긴다. | 계층 구조가 없는 타입 프레임 워크를 만들 수 있다. |

> 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 ‘주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
> 

# 인터페이스

- 래퍼 클래스와 함께 사용하면 기능을 향상시키는 안전하고 강력한 수단이 된다.
- 메서드 중 구현방법이 명백한 것이 있다면 해당 구현을 디폴트 메서드로 제공해 사용 편의성을 증대 시킬 수 있다.
- 인터페이스와 추상골격 구현 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다.
    - 인터페이스로는 타입을 정의, 필요하면 디폴드 메서드 몇개도 함께 제공
    - 골격 구현 클래스는 나머지 메서드까지 구현
    - **이것이 바로 템플릿 메서드 패턴이다.**

# 골격 구현 클래스

인터페이스 이름이 ‘Interface’ 라면 골격 구현 클래스의 이름은 ‘AbstractInterface’로 짓는다.

(저자는 ‘Skeletal’Interface 가 더 낫지 않나 어필하고 있다.)

## 코드

```java
//골격 구현을 사용해 완성한 구체 클래스 예시
static List<Integer> intArrayAsList(int[] a) {
	Objects.requireNonNull(a);

	return new AbstractList<>() {
		@Override
		public Integer get(int i) {
			return a[i];
		}

		@Override
		public public Integer set(int i, Integer val) {
			int oldVal = a[i];
			a[i] = val;
			return oldVal;
		}

		@Override 
		public int size() {
			return a.length;
		}
	}
}

//골격 구현 클래스 예시
public abstract class AbstractMapEntry<K,V> implement Map.Entry<K,V> {
  
  //변경가능한 엔트리는 이 메서드를 반드시 재정의 해야 한다.
  @Override
  public V setValue(V value) {
    throw new UnsupportedOperationException();
  }
  
  //Map.Entry.equals의 일반 규약을 구현
  @Override
  public boolean equals(Obejct o) {
    if(o == this)
      return true;
    if(!(o instanceof Map.Entry))
      return false;
    Map.Entry<?,?> e = (Map.Entry) o;
    return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getvalue(), getValue());
  }
  
  //Map.Entry.hashCode의 일반 규약을 구현
  @Override
  public int hashCode() {
    return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
  }
  
  @Override
  public String toString() {
    return getKey() + "=" + getValue();
  }
}
```

- 추상 클래스처럼 구현을 도와주는 동시에 추상 클래스로 타입을 정의할 때 생기는 심각한 제약에서는 자유롭다.
- 다중 상속의 많은 장점을 제공하는 동시에 단점은 피하게 해준다.

## 순서

1. 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정
2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공
3. 남은 메서드들은 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 작성해 넣는다.
    1. 골격 구현 클래스에는 필요하면 public이 아닌 필드와 메서드를 추가해도 된다.

## 단순 구현

골격 구현의 작은 변종이다. 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만 추상클래스가 아니다.

*(ex) AbstractMap.SimpleEntry*

# 정리

- 일반적으로 다중 구현용 타입은 인터페이스가 적합하나, 복잡한 인터페이스라면 구현 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려하자.
- 골격 구현은 ‘가능한 한' 인터페이스의 디폴트 메서드로 제공하여 해당 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.
    - ‘가능한 한' 이라고 한 이유는, 인터페이스의 구현상 제약때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 떄문이다.