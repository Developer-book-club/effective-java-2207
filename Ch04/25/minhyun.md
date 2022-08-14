**톱레벨 클래스가 한 파일에 여러개 있다면 어느 것을 사용할지는 어느 소스파일을 먼저 컴파일하냐에 따라 달라지기 때문에 위험**하다.

# 예시

```java
//실행 구문
public class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
}
```

```java
// Utensil.java
class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```

여기까지 구현하고 실행 구문을 실행한다면 “pancake”를 잘 출력한다. 하지만 아래 파일까지 추가된다면?

```java
// Dessert.java
class Utensil {
	static final String NAME = "pot";
}

class Dessert {
	static final String NAME = "pie";
}
```

`javac Main.java Dessert.java` 명령으로 컴파일 된다면 컴파일 오류가 발생할 것이다. → 중복 정의

## 하지만

1.  `javac Main.java Utensil.java` 명령으로 실행된다면
    - “pancake”가 출력된다.
2.  `javac Desert.java Main.java` 명령으로 실행된다면
    - “potpie”가 출력된다.

즉 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라진다.

## 해결책

단순히 톱레벨 클래스들을 서로 다른 소스파일로 분리하면 된다.

굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스(아이템 24)를 사용하는 방법을 고민하자.

```java
// 톱레벨 클래스들을 정적 멤버 클래스로 바꾼 모습
public class Test {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}

	private static class Utensil {
		static final String NAME = "pan";
	}

	private static class Dessert {
		static final String NAME = "cake";
	}
}
```

# 정리

- 소스파일 하나에는 반드시 톱레벨 클래스를 하나만 담자.
