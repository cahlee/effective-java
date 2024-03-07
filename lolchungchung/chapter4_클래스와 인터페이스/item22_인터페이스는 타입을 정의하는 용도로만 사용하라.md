### 선요약 : 인터페이스를 상수 공개용 수단인 상수 인터페이스로 사용하면 문제가 있기에 지양하자. 인터페이스는 타입을 정의하는 용도로만 사용하자. 
===========

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.<br>
달리 말하면 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 얘기하주는 것이다.<br>
인터페이스는 반드시 위 용도로만 사용이 되어야한다.<br>
	
### 상수 인터페이스에 관하여..

상수 인터페이스란 메서드없이 상수를 뜻하는 static final 필드로만 구성된 인터페이스를 말한다.<br>
그리고 이 상수들을 사용하려는 클래스에서는 정규화된 이름을 쓰는 걸 피하고자 그 인터페이스를 구현하곤 한다.<br>

#### 예시 코드)상수 인터페이스 안티패턴 - 사용금지
```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

클래스 내부에서 사용하는 상수는 내부구현에 해당한다.<br>
따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위이다. 클래스가 어떤 상수 인터페이스를 사용하던 사용자에게는 아무런 의미가 없고 오히려 혼란을 주게된다.<br>
동시에 클라이언트 코드가 이 상수들에 종속되어 다음 릴리스에서 상수를 사용하지 않게 되더라도 여전히 상수 인터페이스를 구현해야 한다.<br>

#### 상수 공개 방법 3가지

#### 1. 클래스나 인터페이스 자체에 추가하는 방법(ex Integer, MAX_VALUE, MIN_VALUE, DOUBLE)
```java
public final class Integer extends Number implements Comparable<Integer> {
  ...
  @Native public static final int   MIN_VALUE = 0x80000000;

  @Native public static final int   MAX_VALUE = 0x7fffffff;
  ...
}
```

#### 2. 열거 타입으로 공개하는 방법(Enum 사용,item34)
	
#### 3. 유틸리티 클래스에 담아 공개하는 방법(item4)
```java
public class PhysicalConstants {
  private PhysicalConstants() { }  // private 생성자로 인스턴스화 방지
  
  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```
		
#### 3-1. 정적 임포트를 사용해 상수 이름만 사용
```java
import static effectivejava.chapter4.item22.constantutilityclass.PhysicalConstants.*;

public class Test {
  double atoms(double mols){
    return AVOGADROS_NUMBER * mols;
  }
  ...
  // PhysicalConstants 를 빈번히 사용한다면 정적 임포트가 값어치를 한다.
}
```
