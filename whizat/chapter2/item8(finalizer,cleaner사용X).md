## 아이템8. finalizer와 cleaner 사용을 피하라

### finalizer, cleaner
- cleaner가 finalizer보다 덜 위함하지만 둘 다 예측할 수 없고, 느리고, 일반적으로 불필요하다.
- 가비지 컬렉터에 의해서 언제 수행될지 알 수 없다.
- 성능 문제
- 보안 문제

### 그럼 언제 쓰는걸까?
1. 자원 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
2. 네이티브 객체를 회수할 때 사용한다.
``` java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles; // 네이티브 피어를 가리키는 포인터를 담는 final long 변수가 되면 2번 케이스

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }
    @Override public void close() {
        cleanable.clean();
    }
}
```
run 메서드가 호출되는 상황
1. 클라이언트가 close 메서드를 호출할 때
2. 객체를 회수할 때까지 클라이언트가 close를 호출하지 않을 때, cleaner가 호출해줌(아마도)

### 핵심 정리
> cleaner(자바 8까지는 finalizer)는 안정망 역할이나 중요하지 않는 네이티브 자원 회수 용으로만 사용하자. 
> 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.
