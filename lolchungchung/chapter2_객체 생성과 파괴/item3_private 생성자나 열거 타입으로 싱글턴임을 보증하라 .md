# Item3 : private 생성자나 열거 타입으로 싱글턴임을 보증하라  

### 목적 : 싱글턴을 만드는 방법에 대해 알아보자   

* 싱글톤을 만드는 방식 : 생성자는 private로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 맴버를 하나 작성

#### 1. public static 맴버가 final 필드인 방식
```java
public class Elvis{
	
	// priavte 생성자는 이 부분에서 딱 한번만 호출되어 elvis 클래스가 초기화 될 때 만들어진 인스턴스가 전체 시스템에서 하나됨을 보장.
	public static final Elvis INSTANCE = new Elvis(); 
	
	private Elvis() {
		// 생성자는 외부에서 호출못하게 private 으로 지정해야 한다.
	}
	
	public void leaveTheBuilding(){ ... }
}
``` 
(예외인 경우도 권한있는 클라이언트가 리플렉션 API로 생성자를 호출할 수 있으나, 이 공격을 방어하려면 생성자를 수정하여 두번째 
객체가 생성되려 할 때 에외를 던지면 된다 - 추후 아이템 65)

#### 2. 정적 팩터리 메소드를 public static 맴버로 제공.
```java
public class Elvis{

	public static final Elvis INSTANCE = new Elvis(); 
	
	private Elvis() {}
	public static Elvis getInstance() {return INSTANCE; } <= 추가 
	
	public void leaveTheBuilding(){ ... }

}
```

* 1번의 장점은 싱글턴임이 API에서 명백함.  
* 2번의 장점은 수정이 가능하다  
	1. 싱글턴 아니게 변경 가능 - 유일한 인스턴스를 반환하던 팩토리 메소드를 호출하는 스레드별로 다른거 호출 가능  
	2. 정적 팩토리를 제너릭 싱글턴 팩토리로 만드는것 가능 - 아이템30 [무슨말..?]  
	3. 정적 팩터리의 메소드 참조를 공급자(supplier)로 사용할 수 있다. - 아이템 43,44 [무슨말...?]  
<br>

#### 싱글턴의 직렬화 할때 주의! 싱글턴 클래스를 직렬화 하려면 Serializable 구현 선언 뿐만 아니라,
1. 인스턴스 필드를 일시적(trasient) 선언  
2. readResolve 메소드를 제공해야된다(아이템89)  
#### 이렇게 안하면 역직렬화시 새로운 인스턴스가 만들어져서 싱글톤임을 보장하지 않음.  

readResolve 메소드 구현
```java
private Object readResoleve(){
	// 진짜 Elvis 반환, 가짜는 가비지 컬랙터가..
	return INSTANCE;
}
```

### 3. 열거 타입으로 선언
```java
public enum Elvis {
	INSTANCE;
	
	public void leaveTheBuilding(){...}
}
``` 
	
더 간결하고, 추가 노력없이 직렬화 가능하며 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스 생성 막아줌

책에서 추천하길 대부분 상황에서 원소가 하나뿐인 열거 타입 싱글턴을 만드는 가장 좋은 방법.
(싱글턴이 Enum외에 클래스를 상속해야 한다면 이 방법 사용불가)

<br>
<br>

---------------
#### 배경지식
* 싱글턴 : 인스턴스를 오직 하나만 생성할수 있는 클래스를 말한다.  
쓰는 이유? 이점 ?  

장점 :  
1. 메모리, new 연산자를 통한 고정된 메모리 영역 사용, 이미 생성된 인스턴스를 활용하니 속도측면에서 뛰어나다.

2. 클래스간 데이터 공유가 쉽다 -> 전역으로 사용되는 인스턴스이니까 접근이 쉽다.
그러나 동시성문제가 발생할수 있으니 주의,

단점 : 
1. 코드 자체가 많이 필요, 멀티쓰레드 환경에서 발생할수 있는 동시성 문제 해결을 위해 syncronized
키워드 사용

2. 테스트 하기 어렵다. 자원을 공유하고 있기 때문에 테스트를 위해선 매번 인스턴스를 초기화 해야된다.
(MOCK 대체 불가)

3. 의존 관계상 클라이언트가 구체 클래스에 의존하게 된다.
new 키워드를 직접사용하여 클래스안에서 객체를 생성하니 SOLID원칙 중 DIP를 위반하고
OCP 원칙또한 위반할 가능성이 높다.

요약 :
싱글톤 패턴은 안티패턴으로 불릴 만큼 단독으로 사용한다면 객체 지향에 위반되는 사례가 많다. 
그러나 스프링 컨테이너 같은 프레임워크의 도움을 받으면 싱글톤 패턴의 문제점들을 보완하면서 
장점의 혜택을 누릴 수 있다. 실제로 스프링 빈은 컨테이너의 도움을 받아 싱글톤 스콥으로 관리되고 있다.

참고 
싱글톤 패턴이란?  
https://tecoble.techcourse.co.kr/post/2020-11-07-singleton/

스프링에서의 싱글톤 사용  
https://velog.io/@minwest/Spring-%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%80-%EB%B9%88%EC%9D%84-%EC%99%9C-%EC%8B%B1%EA%B8%80%ED%86%A4%EC%9C%BC%EB%A1%9C-%EC%83%9D%EC%84%B1%ED%95%A0%EA%B9%8C

* 동시성 문제란 ?  

* 직렬화(Serializable)?  
직렬화란 객체를 바이트 스트림으로 바꾸는 것, 즉 객체에 저장된 데이터를 스트림에 쓰기(write) 위해 연속적인(serial) 데이터로 변환하는 것이다.

ex)
통화 상대방에게 어떤 사진이나 바디 랭귀지 없이 전화 상으로, 새로 입양한 강아지 Rex를 있는 그대로 설명하고 싶지만 말로만 하기엔 어려운 일이다.
그러나 Rex의 모든 특성을 가진 객체를 생성해서 직렬화한 후 통화 상대방에게 보내면 일이 보다 간단해진다(비유적으로…).

참고 https://medium.com/@lunay0ung/basics-%EC%A7%81%EB%A0%AC%ED%99%94-serialization-%EB%9E%80-feat-java-2f3eb11e9a8

* Enum?  
컴퓨터 프로그래밍에서 Enumerated Type(열거형 타입)을 줄여서 보통 Enum이라고 쓴다. 
요소, 멤버라 불리는 명명된 값의 집합을 이루는 자료형이다. 열거자 이름들은 일반적으로 해당 언어의 상수 역할을 하는 식별자이다.

사용할때 : 보통 도메인을 설계할 때 사용하는 인스턴스의 수가 정해져 있고 관련된어 처리할 수 있는 상수값이 여러개 존재할 때 Enum을 사용한다.

예를 들어 로또 프로그램에서 1등부터 5등까지를 의미하는 등수와 각 등수가 만들어 지는 조건, 
그리고 각 등수에 해당하는 상금을 묶어낼 필요가 있을 경우에 Enum을 사용하면 외부에서도 Enum의 요소를 조회하여 각 상태에 용이하게 접근할 수 있다.
``` java
enum LottoRank {
	FIRST(6, false, 2_000_000_000),
	SECOND(5, true, 30_000_000),
	THIRD(5, false, 1_500_000),
	FOURTH(4, false, 50_000),
	FIFTH(3, false, 5_000),
	NONE(0, false, 0);
}
```
이외에 예시로 Taxi, Bus, Subway 같이 교통수단 요금책정 등등이 있다.

참고 https://eatnows.tistory.com/91
