Item8 : finalizer와 cleaner 사용을 피하라

자바의 두 가지 객체 소멸자 제공 
finalizer : 예측할 수 없고, 상황에 따라 위험 할 수 있어 일반적으로 불필요하다.
cleaner : finalizer 보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불 필요하다.

자바의 자원회수는 try-with-resources와 try-finally를 사용해 해결한다(Item9)

finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.
	EX) 파일닫기를 두 객체 소멸자에게 맡기면 두 객체 소멸자가 실행을 게을리 해서 파일을 계속 열어두게 되어 프로젝트 실패 가능성이 있음.
finalizer나 cleaner를 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터(GC) 알고리즘에 달렸으며, 이는 구현마다 천차만별이다.
	EX) 테스트한 JVM은 완벽히 수행했으나 고객 시스템에서는 재앙을 불러올수도 있다.
 
자바 언어 명세는 finalizer나 cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다.
따라서 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.

System.gc 혹은 System.runFianlization 메소드 또한 
finalizer와 cleaner가 실행될 가능성을 높여줄 수는 있으나 보장해주진 않는다.
(보장해주겠다는 System.runFinalizersOnExit와 Runtime.runFianlizersOnExit 지만 둘다 심각한 결함존재)

-finalizer 동작중 발생한 예외는 무시되며 처리할 작업있어도 강제 종료된다.

finalizer와 cleaner는 심각한 성능 문제도 동반함.
finalizer를 사용하면 GC의 효율이 떨어져 더 느려짐.

finalizer의 보안문제 
	finalizer 공격 : 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성 되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행되고,
		이 finalizer는 GC 수집 못하게 막으며 이 일그러진 객체의 메소드를 호출해 허용되지 않을 작업이 수행하게 된다.
	객체 생성을 막으려면 생성자에서 예외를 던지면 되지만 finalizer가 있으면 그렇지도 않다.
		final 클래스들은 그 누구도 하위 클래스를 만들수 없으니, final이 아닌 클래스를 finalizer 공격으로부터 방어하려면
		아무 일도 하지 않는 finalize 메소드를 만들고 final로 선언한다.
		
So, 파일이나 스레드등 종료해야 할 자원을 담고 있는 객체의 클래스에서의 대체제는?
=> AutoCloseable을 구현하고 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출한다.

그러면 finalizer의 사용처는? 
=> 
1. 안정망 역할, 보장은 없지만 아예안하는것보다 낫다. 그러나 과연 그럴 가치가 있는지 판단해야됌.
	사용예시 : FileInputStream, FileOutPutStream, ThreadPoolExecutor
	
2. 네이티브 피어(nativepeer)와 연결된 객체에서 사용.
	먼저 네이티브(native)란 자바 외의 C나 C++ 등 다른 언어로 작성된 프로그램을 나타낸다. 
	네이티브 피어란? 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 뜻함.
	네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못하여 자바 피어 회수할때 네이티브 객체까지 회수하지 못한다.
	단, 성능 저하 감당할수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을때만 해당.
	=> 그렇지 않다면 close 메소드를 사용.
	
결론 : cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않는 네이티브 자원 회수용으로만 사용하자. 물론 이런경우라도 불확실성과 성능 저하에 주의해야 한다.

참고:
https://camel-context.tistory.com/43
