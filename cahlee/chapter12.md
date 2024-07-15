## 아이템89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

* 생성자를 private으로 선언하여 바깥에서 생성자를 호출하지 못하게 막는 방식의 싱글턴 패턴은 implements Serializable이 추가되는 순간 싱글턴이 아니게 된다.
(이 인스턴스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 됨)

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }

  public void leaveTheBuilding() { ... }
}
```

* readResolve 메서드를 적절히 정의해뒀다면 역 직렬화 시 새로 생성된 객체 대신 적절한 객체를 반환할 수 있다.

```java
// 인스턴스 통제를 위한 readResolve - 개선의 여지가 있다.
private Object readResolve() {
  // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
  return INSTANCE;
}
```

* 위 메서드는 역직렬화한 객체를 무시하고, 클래스 초기에 만들어진 Elvis 인스턴스를 반환한다(실 데이터를 가질 필요가 없으니 모든 인스턴스 필드를 transient로 선언)
* 모든 인스턴스 필드를 transient로 선언하지 않으면 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조 공격 가능
  1. readResolve 메서드와 인스턴스 필드 하나를 포함한 도둑(stealer) 클래스 작성
  2. 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 이 도둑의 인스턴스로 교체(싱글턴은 도둑을 참조하고, 도둑은 싱글턴을 참조하는 순환고리 생성)
  3. 싱글턴이 역직렬화될 때 도둑의 readResolve가 먼저 호출되어, 싱글턴의 readResolve가 수행되기 전 싱글턴의 참조 획득)
  4. 도둑의 readResolve 메서드에서 해당 인스턴스 필드의 참조 값을 정적 필드로 복사(readResolve 메서드 종료 후에도 참조 가능)
  5. 도둑의 readResolve 메서드에서 비휘발성 필드를 원래 타입에 맞는 값으로 반환(생략하면 ClassCastException 반환)

잘못된 싱글턴 - transient가 아닌 참조 필드를 가지고 있다.
```java
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {}

  // transient 가 아님
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() {
    return INSTANCE;
  }
```

도둑 클래스
```java
public class ElvisStealer implements Serializable {
  static Elvis impersonator;
  private Elvis payload;

  private Object readResolve() {
    // resolve되기 전의 Elvis 인스턴스의 참조를 저장한다.
    impersonator = payload;

    // favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
    return new String[] {"A Fool Such as I"};
  }
}
```

직렬화의 허점을 이용해 싱글턴 객체를 2개 생성한다.
```java
public class ElvisImpersonator {
  // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림
  private static final byte[] serializedForm = {
    (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
    0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
    (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
    0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
    0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
    0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
    0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
    0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
    0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
    0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
    0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
    0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
  };

  public static void main(String[] args) {
    // ElvisStealer.impersonator를 초기화한 다음,
    // 진짜 Elvis(즉 Elvis.INSTANCE)를 반환한다.
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;

    elvis.printFavorites();
    impersonator.printFavorites();
  }
}
```

위 프로그램을 실행하면 2개의 인스턴스 생성 결과 표시

[Hound Dog, Heartbreak Hotel]

[A Fool Such as I]

* favoriteSongs 필드를 transient로 선언하여 문제 해결 가능하지만 Elvis를 원소 하나짜리 열거 타입으로 바꾸는게 낫다.
(readResolve 메서드를 사용하여 방어하는 방법은 깨지기 쉽고 신경을 많이 써야함)
* 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장

열거 타입 싱글턴 - 전통적인 싱글턴보다 우수하다.
```java
public enum Elvis {
  INSTANCE;
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
}
```

### readResolve 메서드의 접근성

* final 클래스라면 readResolve 메서드는 private이어야 한다.
* final이 아닌 클래스에서는 하위 클래스 사용 여부를 고려해야한다.
  1. private으로 선언하면 하위 클래스에서 사용 불가
  2. package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용 가능
  3. protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용 가능
  4. protected나 public이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 ClassCastException 발생

## 아이템90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

* Serializable을 구현하면 생성자 이외의 방법으로 인스턴스가 생성되어 버그와 보안문제가 일어날 수 있다.
* 직렬화 프록시 패턴(serializable proxy pattern)으로 위험을 줄일 수 있다.
  1. 바깥 클래스의 논리적 상태를 표현하는 중첩 클래스를 설계해 private static으로 선언(바깥 클래스의 직렬화 프록시)
  2. 중첩 클래스의 생성자는 단 하나고, 바깥 클래스를 매개변수로 받아 단순히 인수로 받은 인스턴스의 데이터를 복사
  3. 바깥 클래스에 아래 writeReplace 메서드 추가(직렬화가 이루어지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환)
  4. 바깥 클래스에 아래 readObject 메서드를 추가
  5. 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가(직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환)

Period 클래스용 직렬화 프록시
```java
private static class SerializationProxy implements Serializable {
  private final Date start;
  private final Date end;

  SerializationProxy(Period p) {
    this.start = p.start;
    this.end = p.end;
  }

  private static final long serialVersionUID = 12903810923812093L;  // 아무 값이나 상관 없다.

  private Object readResolve() {
    return new Period(start, end);
  }
}
```

직렬화 프록시 패턴용 writeReplace 메서드
```java
private Object writeReplace() {
  return new SerializationProxy(this);
}
```

직렬화 프록시 패턴용 readObject 메서드
```java
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
  throw new InvalidObjectException("프록시가 필요합니다.");
}
```

### 직렬화 프록시의 장점

* 방어적 복사처럼, 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단
* Period의 필드를 final로 선언해도 되어 진정한 불변으로 생성 가능
* 직렬화 공격의 목표가 될지 고민이 필요없다.
* 역직렬화 시 유효성 검사를 수행하지 않아도 된다.
* 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다.

#### EnumSet의 사례

- public 생성자 없이 정적 팩터리들만 제공
- OpenJDK에서는 열거 타입의 크기에 따라 원소가 64개 이하면 RegularEnumSet, 이상이면 JumboEnumSet 사용
- 64개의 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개를 추가하고 역직렬화를 하는 경우, 직렬화 프록시 패턴을 사용하여 적절한 인스턴스 반환 가능

EnumSet의 직렬화 프록시
```java
private static class SerializationProxy <E extends Enum<E>> implements Serializable {
  // 이 EnumSet의 원소 타입
  private final Class<E> elementType;

  // 이 EnumSet 안의 원소들
  private final Enum<?>[] elements;

  SerializationProxy(EnumSet<E> set) {
    elementType = set.elementType;
    elements = set.toArray(new Enum<?>[0]);
  }

  private Object readResolve() {
    EnumSet<E> result = EnumSet.noneOf(elementType);
    for (Enum<?> e : elements)
      result.add((E) e);
    return result;
  }

  private static final long serialVersionUID = 2480234809234234L;
}
```

### 직렬화 프록시의 단점

* 클라이언트가 확장할 수 있는 클래스에는 적용 불가
* 객체 그래프에 순환이 있는 클래스에도 적용 불가(직렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어지지 않아 readResolve 안에서 호출하려고하면 ClassCastException 발생)
* 성능 문제
