`equals()` 메서드는 재정의 하기가 생각보다 쉽지않다. 회피하기 쉬운 방식은 아예 재정의 하지 않는 것이지만 나름의 가이드를 제시한다.

# equals를 재정의 하지 않아야 할 경우

- **각 인스턴스가 본질적으로 고유할 때**
    - (ex) Thread, Object.equals()
- **인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없을 때**
    - `Object.equals()` 만으로도 해결이 가능하다.
- **상위 클래스에서 재정의한 `equals()` 가 하위 클래스에도 딱 맞을 때**
    - (ex) 대부분의 List 구현체들은 `AbstractList`로부터 equals를 상속받아 쓴다.
- **클래스가 private거나 package-private 이고 equals 메서드를 호출할 일이 없을 때**
    
    ```java
    // 이 경우 위험을 철저히 막기 위해 아래와 같이 처리하자.
    @Override
    public boolean equals(Object o) {
    	throw new AssertionError();
    }
    ```
    

# equals를 재정의 해야 할 경우

- **논리적 동치성을 확인하여야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않았을 때**
    - (ex) Integer, String 등의 값 클래스 → 객체가 같은지가 아니라 값이 같은지를 알기 위해
    - (예외) 인스턴스가 둘 이상 만들어지지 않음이 보장되는 클래스 (Enum 등)

# Equals 메서드 재정의 시 일반 규약

## Object 명세에 적힌 규약

equals 메서드는 동치관계(Equivalence relation)를 구현하며 다음을 만족한다.

- **반사성(reflexivity)**
    - null이 아닌 모든 참조값  x에 대해 `x.equals(x)` 는 true이다.
- **대칭성(symmetry)**
    - null이 아닌 모든 참조값 x,y에 대해 `x.equals(y)` 가 true이면 `y.equals(x)` 도 true이다.
- **추이성(transitivity)**
    - null이 아닌 모든 참조값 x,y,z에 대해 `x.equals(y)` 가 true이고 `y.equals(z)` 도 true면, `x.equals(z)` 도 true이다.
- **일관성(consistency)**
    - null이 아닌 모든 참조값 x,y에 대해 `x.equals(y)` 를 반복해서 호출하면 항상 true 혹은 false를 반환한다.
- **null-아님**
    - null이 아닌 모든 참조값 x에 대해 `x.equals(null)` 은 false다.

> 동치관계란 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산이다. 이 부분 집합을 ‘동치 클래스' 라고 한다.
> 

동치 관계를 만족시키기 위한 다섯 요건을 하나씩 살펴 보자.

## 1. 반사성

객체는 자기 자신과 같아야 한다는 뜻.

## 2. 대칭성

두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻.

### 예시

`CaseInsensitiveString` 이라는 대소문자 구별이 없는 String 객체가 있다고 하자. 내부 `equals()` 에서는 `equalsIgnoreCase` 를 사용하여 대소문자 구별하지 않고 비교한다.

```java
// CaseInsensitiveString 클래스
class CaseInsensitiveString {
	...
	@Override
	public boolean equals(Object o) {
		if (o instanceof CaseInsensitiveString)
			return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
		
		if (o instanceof String) // 한방향으로만 작동한다.
			return s.equalsIgnoreCase((String) o);
	
		return false;
	}
}

//실행구문
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s); // true
s.equals(cis); // false
```

→  **String 클래스의 equals는 대소문자 구별을 하도록 구성되어 있기 때문에 해당 부분은 대칭성을 명백히 위반하게 된다.**

이러한 부분을 고치려면 해당 부분을 아래와 같이 고쳐야한다.

`if(o instanceof String)` 

→ `if(o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);`

## 3. 추이성

첫번째, 두번째 객체가 같고 두번째, 세번째 객체가 같다면 첫번째 객체와 세번째 객체도 같아야 한다는 뜻.

### 예시

```java
class Point {
	private final int x;
	private final int y;
	
	...
	@Override
	public boolean equals(Object o) {
		if(!(o instanceof Point))
			return false;

		Point p = (Point)o;
		return p.x == x && p.y == y;
	}
}
```

해당 Point라는 객체를 확장해서 color라는 필드를 추가한다면?

```java
class ColorPoint extend Point {
	private final Color color;

	...
}
```

이 경우 색상 정보는 무시한 채 부모클래스의 `equals()` 를 사용하여 비교하게 된다. **이부분에서 대칭성이 깨진다.**

해결하고자 Point 클래스에 ColorPoint가 아닐경우 Color를 비교하지 않도록, 그리고 ColorPoint라면 Color까지 비교하도록 코드를 수정한다면?

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2); // true
p2.equals(p3); // true
p1.equals(p3); // false
```

**위와 같이 추이성에 위배된다.** p1과 p2, p2와 p3 비교에서는 색상을 무시했지만, p1과 p3 비교에서는 색상까지 고려했기 때문이다.

또한 무한 재귀에 빠질 위험도 있다.

**구체클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**

- **잘못된 방식**
    - `*getClass()` 를 사용하여 클래스 자체를 비교하는 방식*
        
        → Point의 하위 클래스는 Point 이기 때문에 어디서든 Point로써 활용되어야하지만 리스코프 치환 원칙에 위배된다.
        
        > 리스코프 치환원칙이란, 어떤 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야한다는 개념이다.
        > 
- **나은 방식**
    - 상속대신 컴포지션을 사용하라 (아이템 18)
        
        → Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고 일반 Point를 반환하는 뷰(view) 메서드를 public으로 추가하는 식
        
        ![image](https://user-images.githubusercontent.com/23326757/183250357-22907ba7-66f0-423e-8428-34ddc82b0618.png)

        (ex) java.sql.Timestamp
        

## 4. 일관성

equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다는 뜻.

### 5. null-아님

모든 객체가 null과 같지 않아야 한다는 뜻.

추가적으로 NullPointException을 던지는 경우조차 허용하지 않는다.

→ 내부에서 null을 확인하려 하지 말고 `instanceof` 를 통해 형변환 하기 전 매개변수가 올바른 타입인지 검사하라. 

(`instanceof`는 피연산자가 null이면 false를 묵시적으로 반환한다)

# equals 메서드 구현 방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 ‘핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

## 예외사항

- float와 double의 경우 부동소수점 문제가 있기 때문에 `Double.compare(double, double)` 식의 compare 메서드를 사용하여 비교하자.
- null도 정상값으로 취급하는 참조타입 필드도 있기 때문에 `Objects.equals(Object, Object)` 로 비교하여 NPE 발생을 예방하자.

## 팁

- 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.
- 단위 테스트를 작성해 돌려보자.
- equals를 재정의할땐 hashCode도 반드시 재정의하자(아이템 11)
- 너무 복잡하게 해결하려 들지 말자
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
    
    ```java
    public boolean equals(MyClass o) {
    	...
    }
    ```
    
    → 이 메서드는 Object.equals를 재정의한게 아니라 다중정의 한 것이다. (파라미터가 다름)
    

# 정리

꼭 필요한 경우가 아니면 equals를 재정의 하지 말자. 재정의 해야 할 때에는 핵심 필드를 빠짐없이, 다섯가지 규약을 확실히 지키며 비교하자.