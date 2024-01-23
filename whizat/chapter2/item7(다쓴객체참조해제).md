## 아이템7. 다 쓴 객체 참조를 해제하라
메모리 누수에 신경써야 하는 케이스
### 자기 메모리 직접 관리하는 클래스
예) Stack
``` java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;  // 다 쓴 참조 해제
    return result;
}
```
### 캐시
- 캐시에 넣어놓은 객체 참조는 객체를 다 사용한 후에는 반환해야한다.
- WeakHashMap을 사용
  - key를 null로 할동하면 자동으로 GC
- 백그라운드 스레드를 이용해서 주기적으로 GC
- 새 엔트리를 추가할 때마다 참조 해제(예: LinkedHashMap.afterNodeInsertion)

### 리스너 또는 콜백
- 콜백을 해지하지 않으면 계속 쌓임
- 역시나 WeakHashMap을 사용
