# 명명패턴

Junit3에서 test메서드 이름을 ‘test’로 무조건 시작하게끔 정의하는 것.

## 단점

1. 오타가 나면 안된다.
2. 올바른 프로그램 요소에서만 사용되리라 보증 할 방법이 없다.
    1. 특정 프로그램은 앞에 test를 붙이던 말던 보지 않는다. 이경우 테스트 수행이 되지 않는다.
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.
    - ?? 이해 안됨.

# 애너테이션

```java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- @Retention과 @Target처럼 애너테이션 선언에 다는 애너테이션을 메타 애너테이션이라고 한다.
    - @Retention → @Test 가 런타임에도 유지되어야 한다는 표시.
    - @Target(ElementType.METHOD) → 메서드 선언에서만 사용되어야 한다는 표시.

추가적으로 메서드 주석처럼 ‘매개변수 없는 정적 메서드 전용'을 컴파일러가 강제하게 하려면 적절한 애너테이션 처리기를 직접 구현해야한다.

→ 관련 방법은 `javax.annotaion.processing` API 문서 참고

## 마커 애너테이션

위의 @Test 애너테이션은 **‘아무 매개변수 없이 단순히 대상에 마킹' 한다고 하여 마커 애너테이션이라고 한다.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d41419e2-4382-4f2f-8a12-3b8109b84cb3/Untitled.png)

@Test 애너테이션은 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다.

그저 이 애너테이션에 관심있는 프로그램에게 추가 정보를 제공할 뿐이다.

## 마커 애너테이션을 처리하는 프로그램

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) { //이부분 중요
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```

- Exception exc 쪽으로 빠지게 되었다면 애너테이션을 잘못 사용했다는 뜻.
    - 인스턴스 메서드, 매개변수가 있는 메서드, 호출할 수 없는 메서드 등에 달았을 것

# 매개변수 하나를 받는 애너테이션 타입

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

## 실제 활용 사례

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
```

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) { //중요
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedEx) {
                    Throwable exc = wrappedEx.getCause();
                    Class<? extends Throwable> excType =
                            m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf(
                                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }

        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```

- 형변환 코드가 없어 ClassCastException 걱정이 없다.
- 클래스 파일이 컴파일엔 존재하지만 런타임엔 존재하지 않을 수 있는데 이 경우 TypeNotPresentException을 던진다.

# 배열 매개변수를 받는 애너테이션 타입

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
```

```java
@ExceptionTest({ IndexOutOfBoundsException.class,
                     NullPointerException.class })
    public static void doublyBad() {   // 성공해야 한다.
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
```

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
```

# Repeatable 애너테이션

자바 8에서는 여러개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다.

배열 매개변수 사용하는 대신 애너테이션에 `@Repeatable` 메타 애너테이션을 다는 방식이다.

해당 애너테이션은 하나의 프로그램 요소에 여러번 달 수 있다.

## 사용시 주의점

- @Repeatable을 단 애너테이션을 반환하는 ‘컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
- 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.
    - 안할 경우 컴파일 안됨.

## 예시

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

//컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

```java
// 코드 39-9 반복 가능 애너테이션을 두 번 단 코드 (244쪽)
    @ExceptionTest(IndexOutOfBoundsException.class)
    @ExceptionTest(NullPointerException.class)
    public static void doublyBad() {
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
```

```java
if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
```

- 코드 가독성은 높아지지만, 애너테이션을 선언하고 처리하는 부분에서 코드양이 늘어나며 처리 코드가 복잡해져 오류가 날 가능성이 있다.

# 정리

- **애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.**
- 다른 프로그래머가 소스코드에 추가정보를 제공할 수 있는 도구를 만드는 일을 한다면 적당한 애너테이션 타입도 함께 정의해 제공하자.