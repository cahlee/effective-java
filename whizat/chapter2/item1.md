# 2장 객체 생성과 파괴
## 아이템1: 생성자 대신 정적 팩토리 메서드를 고려하라
### 장점
1. 이름을 가질 수 있다.
- 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같은면, 생성자를 정적 팩토리 메서드로 바꾸고 각각의 차이가 잘 들어나는 이름을 지어준다.
```java
// 자동차 클래스
public class Car {
  // 바퀴수
  private int wheelCount;

  public Car(int wheelCount) {
    this.wheelCount = wheelCount;
  }
}

public class CarFatory {
  // 이륜차 객체 생성
  public static Car create2WdCar() {
    return new Car(2);
  }
  // 사륜차 객체 생성
  public static Car create4WdCar() {
    return new Car(4);
  }
}
```
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
- 인스턴스를 통제할 수 있다.
  - 인스턴스를 하나만 생성해서 해당 인스턴스만 사용하도록 할 수 있다.(싱글턴)
  - 생성자를 private으로 만들어 메소드들 통해서만 객체 생성가능하도록 제한할 수 있다.
  - 인스턴스를 미리 만들어 놓아 생성되는 인스턴스를 한정할 수 있다.
  
```java
public class Car {
  private static Map<Integer, Car> carCache = new HashMap<>();

  static {
    carCache.put(2, new Car(2));  // 이륜
    carCache.put(3, new Car(3));  // 삼륜
    carCache.put(4, new Car(3));  // 사륜
  }

  private int wheelCount;
  // 생성자를 private으로 설정해서 정적 팩토리 메소드를 통해서만 객체 생성 가능하도록 제한(아이템4)
  private Car(int wheelCount) {
    this.wheelCount = wheelCount;
  }

  public Car createCarWithWheelCount(int wheelCount) {
    return carCache.get(wheelCount);  // 오륜 차는 인스턴스화 할 수 없음
  }
}
```
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
```java
public class TwoWheeledCar extends Car {
  ...
}
public class ThreeWheeledCar extends Car {
  ...
}
public class FourWheeledCar extends Car {
  ...
}

public class CarFatory {
  public static Car getCarByWheelCount(int wheelCount) {
    if (wheelCount == 2) {
      return new TwoWheeledCar();
    } else if (wheelCount == 3) {
      return new ThreeWheeledCar();
    }
    return new FourWheeledCar();
  }
}
```
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
```java
public class Chapter02 {
  public static void main(String[] args) {
    Car car = CarFatory.getCarByWheelCount(2);
    System.out.println(car.getClass());
  }
}
```
5. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
- DI 패턴으로 설명
``` java

public interface CarRepository {
  Optional<Car> findByCarWheelCount(int wheelCount);
}
@Repository
public class JdbcCarRepository implements CarRepository {
  ...
  @Override
  Optional<Car> findByCarWheelCount(int wheelCount) {
    ...
  }
}

public class MyBatisRepository implements CarRepository {
  ...
  @Override
  Optional<Car> findByCarWheelCount(int wheelCount) {
    ...
  }
}

@Service
public class CarService {
  // 등록된 Bean 중에 타입에 맞는 Bean을 찾아서 DI
  // 기존에 JDBC로 접속하는 방식을 MyBatis를 사용하는 Repository로 바꾸고 싶다면, MyBatisRepository를 생성해서 @Repository 어노케이션을 붙여주면 된다.
  private final CarRepository carRepository;  
  @Autowired
  private CarService(CarRepository carRepository) {
    this.carRepository = carRepository;
  }

  public Optional<Car> findByCarWheelCount(int wheelCount) {
    return carRepository.findByCarWheelCount(wheelCount);
  }
}
```
### 단점
1. private 생성자일 경우 상속이 불가능하다.
- 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩토리 매소드는 프로그래머가 찾기 어렵다.
- 자바독에는 드러나지 않아서 정적 팩토리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.

### 명명 방식
1. from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
```java
Date d = Date.from(instant);
```
3. of
```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```
5. valueOf
```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```
7. instance, getInstance
```java
StackWalker luke = StackWalker.getInstance(options);
```
9. create, newInstance
```java
Object newArray = Array.newInstance(classObject, arrayLen);
```
11. getType
```java
FileStore fs = Files.getFileStore(path);
```
13. newType
```java
BufferedReader br = Files.newBufferedReader(path);
```
15. type
```java
List<Complaint> litany = Collection.list(legacyLitany);
```
