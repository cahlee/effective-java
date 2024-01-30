# Item5 : 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 

### 목적 : 사용하는 자원에 따라 동작이 달라지는 클래스에는 인스턴스를 생성할때 생성자에 필요한 자원을 넘겨주는 의존객체 방식을 택해라

예시로 맞춤법 검사기 프로그램을 짜려고 한다.  
해당 프로그램은 dictionary에 의존하는데,   
이런 클래스를 보통 정적 유틸리티 클래스(item4), 혹은 싱글턴(item3)으로 구성한다.  
그러나 테스트 하기 힘듬의 단점, 언어별, 특수어휘용 dictionary가 필요하면 자원에 따라 달라지는 동작을 구현해야된다.  

#### => 따라서 의존 객체 주입 패턴을 사용한다.

```java
public class SpellCheck{
	private final Lexicon dictionary;
	
	// 의존 객체 주입
	public SpellChecker(Lexicon dictionary){
		this.dictionary = Objects.requireNonNull(dictionary);
	}

	public boolean isValid(String word) {...}
	public List<String> suggestions(String typo) {...}
}
```

### 결론 : 의존객체 주입은 불변(item17)을 보장하고 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.  

변형으로는 생성자에 자원 팩토리를 넘겨주는 방식인 팩토리 메소드 패턴이 있다.  
자바8에서 나온 Supplier<T> 이터페이스가 팩토리를 표현하였음.  

<br>
<br>

---------------
#### 배경지식

* 의존성 주입(Dependency Injection)  
Spring의 DI 컨테이너, 애플리케이션 실행 시점에 필요한 객체(빈)를 생성해야 하며, 의존성이 있는 두 객체를 연결하기 위해 한 객체를 다른 객체로 주입시켜야 한다. 
그리고 이러한 개념은 제어의 역전(Inversion of Control, IoC)라고 불리기도 한다.  
어떠한 객체를 사용할지에 대한 책임은 프레임워크에게 넘어갔고, 자신은 수동적으로 주입받는 객체를 사용하기 때문이다.  

출처: https://mangkyu.tistory.com/150 [MangKyu's Diary:티스토리]
