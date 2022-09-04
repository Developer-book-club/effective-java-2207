해당 애너테이션은 메서드 선언에만 달 수 있으며, 상위 타입의 메서드를 재정의했음을 뜻한다.

# 해당 소스의 문제점은?

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

26이 나와야하지만 260이 나온다.

- equals와 hashCode를 재정의 하지 못했다.
    - 현재 다중 정의 되어있다.

이러한 부분을 해결하기 위해 `@Override` 를 달아 의도를 명시하자.

(대신 저 위 코드에 equals 매개변수가 Object 타입이 되어야하기 때문에 해당 부분에 대한 변경요구하는 컴파일 오류도 제공해준다.)

**무조건 상위 클래스의 메서드를 재정의 하려는 모든 메서드에 @Override 애너테이션을 달자.**

예외는 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때 뿐이다. 그래도 애너테이션을 달자.

# 정리

- 무조건 @Override 애너테이션을 달자 (실수 했을 떄 컴파일러가 바로 알려준다)