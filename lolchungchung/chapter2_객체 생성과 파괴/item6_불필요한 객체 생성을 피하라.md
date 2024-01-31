# Item6 : 불필요한 객체 생성을 피하라

### 목적 : 구현시 성능에 대하여

생성자는 호출할때마다 새로운 객체를 만들지만, 팩토리 매소드는 그렇지 않다.  
ex) Boolean(String) 생성자 대신 Boolean.valueOf(String)을 사용하는것이 더 좋다.  
불변객체 뿐만 아니라 가변객체도 사용중에 변하지 않을 것임을 안다면 재사용할 수 있다.  

생성비용이 비싼 객체들은 캐싱하여 재사용한다.  
(질문 : 생성비용이 비싸다..? : BIG O 알고리즘 같은거 )  
생성비용이 비싼 객체들 재사용 관련 예시 => 정규표현식 패턴  
```java
static boolean isRomanNumeral(String s){
	return s.matches("^(?=,.)m*(C[MD]|D?C{0,3})" 
		+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

위의 사례에서는 String.matches는 한번쓰고 버려져서 GC 대상이 되지만 인스턴스 생성 비용이 높다.  
(성능이 중요한 상황에서 반복 사용이 적합하지 않음)  

성능 개선을 위해 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고  
나중에 isRomanNumeral 메소드가 호출될 때마다 이 인스턴를 재사용한다.  

```java
public class RomanNumerals{
	static boolean isRomanNumeral(String s){
		return s.matches("^(?=,.)m*(C[MD]|D?C{0,3})" 
			+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
	}
	
	static boolean isRomanNumeral(String s){
		return ROMAN.matcher(s).matchers();
	}
}
```

객체가 불변이라면 재사용해도 안전하다. ( 반대 경우는 어댑터 )  

불필요한 객체 만들어내는 예시 : 오토박싱(auto Boxing)    
> (오토박싱 : 기본타입과 박싱된 기본 타입을 섞어 쓸때 자동으로 상호 변호해주는 기술.)

예시코드
```java
private static long sum(){
	Long sum = 0L;
	for(long i = 0 ; i <= Integer.MAX_VALUE; i++)
		sum += i;
	
	return sum;	
}
```
위의 사례는 long 타입인 i 가 Long 타입인 sum에 더해질때마다 객체가 생성되고 느려진다!  
박싱된 기본타입 보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의  

<br>
<br>

---------------
#### 배경지식
* 객체 풀?  
의도 : 객체를 매번 할당/해제 하지 않고 고정 크기 풀에 들어있는 객체를 재사용함으로써 메모리 사용 성능을 개선한다.

	객체를 필요로 할때 풀에 요청을 하고, 반환하고 일련의 작업을 수행하는 패턴.
	많은 수의 인스턴스를 생성할때 혹은 무거운 오브젝트를 매번 인스턴스화 할때 성능 향상을 가져온다.    
	예를들어, 데이터베이스에 접속하는 여러 객체를 만들때 매번 새로 생성하는 것보단 미리 생성된 풀에서 객체를 반환받아오는 것이 더 이득.    
	이런 문제로 JDBC 에서는 JDBC Connection Pool 을 제공하고 있으며 Thread Pool 역시 오브젝트 풀이 기본 원리이다.  

	출처: 
	https://creatordev.tistory.com/73 [Creator Developer:티스토리]  
	https://luv-n-interest.tistory.com/1116  

* 어댑터?  
	한 클래스의 인터페이스를 클라이언트에서 사용하고자하는 다른 인터페이스로 변환한다.  
	어댑터를 이용하면 인터페이스를 호환성 문제 때문에 같이 쓸 수 없는 클래스들을 연결해서 쓸수있다.  

	참조 https://jusungpark.tistory.com/22
