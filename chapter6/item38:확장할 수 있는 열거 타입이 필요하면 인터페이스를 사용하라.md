■열거 타입 확장을 지양하자
-----
열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수하다.

```java
// 타입 안전 열거 패턴 (typesafe enum parttern) 예시
public class TypesafeOperation {
    private final String type;
    private TypesafeOperation(String type) {
        this.type = type;
    }

    public String toString() {
        return type;
    }
    
    public static final TypesafeOperation PLUS = new TypesafeOperation("+");
    public static final TypesafeOperation MINUS = new TypesafeOperation("-");
    public static final TypesafeOperation TIMES = new TypesafeOperation("*");
    public static final TypesafeOperation DIVIDE = new TypesafeOperation("/");
}
```

단, 예외가 있는데 타입 안전 열거 패턴은 확장할 수 있지만 열거 타입은 그럴 수 없다.

타입 안전 열거 패턴은 열거한 값들을 그대로 가져온 다음, 값을 더 추가하여 다른 목적으로 쓸 수 있지만 열거 타입은 그럴 수 없다. 

사실 웬만하면 **열거 타입을 확장을 지양** 해야 한다.

확장한 타입의 원소를 기반 타입의 원소로 취급한다면

그 반대도 성립해야 하는데, 열거 타입은 그렇지 않다.

따라서 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅치 않다.



■특별한 상황에선 열거 타입을 확장하자
-----
연산 코드(operation)는 확장할 수 있는 열거 타입이 어울리는 쓰임이다.

연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻한다.(item 34)

열거 타입으로 이 효과를 내는 방법이 있다.

열거 타입이 **인터페이스**를 구현할 수 있다는 사실을 이용하는 것이다.

연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다.

이때 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다

```java
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operation {
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

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
``` 

열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있다.

그리고 이 인터페이스를 연산의 타입으로 사용하면 된다. 

이렇게 하면 Operation을 구현한 또 다른 열거 타입을 정의해서

기본 타입인 BasicOperation을 대체할 수 있다.

#### 예시 코드) 지수 연산(EXP)과 나머지 연산
```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
``` 

Operation 인터페이스를 사용하도록 작성되어 있기만 하면 된다.

apply가 인터페이스에 선언되어 있으니 열거 타입에 따로 추상 메서드로 선언하지 않아도 된다.

------


개별 인스턴스 수준에서뿐 아니라 타입 수준에서도

기본 열거 타입 대신 확장된 열거 타입을 넘겨서 확장된 열거 타입의 원소 모두를 사용하게 할 수도 있다.


``` java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
``` 

main 메서드는 test 메서드에 ExtendedOperation의 class 리터럴을 넘겨서

확장된 연산들이 무엇인지 알려준다. (여기서 class 리터럴은 한정적 타입 토큰 item 33 역할을 한다) 

``` java
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y)
```

위 매개변수 타입은 class 객체가 열거 타입인 동시에 

Operation의 하위 타입이어야 한다는 뜻이다.

열거 타입이어야 원소를 순회할 수 있고

Operation이어야 원소가 뜻하는 수행할 수 있기 때문이다.

-----

다른 방법으로도 인터페이스를 구현한 enum을 사용할 수 있다.

바로 class 객체 대신 한정적 와일드 카드 타입(item 31)인 Collection<? extends Operation>을 넘기는 방법이다.

``` java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

위 코드는 더욱 간단하며 test 메서드가 조금 더 유연해졌다.

즉, 여러 구현 타입의 연산을 조합해 호출할 수 있게 되었다.

반면 특정 연산에서는 EnumSet(item 36)과 EnumMap(item 37)을 사용하지 못한다.



■인터페이스를 이용한 확장 enum의 단점
-----
인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 문제가 있는데

**바로 열거 타입끼리 구현을 상속할 수 없다**는 점이다.

아무 상태에도 의존하지 않는 경우에는 디폴트 구현(item 20)을 이용해 인터페이스에 추가하는 방법이 있으나,

위의 Operation 인터페이스는 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOperation 모두에 들어가야만 한다.

사실 이 경우는 중복되는 코드가 적지만, 만약 공유하는 기능이 많다면

그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로

코드 중복을 없애야 한다.

■ 정리 
-----
열거 타입(enum)은 확장할 수 없다.

단, 인터페이스를 통해 여러 열거 타입에 동일한 인터페이스를 구현하게 하면 마치 확장하는 것과 비슷한 효과를 낸다.

인터페이스로 작성되었다는 가정하에 서로 얼마든지 대체도 가능하다.
