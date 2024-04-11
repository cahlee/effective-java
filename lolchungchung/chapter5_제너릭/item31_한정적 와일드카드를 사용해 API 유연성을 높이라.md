# Item31 : 한정적 와일드카드를 사용해 API 유연성을 높이라

## 목차
- ### 한정적 와일드 카드 사용할때는 언제인가 - PECS
- ### 타입 매개변수와 와일드 카드, 어떤것을 써야하는가?
<br>

### 한정적 와일드 카드 사용할때는 언제인가 - PECS

매개변수화 타입은 불공변(invariant)이다. 

[불공변(Invariant)이란 SubType이 SuperType의 하위 타입일 때, SubType은 SuperType이 될 수 없고, SuperType도 SubType이 될 수 없다는 것을 말한다.]  

(ex)List<String>은 List<Object>의 하위 타입이 아니다.   
List<String>은 List<Object>가 하는일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.   

하지만 때론 불공변 방식보다 유용한게 필요할 때가 있다.

#### 예시코드 - stack
```java
public class Stack<E>{
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
```

여기서 원소를 스택에 넣는 메소드를 추가한다고 하자.

#### 코드31-1) 와일드 카드 타입을 사용하지 않는 pushAll 메소드 - 결함있음
```java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```

예시로 아래의 코드를 동작한다고 했을때, 문제가 발생한다.
아래코드에서는 Iterable src의 원소타입이 스택의 원소 타입과 일치하면 잘 동작한다.   
그러나 아래와 같이 구현할경우, Integer는 Number의 하위타입이니 논리적으로 잘 동작해야 할 것 같지만 매개변수화 타입이 불공변이기 때문에 오류가 발생한다.

```java
    StackWithGeneric<Number> stack = new StackWithGeneric<>();
    List<Integer> integers = List.of(1, 2, 3);
    stack.pushAll(integers);

/* ERROR */
  StackTest.java7: error: incompatible types: Iterable<Integer>
  cannot be converted to Iterable<Number>
  
``` 

이러한 해결책으로 자바는 한정적 와일드 카드 타입이라는 특별한 매개변수화 타입을 지원한다.   
pushAll의 입력 매개변수 타입 'E의 Iterable'이 아니라 'E의 하위 타입의 Iterable'이어야 하며    
와일드 카드 타입 Iterable<? extends E>가 정확히 이런 뜻이다.   

#### 코드31-2) E 생산자(producer) 매개변수에 와일드카드 타입 적용
```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

이번엔 모든 원소를 주어진 컬렉션으로 옮기는 popAll을 구현하자

#### 코드31-3) 와일드 카드 타입을 사용하지 않은 popAll 메소드 - 결합 존재
```java
public void popAll(Collection<E> dst) {
    while(!isEmpty() {
        dst.add(pop());
    }
}
```

위의 코드도 주어진 컬렉션의 원소타입이 스택의 원소 타입과 일치한다면 말금히 컴파일 되고 문제없이 동작되지만 아래 코드는 오류가 발생한다.
```java
  Stack<Number> numberStack = new  Stack<>();
  Collection<Object> objects = ...;
  numberStack.popAll(objects);

/* ERROR */
```

이번에는 popAll의 입력 매개변수의 타입이 'E의 Collection' 이 아니라 'E의 상위 타입의 Collection'이어야 한다.  

#### 코드31-4) E 소비자(consumer) 매개변수에 와일드 카드 타입 적용
```java
public void popAll(Collection<? super E> dst) {
    while(!isEmpty() {
        dst.add(pop());
    }
}
```

#### 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드 카입 타입을 사용하라.
#### 팩스(PECS) : producer-extends, consumer-super   
#### 즉, 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용해라.

##### 예시 1) 아이템 28의 Chooser 생성자, choices 컬렉션은 T 타입의 값을 생산만 하니 T를 확장하는 와일드 카드 타입 사용 
```java
public Chooser(Collection<T> choices)    
public Chooser(Collection<? extends T> choices)
```

#### 예시 2) 코드 30-2의 union 메소드, s1 E와 s2 E 모두 생산자
```java
public static <E> set<E> union(Set<E> s1, Set<E> s2)    
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

( 반환타입에는 한정적 와일드 카드 타입을 사용하지 말것, 유연하긴 커녕 오히려 클라이언트 코드에서도 와일드 카드를 기술해야됌)

#### 예시 3) 코드 30-7의 max 메소드 
```java
public static <E extends Comparable<E>> E max(List<E> list)   
public static <E extends Comparable<? super E>> E max(List<? extends E> list)   
```

위는 PECS를 두번 적용하였음.   
1. 입력 매개변수에서는 E 인스턴스를 생산하므로 extends 추가
2. comparable<E>는 E 인스턴스를 소비한다.
##### comparable은 언제나 소비자 이므로 일반적으로 comparable<E>보다는 comparable<? super E>를 사용하는게 낫다.(comparator 포함)


### 타입 매개변수와 와일드 카드, 어떤것을 써야하는가?

## 정리   
#### 클라이언트에서 입력 매개변수 및 반환값을 명시적으로 형변환하는 메서드보다 제네릭 메서드가 더 안전하고 사용하기도 쉽다.   
#### 형 변환은 런타임 시에 에러를 동반하기 쉬우므로 제네릭을 사용하자.   
