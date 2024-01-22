## 아이템3: private 생성자나 열거 타입으로 싱글턴임을 보증하라
- 싱글턴: 인스턴스를 오직 하나만 생성할 수 있는 클래스
### 방법1: public static final 필드 방식
```java
public class Elvis {
  public staic final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  public void leaveTheBuilding() { ... }
}
```
- Elvis 생성자가 초기화 할 때 딱 한번 호출된다.
- 리플렉션 API인 AccessibleObject.setAccessible을 사용해서 private 생성자를 호출할 수 있다.
``` java
Class<?> classType = Elvis.class;
Constructor<?> constructor = classType.getDeclaredConstructor();
constructor.setAccessible(true);
Elvis elvis = (Elvis) constructor.newInstance();
```
### 방법2: 정적 팩토리 방식
``` java
public class Elvis {
  public staic final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  public static Elvis getInstance() { return INSTANCE; }
  public void leaveTheBuilding() { ... }
}
```
- 
- 리플렉션을 통한 예외는 똑같이 적용
### 방법3: 열거 타입 방식
