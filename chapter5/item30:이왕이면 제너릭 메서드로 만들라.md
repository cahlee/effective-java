# Item30 : 이왕이면 제너릭 메서드로 만들라

## 목차
- ### 제너릭 메서드
- ### 재너릭 싱글턴 팩터리
- ### 재귀적 한정적 타입   
<br>

### 재너릭 메서드

매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.   
Collections의 binarySearch, sort 등 알고리즘 메서드는 모두 제네릭이다.   
아래 코드를 예시로 제너릭 메서드를 만들어 보자   

#### 코드30-1) 로 타입 사용 - 수용불가(아이템26)
```java
public static Set union (Set s1, Set s2){
    Set result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

위의 사례에서는 로 타입을 썼기 때문에 타입안정성을 잃어 컴파일은 되지만 경고 두개가 발생한다.

```java
Union.java:5 warning : [unchecked] unchecked call to   
HashSet(Collection<? extends E>) as a member of raw type HashSet   
  Set result = new HashSet(s1);

Union.java:5 warning : [unchecked] unchecked call to   
addAll(Collection<? extends E>) as a member of raw type HashSet   
  result.addAll(s2)
```

따라서 경고를 없애려면 이 메서드를 타입 안전하게 만들어야한다.   
메서드 선언에서의 세집합(입력2개, 반환1개)의 원소 타입을 타입 매개변수로 명시하고 리턴값도 이 매개변수만 사용하게 수정하면된다.   
위 코드는 세개의 Set 집합이 타입이 모두 같아야 한다.    
이를 한정적 와일드 카드 타입을 이용하면 더 유연하게 개선이 가능하다.(아이템31)



#### 코드30-2) 제너릭 메서드   
```java
public static <E/*타입 매개변수 목록*/> Set<E/*반환 타입*/> union(Set<E/*파라미터 타입*/> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

#### 코드 30-3) 제너릭 메서드를 활용하는 간단한 프로그램
```java
public void main(String[] args) {
    Set<String> guys = Set.of("톰", "딕", "헤리");
    Set<String> stooges = Set.of("래리", "모에", "컬리");
    Set<String> aflCio = union(guys, stooges);
    System.out.println(aflCio);
}
```

### 재너릭 싱글턴 팩터리

불변 객체를 여러 타입으로 활용할 때가 있다.   
제네릭은 런타임시 타입 정보가 소거(아이템28) 되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.   
객체를 매개변수화하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리가 필요하다.  
##### 이 정적 팩터리를 제네릭 싱글턴 팩터리라고 한다.

#### 코드30-4) 제너릭 싱글턴 팩터리 패턴 - 항등함수
```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarinings("unchecked")
public static <T> UnaryOperator<T> identityFunction(){
    return (UnaryOperator<T>) IDENTITY_FN;
}
```
IDENTITY_FN을 UnaryOperator<T>로 형변환하면 T가 어떤 타입이든 UnaryOperator<Object> UnaryOperator<T>가 아니여서 비검사 형변환 경고가 발생하지만,   
항등함수는 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로 T가 어떤 타입이든 UnaryOperator<T>를 사용해도 타입 안전하다.   
따라서 비검사 형변환 경고는 숨겨도 된다.   

#### 코드30-5) 제너릭 싱글턴을 사용하는 예
```java
public static void main(String[] args) {
    String[] strings = { "삼베", "대마", "나일론" };
    UnaryOperator<String> sameString = identityFunction();
    for (String string : strings) {
        System.out.println(sameString.apply(s));
    }

    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number number : numbers) {
        System.out.println(sameNumber.apply(number));
    }
}
```

### 재귀적 한정적 타입   

재귀적 타입 한정은 자신이 들어간 표현식을 사용하여 타입 매개변수의 범위를 한정하는 개념이다.   
이런 재귀적 타입 한정은 주로 타입의 자연적 순서를 지정해주는 Comparable과 함께 사용된다.     

```java
public interface Comparable<T>{
	int compareTo(T o);
}
```
타입 매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.

실제 거의 모든 타입은 자기 자신과 같은 타입만 비교가 가능하다. String은 Comparable<String>을 구현하고 Integer는 Comparable<Integer>를 구현한다.

Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 정렬, 검색등 기능을 수행하는데 이런 기능을 수행하기 위해 컬렉션에 담긴 모든 원소가 상호 비교되어야 한다.

#### 코드30-6) 재귀적 타입 한정을 이용해 상호 비교 할 수 있음을 표현
```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```
<E extends Comparable<E>>가 모든 타입 E는 자신과 비교할 수 있다는 의미를 갖는다.   

=> E로 받을 타입은 오직 Comparable<E>를 구현한 타입만 가능하다는 뜻이다. 즉, Comparable을 구현한 타입만 가능하다는 뜻이다.  

아래는 재귀적 타입 한정을 이용한 메서드를 구현했다. 컴파일오류나 경고는 발생하지 않으며 컬렉션에 담긴 원소의 자연적 순서를 기준으로 최댓값을 계산한다.

#### 코드30-7) 컬렉션에서 최대값을 반환한다. - 재귀적 타입 한정 사용
```java
public static <E extends Comparable<E>> E max(Collection<E> c){
    if(c.isEmpty()){
       throw new IllegalArgumentException("컬렉션이 비었습니다.");
    }
        
    E result = null;
    for (E e : c){
        if(result == null || e.compareTo(result) > 0){
            result = Objects.requireNonNull(e);
        }
    }
    return result;
}
```

## 정리   
#### 클라이언트에서 입력 매개변수 및 반환값을 명시적으로 형변환하는 메서드보다 제네릭 메서드가 더 안전하고 사용하기도 쉽다.   
#### 형 변환은 런타임 시에 에러를 동반하기 쉬우므로 제네릭을 사용하자.   
