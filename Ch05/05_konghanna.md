# 아이템 26: 로 타입은 사용하지 말라

### 제네릭 타입(generic type)
- 제네릭 클래스 + 제네릭 인터페이스, 클래스와 인터페이스 선언에 타입 매개변수가 쓰인 경우
  - ex. `List<E>`

### 로 타입(raw type)
- 제네릭 타입에서 타입 매개변수를 사용하지 않을 때, 제네릭 타입 정보가 지워진 것처럼 동작
  - 제네릭 도입 이전 코드와의 호환성을 위해 제공될 뿐
  - ex. List 

```java
// 제네릭 X : 컴파일 오류 발생 X, unchecked call 경고만 
private final Collection stamps = ...;
stamps.add(new Coin(...));

// 제네릭 O : 컴파일 오류 발생 O
private final Collection<stamps> stamps = ...;
stamps.add(new Coin(...));
```

### 로 타입은 사용해서는 안된다
1. List 같은 로 타입은 안되나 `List<Object>`처럼 임의 객체를 허용하는 매개변수화 타입은 OK
    - List는 제네릭 타입과 관련 X, `List<Object>`는 모든 타입을 허용한다는 것
    - 제네릭의 하위 타입 규칙
       - 매개변수로 List를 받는 메서드에는 `List<String>`을 넘길 수 있지만 `List<Object>`에는 넘길 수 없음
       - `List<String>` 은 로 타입인 List의 하위 타입이지만 `List<Object>`의 하위 타입은 아님
       - -> 로 타입(List) 사용 시, 매개변수화 타입(`List<Object>`)을 사용할 때와 달리 타입 안전성을 잃게 됨 

### 비한정적 와일드카드 타입
- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않을 때 사용
  - ex. Set<?>
- Collection<?> : null 외에는 어떤 원소도 넣을 수 없음 (컴파일 시 에러 발생) 
  - 아무 원소나 넣을 수 있어서 타입 불변식을 훼손하기 쉬운 로 타입보다 안전함

### 예외 규칙: 로 타입을 사용해야 하는 경우 
1. class 리터럴에는 매개변수화 타입이 아닌 로 타입을 써야 함
   - List.class, String[].class, int.class (O)
   - `List<String>.class, List<?>.class` (X)
2. instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없음  (런타임에 제네릭 타입 정보가 지워지기 때문)
    - 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 똑같이 동작함 -> 로 타입을 쓰는 편이 깔끔 

```java
// instanceof를 사용하여 타입이 Set인지 확인한 후,
// 와일드카드 타입으로 형변환해야 함
if( o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```
<br><br>

# 아이템 27: 비검사 경고를 제거하라

- 비검사 경고를 제거하면 런타임에 ClassCastException이 발생하지 않음
- 경고를 제거할 수는 없지만 타입이 안전하다고 확신할 수 있으면 @Suppress Warnings("unchecked") 어노테이션을 달아 경고 숨기기 

### @SuppressWarnings 어노테이션
- 개별 지역변수 선언부터 클래스 전체까지 모든 선언에 달 수 있으나 가능한 좁은 범위에 적용할 것
  - 중요한 경고를 놓칠 수 있으니 클래스 전체에 적용해서는 안 됨 
- 경고를 무시해도 안전한 이유를 주석으로 남겨야 함

<br><br>

# 아이템 28: 배열보다는 리스트를 사용하라

### 배열 vs 제네릭
 
1. 배열은 공변, 제네릭은 불공변 
    - 배열: 공변(=함께 변함, covariant) 
         - Sub가 Super의 하위 타입이면 배열 Sub[]는 배열 Super[]의 하위 타입 
    - 제네릭: 불공변(invariant)
         - 서로 다른 Type1과 Type2가 있을 때, `List<Type1>`은 `List<Type2>`의 상위 타입도 하위 타입도 아님     
```java
//배열
Object[] objectArray = new Long[1];
objectArray[0] = "문자열문자열"; // 컴파일은 O, 런타임 시 오류 발생

//제네릭
List<Object> objectList = new ArrayList<Long>();
objectList.add("문자열문자열"); // 컴파일조차 X
```

2. 배열은 실체화(reify)되나 제네릭은 타입이 소거되어 실체화가 불가능함
    - 배열: 실체화됨
         - 런타임에도 담기로 한 원소의 타입을 인지하고 확인 
         - 런타임에 타입 안전(컴파일타임에는 X)
    - 제네릭: 실체화 불가
         - 컴파일타임에만 원소 타입을 검사, 런타임에는 알 수 없음 (실체화되지 않음=타입 정보가 소거됨) 
         - 컴파일타임에 타입 안전(런타임에는 X)
         - 매개변수화 타입 중 실체화될 수 있는 타입은 비한정적 와일드카드 타입뿐 
<br>

- 배열과 제네릭에는 상반된 타입 규칙이 적용됨 -> 둘을 섞어 쓰는 제네릭 배열 생성 불가능 -> 배열 대신 List 사용할 것

<br><br>

# 아이템 29: 이왕이면 제네릭 타입으로 만들라

### 일반 클래스를 제네릭 클래스로 만들기
 
1. 클래스 선언에 타입 매개변수 추가  
2. 일반 타입을 타입 매개변수로 변경 

```java
// Object 기반 스택 -> 제네릭으로 수정
public class Stack { // 1. Stack<E> 로 수정
    private Object[] elements; // 2. E[] elements 로 수정
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object [DEFAULT_INITIAL_CAPACITY]; 
         // 2. (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; 로 수정
    }

    public void push(Object e) { // 2. push(E e) 로 수정
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() { // 2. E pop() 로 수정
        if (size ==0) {
            throw new EmptyStackException();
        }

        
        Object result = elements[--size]; // 2. E result = elements[--size]; 로 변경한다.
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
3. 제네릭 배열(E[] elements) 생성 오류 해결
    - 배열 생성 시 형변환: Object 배열 생성 후 제네릭 배열로 형변환 
    - 배열에서 원소를 읽을 때 형변환: elements 필드의 타입을 E[]에서 Object[]로 변경

### 타입 매개변수 사용 규칙
- 대다수의 제네릭 타입은 타입 매개변수에 제약을 두지 않지만 기본 타입은 사용할 수 없음 -> 박싱된 기본 타입을 사용해 우회
- 타입 매개변수에 제약을 두는 제네릭 타입도 존재함 ex. `class DelayQueue <E extends Delayed>~`
    - 한정적 타입 매개변수  

<br><br>

# 아이템 30: 이왕이면 제네릭 메서드로 만들라

### 제네릭 싱글톤 팩터리 
- 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있음
- 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리가 필요함 -> 제네릭 싱글톤 팩터리 
- ex.  Collections.reverseOrder, Collections.emptySet
 
### 재귀적 타입 한정(recursive type bound)
- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정하는 것
- 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰임
```java
//재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현
//<E extends Comparable<E>> : 모든 타입 E는 자신과 같은 타입의 원소와만 비교할 수 있다 
public static <E extends Comparable<E>> E max(Collection<E> c);
```

<br><br>

# 아이템 31: 한정적 와일드카드를 사용해 API 유연성을 높이라

- 매개변수화 타입은 불공변 -> API에서 타입을 유연하게 사용하기 위해 한정적 와일드카드 타입 적용

### 생산자(producer) 매개변수에 와일드카드 타입 적용 
- 생산자: 인스턴스를 생산해서 원소를 옮겨 담는 매개변수
```java
// ex. Stack 클래스 - public API에 pushAll 메소드를 추가할 경우
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```
- ex. Number 타입으로 선언된 Stack 객체의 pushAll 메서드에 Integer 타입의 매개변수를 전달한다면? 컴파일 오류 발생
  - Integer는 Number의 하위 타입이지만 매개변수화 타입은 불공변이기 때문에 Integer와 Number를 별개의 타입으로 취급 -> 한정적 와일드카드 자료형 사용
```java
// Iterable<? extends E>: pushAll의 입력 매개변수 타입은 'E의 하위 타입의 Iterable' 
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

### 소비자(Consumer) 매개변수에 와일드카드 타입 적용 
- 소비자: 인스턴스의 원소를 소비하는(가져오는) 매개변수
```java
// ex. Stack 클래스 - public API에 popAll 메소드를 추가할 경우
public void popAll(Collection<E> dst) {
   while(!isEmpty()) {
        dst.add(pop());
    }
}
```
- ex. Number 타입으로 선언된 Stack의 원소를 popAll 메서드를 통해 Object용 컬렉션으로 옮긴다면? 컴파일 오류 발생

```java
// Collection<? super E>: popAll의 입력 매개변수 타입은 'E의 상위 타입의 Collection' 
public void popAll(Collection<? super E> dst) {
   while(!isEmpty()) {
        dst.add(pop());
    }
}
```

### 펙스: PECS(Producer-Extends-Consumer-Super) 
- 와일드카드 타입을 사용하는 기본 원칙(=겟풋원칙)
- 메서드의 매개변수 타입이 생산자면 <? extends T>
- 메서드의 매개변수 타입이 소비자면 <? super T>
- 특히 Comparable(Comparator)를 구현한 다른 타입을 지원하기 위해 와일드카드 타입을 적용하는 것이 좋음 
    - Comparable과 Comparator은 모두 소비자

### 타입 매개변수 vs 와일드카드
```java
//타입 매개변수 방식
public static <E> void swap(List<E> list, int i, int j) {
   list.set(i, list.set(j, list.get(i)));
}
//와일드카드 방식 - public API에서 사용할 것을 권장
public static void swap(List<?> list, int i, int j) {
   swapHelper(list, i, j);
}

//와일드카드 타입인 List<?>에는 null 외에 다른 값을 넣을 수 없음 -> 와일드카드 타입의 실제 타입을 알려주는 도우미 메서드를 추가하여 사용 
public static <E> void swapHelper(List<E> list, int i, int j) {
   list.set(i, list.set(j, list.get(i)));
}
```
1. 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체할 것
    - but, 코드 구현 시 null을 제외한 값을 넣을 수 없으므로 제네릭 메서드를 추가하여 사용할 것

<br><br>

# 아이템 32: 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 가변인수: 메서드에 넘기는 인수의 개수를 조절할 수 있게 해줌
- 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 만들어짐 
  - 가변인수 매개변수에 제네릭, 매개변수화 타입이 포함되면 컴파일 경고 발생 -> 타입 안정성 깨짐

### 메서드가 타입 안전한지 확인하는 방법
- 가변인수 메서드가 호출될 때 생성되는 제네릭 배열에 1)아무것도 저장되지 않고, 2)배열의 참조가 밖으로 노출되지 않는다면 타입 안전함 
  - 가변인수 매개변수 배열이 메서드로 인수들을 전달하는 일만 하면 메서드는 안전함
- 제네릭 가변인수 매개변수 배열에 다른 메서드가 접근할 수 있게 허용하는 것은 타입 안전하지 않으나  1)@SafeVarargs로 어노테이트된 또 다른 가변인수 메서드에 넘기는 것, 2)배열 내용의 일부 함수를 호출만 하는(가변인수를 받지 않는) 일반 메서드에 넘기는 것은 안전 


### 타입 안전성 보장 방법 1. @SafeVarargs 어노테이션
- 제네릭 가변인수 메서드를 호출할 때 발생하는 컴파일 경고를 숨김
- 메서드 작성자가 메서드의 타입 안정성을 보장하는 장치
- 제네릭, 매개변수화 타입의 가변인수 매개변수를 받는 모든 메서드에 달 것
- 재정의할 수 없는 메서드에만 달아야 함

### 타입 안전성 보장 방법 2.List 매개변수로 대체하기
- (배열로 생성되는) 가변인수 매개변수를 List 매개변수로 바꾸는 방식은 컴파일러가 메서드의 타임 안전성을 검증할 수 있게 함
- List.of와 같은 안전한 메서드를 사용할 수 있게 함

<br><br>

# 아이템 33: 타입 안전 이종 컨테이너를 고려하라 (Q)
- 제네릭은 `Set<E>`, `Map<K, V>` 등의 컬렉션과 `ThreadLocal<T>`, `AtomicReference<T>` 등의 단일원소 컨테이너(클래스?)에도 쓰임 
  - 이 때, 매개변수화되는 대상은 원소가 아닌 컨테이너 자신이기 때문에 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한됨

### 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)
- 타입의 수를 제한하지 않고 유연성을 높이기 위해 사용
- 컨테이너 대신 키를 매개변수화한 후, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 패턴

### 타입 토큰(type token)
- 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴(Class 객체)
- class 리터럴의 타입은 `Class<T>` ex. String.class의 타입: `class<String>`

```java
// 타입 안전 이종 컨테이너 패턴 - 구현
public class Favorites {
    // 와일드카드 타입이 중첩되었음(map이 아닌 key가 와일드카드 타입): 다양한 타입의 class 리터럴을 넣을 수 있음
    //map의 타입은 Object: 모든 값이 키로 명시한 타입임을 보증하지 않음 -> getFavorite 메서드에서 되살릴 수 있음
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
      favorites.put(Objects.requireNonNull(type), instance);
    }
   
    public <T> T getFavorite(Class<T> type){
      return type.cast(favorites.get(type)); // Class의 cast 메서드: 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환함
    }
}
```
### 위와 같은 클래스의 제약사항 
1. Class 객체를 제네릭이 아닌 로타입으로 넘기면 타입 안전성이 깨짐 (Q)
2. 실체화 불가 타입에는 사용할 수 없음 
    - String, String[] 클래스 객체는 저장할 수 있어도 `List<String>` 클래스는 저장 불가능
      - 슈퍼 타입 토큰으로 우회하는 방법? 
```java
// 슈퍼 타입 토큰 적용: 제네릭 타입도 저장 가능
List<String> pets = Arrays.asList("개", "고양이");
f.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = f.getFavorite(new TypeRef<List<String>>(){});
```
- cf. 한정적 타입 토큰 (Q)
