# 아이템34: int 상수 대신 열거 타입을 사용하라
선결론: 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
- 정의된 상수 외에 다른 값은 허용 X
- 예) 사계절, 태양계, 카드게임
## 정수 열거 패턴(안티 패턴)
``` java
public static final int FRUIT_APPLE = 0;
public static final int FRUIT_ORANGE = 1;
public static final int FRUIT_BLUEBERRY = 2;

public static final int SEASON_SPRING = 0;
public static final int SEASON_SUMMER = 1;
public static final int SEASON_AUTUMN = 2;
public static final int SEASON_WINTER = 3;
```
### 단점
- 컴파일시 타입 안전 보장 X
- 접두어를 통해서 이름 충돌을 방지해야 함
- 상수값이 바뀌면 다시 컴파일 해야 함
- 같은 정수 열거 그룹에 속한 모든 상수를 순회할 방법 없음
### 문자열 열거 패턴(안티 패턴)
- 문자열 값 하드코딩하는 케이스 발생할 수 있음(컴파일 타임에 잘못 되었는지 확인 불가)
- 문자열 비교 성능 느림

## 열거 타입(enum type)
``` java
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH;
}
// 열거 타입은 사실 아래와 같음(상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개)
public class Apple {
    public static final FUJI = new Apple();
    public static final PIPPIN = new Apple();
    public static final GRANNY_SMITH = new Apple();
}
```
### 열거 타입은 하나의 클래스
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개
- 밖에서 접근할 수 있는 생성자를 제공하지 않아 사실상 final
- 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나만 존재함을 보장
### 장점
- 컴파일 타입 안전성 제공
- 열거 타입에 각자 네임스페이스가 있어 같은 이름 상수도 사용할 수 있음
- 상수값이 바뀌어도 컴파일 할 필요 없음. 상수에 인스턴스를 할당 했기 때문에 클라이언트에 각인되지 않음.
- 그룹내 상수 순회 가능(values())
- 메서드나 필드 추가 가능
- 인터페이스 구현 가능
- Object 클래스의 메소드, Comparable, Serializable 인터페이스도 구현해 놓음
### 열거 타입의 메소드 및 필드 추가
- 각 상수에 연관된 데이터나 동작을 표현하고 싶을 때 사용
- 열거 타입 상수 각각을 특정 데이터와 연결 지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 됨
- 태양계 행성 예제
  - 열거 타입은 근본적으로 불변이라 모든 필드는 final
  - 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메소드를 두는게 나음
``` java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```
### 상수 순회
- 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메소드 values()
``` java
for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
```
### 상수 제거
- 열거 타입에 있는 상수를 하나 제거 하더라도 클라이언트에서 직접 참조하지 않으면 문제 없음
- 직접 참조한 함수는 컴파일 에러가 발생하여 알 수 있음(근데 이건 static 상수도 마찬가지 아닌가..?)
### 접근 제한
- 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메소드로 구현
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만드록, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만들자
### 상수별 동작을 다르게 하기
#### switch문 사용(안티 패턴)
``` java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
    // 상수가 뜻하는 연산을 수행한다.
    public double apply(double x, double y) {
        switch(this) {
          case PLUS: return x + y;
            ...
        }
        throw new AssertionError("...");
    }
}
```
- throw 절은 실행될 일이 없지만 생략하면 컴파일 에러(혹은 default 작성 필요)
- 새로운 상수 추가하면 case문 추가 필요
#### 상수별 메소드 구현
- 추상 메소드를 선언하고 각 상수별 재정의
``` java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    public abstract double apply(double x, double y);
}
```
### 열거 타입용 fromString
``` java
- toString: 상수 이름을 문자열로 반환
- valueOf: 상수 이름을 입력 받아 해당하는 상수를 반환
- toString을 오버라이딩 하여 해당 열거 타입 상수로 반환해주는 fromString 예제
@Override public String toString() { return symbol; }

// 코드 34-7 열거 타입용 fromString 메서드 구현하기 (216쪽)
private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(
                toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```
- 열거 타입 상수가 생성된 후, 정적 필드 초기화
  -> 생성자에서 stringToEnum에 put 할 수 없음
- Optional을 사용하여 해당 문자열이 없는 상수가 있음을 알려 클라이언트에서 대체 가능하게 함
### 열거 타입 상수끼리 코드 공유하는 방법들
#### 값에 따라 분기(안티 패턴)
``` java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY,
    SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minsWorked, int payRate) {
        int basePay = minsWorked * payRate;
        int overtime;
        switch(this) {
            case SATURDAY: case SUNDAY:  // 주말
                overTimePay = basePay / 2;
                break;
            default:  // 주중
              overtimePay = minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```
- 새로운 값을 열거 타입에 추가하면, case 문을 추가해야 함
#### 전략 열거 타입 패턴
``` java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```
- 열거 타입 생성자에서 원하는 전략적 열거 타입 선택하여, 잔업 수당 계산을 전략적 열거 타입에 위임
- switch문 필요 없음
#### 상수별 동작 혼합(switch 사용)
``` java
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS:   return Operation.MINUS;
        case MINUS:  return Operation.PLUS;
        case TIMES:  return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;

        default:  throw new AssertionError("Unknown op: " + op);
    }
}
```
- 각 연산에 반대되는 연산으 반환하는 메소드
- 원래 열거 타입에 없는 기능을 수행
- 의미상 열거타입에 속하지 않거나, 종종 쓰여서 열거 타입 안에 포함될만큼 유용하지 않은 경우 사용
### 결론
- 성능은 정수 상수와 별반 다르지 않음
- 메모리에 올리는 공간, 시간이 들지만 크게 체감되는 수준은 아님
- **필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자**
