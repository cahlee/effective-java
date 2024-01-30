# Item2 : 생성자에 매개변수가 많다면 빌더를 고려하라

* 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려움.
  
* 자바빈즈 패턴에서는 객체 하나를 만들려면 메소드를 여러개 호출해야되고 개게가 완전히 생성되지 전까지는 일관성(consistency)가 무너진 상태이다.  
=> 일관성이 무너지므로 클래스를 불변으로 만들 수 없다. 이는 Open-Closed Principle에 위배됌
	
### 결론 => 생성자나 정적 팩토리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다.  
(점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈 보다 훨씬 안전하다.)

<br>
<br>

---------------
#### 배경지식  
개방 폐쇄 원칙(OCP)?  
=> 객체 지향 설계 5원칙 중 하나(S.O.L.I.D)
기존의 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계되어야 한다는 원칙.
확장에 대해서는 개방적(open)이고, 수정에 대해서는 폐쇄적(closed)이어야 한다.

* 객체 지향 설계 5원칙(S.O.L.I.D)  
	1. SRP(Single Responsibility Principle): 단일 책임 원칙  
	2. OCP(Open Closed Priciple): 개방 폐쇄 원칙  
	3. LSP(Listov Substitution Priciple): 리스코프 치환 원칙  
	4. ISP(Interface Segregation Principle): 인터페이스 분리 원칙  
	5. DIP(Dependency Inversion Principle): 의존 역전 원칙  
