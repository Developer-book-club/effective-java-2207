예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적 (stack trace) 정보를 자동으로 출력한다. 

스택 추적은 예외 객체의 toString 메서드를 호출해 얻는 문자열로, 보통은 예외의 클래스 이름 뒤에 상세 메시지가 붙는 형태이다.

이러한 정보는 아주 중요하. 실패 순간의 상황을 포착하는 것이기 때문이다.

**실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.**

**또한 최종 사용자에게는 친절한 안내 메시지를 보여줘야하지만, 예외 메세지는 가독성보단 담긴 내용이 훨씬 중요하다.**