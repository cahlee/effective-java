
선요약 : 인터페이스에 디폴트 메소드 추가할 때 잘 생각해야된다.

1. 불변식을 해치지 않고 디폴트 메소드를 작성하긴 어렵다.

예시코드) 자바8의 Collection 인터페이스에 추가된 디폴트 메소드   
```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}

```

이 메소드는 주어진 boolean 함수가 true를 반환하는 모든 원소를 제거한다.   
디폴트 구현은 반복자를 이용해 순회하면서 각 원소를 인수로 넣어 Predicate가 true를 반환하면 반복자의 remove 메소드를 호출해 그 원소를 제거한다.   

그러나  Collection을 구현하는 SynchronizedCollection 에서 디폴트 메서드 removeIf가 제대로 동작하지 않는다.   
아파치버전은 클라이언트가 제공한 객체로락을 거는 능력을 추가로 제공한다. 모든 메소드에서 주어진 락 객체로 동기화한후 내부 컬랙션 객체에 기능을 위임하는 래퍼 클래스이다.   

모든 메소드 호출을 알아서 동기화 해주지 못한다.   

동기화와 관련된 코드가 전혀 없다.   
Collection를 구현하는 (동기화를 제공하는) SynchronizedCollection 에서 디폴트 메서드 removeIf가 제대로 동작하지 않는다.   
Concurrentmodificationexception 발생할 수 있다. 한 스레드가 어떤 Collection을 반복자(iterator)를 이용하여 순회하고 있을때, 다른 한스레드가 해당 Collection에 접근하여 변경을 시도하는 경우이다. 하지만 꼭 멀티스레드 환경에서만 발생 하는것은 아니다. 싱글 스레드 환경에서도 발생할 수 있는데, 위와 같이 어떤 Collection을 순회하고 있는 반복문 안에서, 순회되고 ****있는 Collection에 대한 변경이 시도 될 때 또한 해당 Exception이 발생 하게 된다.   


2. 디폴트 메소드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있다.   

3. ... 잘 설계하자!   



