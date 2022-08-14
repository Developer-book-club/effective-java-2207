# 3장 모든 객체의 공통 메서드

* Object 에서 final 이 아닌 메서드들은 재정의할 때 지켜야 할 규약이 있다.
	* equals, hashCode, toString, clone, finalize
	* Comparable.compareTo


## 아이템 10 equals는 일반 규약을 지켜 재정의하라

* 재정의하지 않아야 하는 경우
	* 각 인스턴스가 고유한 경우
	* 논리적 동치를 검사할 필요가 없는 경우
	* 상위 클래스의 equals로 충분한 경우
	* 클래스가 private, package-private 이며 equals 를 호출할 일이 없는 경우
* 재정의해야 하는 경우
	* 논리적 동치를 검사해야 하는데 상위 클래스에서 하지 않는 경우 (주로 값 클래스)
* equals 메서드의 규약
	* 반사성(reflexivity): x.equals(x) => true
	* 대칭성(symmetry): x.equals(y) => true, y.equals(x) => true
	* 추이성(transitivity): x.equals(y) => true, y.equals(z) => true, z.equals(x) => true
	* 일관성(consistency): x.equals(y) => 항상 true 또는 false
	* not-null: x.equals(null) => false
* equals 메서드 규약 지키기
	* 반사성을 준수하기는 쉽다.
	* 상속을 이용하는 경우 대칭성과 추이성을 깨트리기 쉽다.
	* 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
	* 상속 대신 합성을 사용하라.
	* 일관성을 준수하려면 신뢰할 수 없는 자원을 equals 에서 참조하면 안 된다.
* equals 메서드 구현하기
	* 자기 참조 확인: `==`
	* 타입 확인: ``isinstanceof``
	* 인자 형 변환
	* 모든 핵심 필드 검사
	* 대칭성, 추이성, 일관성을 단위 테스트로 검사
	* hashCode 재정의
	* 단순하게 접근할 것
* AutoValue 프레임워크 활용
* 핵심 정리: 꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.


## 아이템 11 equals를 재정의하려거든 hashCode도 재정의하라
* equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.
* 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
* 규약
	* equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션 실행동안 hashCode 메서드는 항상 같은 값을 반환해야 한다.
	* equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
	* equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 값을 반환하는 것이 해시테이블 성능에 유리하다.
* hashCode 를 상수함수로 정의하면 규약은 지키지만, 성능이 매우 나빠져 해시테이블이 O(N) 으로 동작하게 된다.
* hashCode 정의하는 법
	* 각 핵심 필드의 hashCode를 재귀적으로 계산하고 초기 해시값 c 를 더한다.
	  `result' = 31 * result + c`
	* 파생 필드는 해시코드 계산에서 제외해도 된다.
	* equals 비교에 사용되지 않는 필드는 반드시 제외한다.
	* 31을 곱하는 이유
		* 홀수: 짝수이면 오버플로 발생 시 정보 유실
		* 소수: 전통
		* 최적화: 시프트 연산과 뺄셈으로 최적화
	* 불변 클래스인 경우 해시값을 캐시하는 것을 고려한다.
	* 해시 성능을 위해 핵심 필드를 제외하지 말자.
	* 해시 규칙을 공개하지 마라. 구현을 바꿀 여지를 둘 수 있다.
* 핵심 정리: equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다. 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다. 이렇게 구현하기가 어렵지는 않지만 조금 따분한 일이긴 하다(68쪽의 요령을 참고하자). 하지만 걱정마시라. 아이템 10에서 이야기한 AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다. IDE들도 이런 기능을 일부 제공한다.


## 아이템 12 toString을 항상 재정의하라
* Object.toString 은 별로 도움이 되지 않는다.
* 규약
	* 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 출력하라
	* 모든 하위 클래스에서 이 메서드를 재정의하라
* toString 메서드는 여러 곳에서 자동으로 호출된다.
	* println, printf
	* 문자열 연결 연산자(+)
	* assert
	* 디버거의 객체 출력
* 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.
	* 예외: 객체가 거대하거나, 문자열로 표현하기에 적합하지 않은 경우
* toString 반환값의 형식을 문서화할 것인지 정하기
	* 장점과 단점(명시적, 묵시적 의존성 발생) 고려할 것
* toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 한다.
	* 안 그러면 toString을 파스해야 한다.
* 핵심 정리: 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.


## 아이템 13 clone 재정의는 주의해서 진행하라
* Cloneable 믹스인 인터페이스를 구현하는 것만으로 clone을 호출할 수 없다.
	* clone 메서드가 Object 에 있으며 protected 이기 때문.
* Cloneable 인터페이스가 Object.clone 의 동작을 결정한다.
	* Cloneable 을 구현하면 Object.clone 이 객체의 필드들을 복사한 객체를 생성한다.
	* 상위 클래스의 protected 메서드의 동작을 변경: 이상한 패턴이니 따라하지 말 것!
* 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.
	* 이 기대를 만족시키면 생성자를 호출하지않고도 객체를 생성할 수 있게 된다.
* 상위 클래스의 clone 을 상속해 Cloneable 구현하기 (불변 클래스인 경우)
	* super.clone 호출
	* Object 인스턴스를 하위 클래스로 형 변환하여 반환
	* 공변 반환 타입: 상위 클래스의 메서드를 하위 클래스에서 오버라이드 할 때, 반환 타입을 좁히는 것
* 상위 클래스의 clone 을 상속해 Cloneable 구현하기 (가변 클래스인 경우)
	* 문제가 복잡해진다.
	* 동일한 컬렉션의 레퍼런스를 참조하는 문제가 생긴다.
	* 각 필드에 대하여 clone을 재귀적으로 호출하거나, 깊은 복사를 해야 한다.
* 생성자, 그리고 clone 에서는 재정의될 수 있는 메서드를 호출하지 않아야 한다.
	* 하위 클래스를 복제할 때 문제가 생길 수 있다.
* 상속용 클래스는 Cloneable을 구현해서는 안 된다.
* Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야 한다.
* 대안: 생성자와 복사 팩터리
* 핵심 정리: Cloneable이 몰고 온 모든 문제를 되짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다(아이템 67). 기본 원칙은 ‘복제 기능은 생성자와 팩터리를 이용하는 게 최고’라는 것이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.


## 아이템 14 Comparable을 구현할지 고려하라
* Comparable.compareTo는 Object.equals 와 비슷하지만, 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.
* Comparable 을 구현하면
	* 자연적인 순서(natural order)가 있음을 보장한다.
	* 정렬할 수 있다. `Arrays.sort(a);`
	* 기타 순서, 검색, 정렬과 관련된 여러 기능을 활용할 수 있다.
* Comparable.compareTo의 규약
	* 이 객체와 주어진 객체의 순서를 비교한다.
	* 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.
	* 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
	* 대칭성(?): `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`
	* 추이성: `(x.compareTo(y) > 0 && y.compareTo(z) > 0)` =>  `x.compareTo(z) > 0`
	* 대칭성(?): `x.compareTo(y) == 0` => `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`
	* equals와의 관계(권장사항): `(x.compareTo(y) == 0) == (x.equals(y))`
* 비교 대상과의 타입이 다르면 보통 ClassCastException을 던지면 된다.
* 핵심 정리: 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.

