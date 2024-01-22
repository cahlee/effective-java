## 아이템3: private 생성자나 열거 타입으로 싱글턴임을 보증하라
- 싱글턴: 인스턴스를 오직 하나만 생성할 수 있는 클래스
### 방법1: public static final 필드 방식
```java
public class Elvis {
  public staic final Elvis INSTANCE = new Elvis();
  private Elvis() {}
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
#### 장점
1. 싱글턴 클래스임이 명확하게 드러난다. public static final 필드는 다른 객체를 참조할 수 없다.
2. 간결하다

### 방법2: 정적 팩토리 방식
``` java
public class Elvis {
  private staic final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  public static Elvis getInstance() { return INSTANCE; }
}
```
- 리플렉션을 통한 예외는 똑같이 적용
#### 장점
1. 싱글턴 객체가 아니게 변경이 용이하다.(return new ...)
2. 제네릭 싱글턴 팩터리로 만들 수 있다.(아이템30에서 보자)
3. 정적 팩토리 메서드를 공급자로 사용할 수 있다.
``` java
Supplier<Elvis> supplier = Elvis::getInstance
```
#### 직렬화
1. 필드에 transient(일시적) 추가
2. Serializable에서 역직렬화를 할 때 호출하는 readResolve 메서드를 아래와 같이 수정한다.
``` java
private staic final transient Elvis INSTANCE = new Elvis();
private Object readResolve() {
  return INSTANCE;
}
```

### 방법3: 열거 타입 방식
- 간결, 직렬화, 리플레션 공격 방어
- 상속은 안되고, 구현은 가능
``` java
public enum Elvis {
  INSTANCE;
}
```
