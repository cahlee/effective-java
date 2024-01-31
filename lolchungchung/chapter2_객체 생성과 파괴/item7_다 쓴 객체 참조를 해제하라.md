# Item7 : 다 쓴 객체 참조를 해제하라  
### 목적 : 자바의 메모리 누수에 대해 알아보자

아래 예시 코드 스택에서 메모리 누수 문제점 찾아보자
```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	
	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	
	public void push(Object e){
		ensureCapacity();
		elemets[size++] = e;
	}
	
	public Object pop() {
		if(size = 0) throw new EmptyStackException();
		return elements[--size];
	}
	
	public void ensureCpacity(){
		if(elements.length == size) elements = Arrays.copyOf(elements, 2 * size + 1);
	}
}
```

##### pop에서 문제 => 다 쓴 참조(obsolete reference)들이 GC에 의해 수거되지 않았다.  
스택에서 꺼내진 객체들은 가비지 컬렉터가 회수하지 않는다.  
가비지 컬렉터는 A 객체를 보유하면 A 객체를 통해 참조되는것들도 수거 처리에서 제외된다(여기선 A 객체 = 스택)  

스택은 자기 메모리를 직접 관리하기 때문에 가비지 컬랙터가 알 수 없다.  
자기 메모리를 직접 관리하는 클래스는 메모리 누수에 주의해야한다.  

=> 해법 : 해당 참조를 다 쓰면 null 처리 한다.

제대로 구현한 pop
```java
public Object pop(){
	if(size == 0) throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null ; // 다 쓴 참조 해제
	return result;
}
```

null 처리를 따로하여 가비지 컬랙터에서 해당 객체를 쓰지 않을것을 알린다.  
##### 그러나 다 쓴 객체를 null 처리하는것이 추후 해당 객체 참조시 NullPointException으로 방지가 되서 좋긴하지만 객체참조를 null처리하는 일은 예외적인 경우여야 한다.  
#### => 가장 좋은 방법은 그 참조를 담은 변수를 유효범위(scope) 밖으로 밀어내는것.[뭔말?]

* 메모리 누수 케이스
	1. 캐시  
		캐시 메모리 관리 해법  
			1. 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이면 WeakHashMap을 사용해 캐시를 만들자 -> 다 쓴 엔트리는 그 즉시 자동 제거.  
			2. 보통 캐시 엔트리의 유효기간을 정확히 정의하기 어려워 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식으로 구현, 이런 방식에서는 쓰지 않는 엔트리 청소  
				백그라운드 스레드 활용 혹은 캐시에 새 엔트리 추가시 부수 작업 수행(LinkedHashMap은 removeEldestEntrty 메소드 사용)  
				
			
	2. 리스너(listener) 혹은 콜백(callback)  
		1. 콜백 등록만 하고 명확히 해지 하지 않으면 계속 쌓이기에 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬랙터 즉시 수거  
			ex) WeakHashMap에 키로 저장  
		
<br>
<br>

---------------
#### 배경지식
	
* 강한 참조?  
	new 할당 후 새로운 객체를 만들어 해당 객체를 참조하는 방식, 참조가 해제되지 않는 이상 GC의 대상이 되지 않음  

* 약한 참조?  
	weakReference 클래스를 사용하여 생성. GC가 발생하면 무조건 수거된다.  
	weakReference가 사라지는 시점이 GC의 실행 주기와 일치함.  
	
	WeakHashMap은 weakReference의 특성을 이용하여 HashMap의 element를 자동으로 GC 해버린다.  
	key에 해당하는 객체가 더 이상 사용되지 않는다고 판단되면 제거한다.  

* 참조?  
	참고:  
	https://gyubgyub.tistory.com/83  
	https://opentutorials.org/module/2495/14152  

* 순환참조?  
	둘 이상의 클래스나 빈(Bean)이 서로를 참조하는 상황을 의미  
	순환 참조가 발생하면 객체 생성 시점에서 무한루프에 빠져 프로그램이 정상적으로 동작불가.  
	의존성 주입 방법을 적절히 선택하여 해결  

	참고:  
	https://bepoz-study-diary.tistory.com/340  
