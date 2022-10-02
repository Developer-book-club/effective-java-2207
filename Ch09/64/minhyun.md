적합한 인터페이스가 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하라.

객체의 실제 클래스를 사용해야 할 상황은 오직 ‘생성자로 생성할 때 뿐’ 이다.

**인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해진다.**

```java
Set<Son> sonSet = new LinkedHashSet<>();
```

대신 적합한 인터페이스가 없다면 당연히 클래스로 참조하자.