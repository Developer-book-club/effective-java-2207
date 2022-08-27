# 태그 달린 클래스

두가지 이상의 의미를 표현하며, 현재 표현하는 의미를 태그값으로 알려주는 클래스를 본 적이 있을 것이다.

→ 대략 flag 사용 느낌이다.

![image-20220813231945709](https://tva1.sinaimg.cn/large/e6c9d24egy1h55i2zltf0j20go0hzwfc.jpg)

## 단점

- 열거타입 선언, 태그 필드, switch문 등 쓸데없는 코드가 많다.
- 여러 구현이 한 클래스에 혼합되어 있어 가독성이 나쁘다
- 다른 의미의 코드도 언제나 함께하여 메모리도 많이 사용한다
- 쓰지않는 필드를 초기화하는 불필요한 코드가 늘어난다.
- 새로운 의미를 추가할 때마다 switch문에 새 의미를 처리하는 코드를 추가해야 한다. (문제 생길 가능성 99%)
- 인스턴스의 타입만으로 현재 나타내는 의미를 알 수 없다.

즉, **태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적**이다.

# 클래스 계층 구조

클래스 계층 구조를 활용하는 서브 타이핑을 사용하자.

## 태그달린 클래스를 바꾸는 방법

1. 계층구조의 루트(root)가 될 추상 클래스를 정의
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언
3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의
6. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다
7. 루트 클래스가 정의한 추상 메서드를 각자 의미에 맞게 구현

## 변환 예제

위 태그달린 클래스를 클래스 계층 구조로 변경하였다. (근데 본인이라면 인터페이스로 빼버릴듯..)

```java
abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;

	Circle(double radius) { this.radius = radius; }

	@Override
	double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
	final double length;
	final double width;

	Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}

	@Override
	double area() { return length * width; }
}
```

- 간결하고 명확하며, 쓸데없는 코드도 모두 사라졌다.
- 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했다.
- 유지보수가 편해졌다.
- 타입 사이의 자연스러운 계층관계 반영이 가능해 유연성은 물론 컴파일 타입 타입 검사 능력을 높여준다.

# 정리

- 새로운 클래스를 작성할 때 태그 필드가 등장하면 계층 구조로 대체하자.
