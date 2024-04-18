## 아이템42. 익명 클래스보다는 람다를 사용하라

### 기존 자바(JDK 1.8 이전)
- 기존 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용
- 이러한 인터페이스의 인스턴스를 함수 객체라고하고, 특정 함수나 동작 표현
- JDK 1.1 이후에는 함수 객체를 만들 때 익명 클래스를 사용

익명 클래스의 인스턴스를 함수 객체로 사용하는 경우
```java
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```

- 전략 패턴(Comparator 인터페이스가 정렬을 담당하는 추상 전략을 의미, 익명 클래스로 구체적인 전략 전달)
- 코드가 너무 길어 자바는 함수형 프로그래밍에 적합하지 않았음

### 자바 8 이후
1. 함수형 인터페이스를 람다식으로 표현 가능

전략을 람다로 전달
```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- 람다, 매개변수(s1, s2), 반환값의 타입은 컴파일러가 추론(제네릭)
- 컴파일러가 타입을 결정하지 못할 경우 컴파일 오류가 나고, 프로그래머가 직접 명시 필요(나머지는 명시 생략)

람다 자리에 비교자 생성 메서드를 사용하여 간결하게 표현 가능
```java
Collections.sort(words, comparingInt(String::length));
```

자바 8에 List 인터페이스에 추가된 sort 메소드를 사용하여 더 간결하게 표현 가능(인터페이스 default 메소드)
```java
words.sort(comparingInt(String::length));
```

2. 함수형 객체를 실용적으로 사용 가능해짐

아이템 34의 Operation 열거 타입 개선 가능

기존 방식(상수별 클래스 몸체를 사용해 각 상수에서 apply 메서드를 재정의)
```java
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
  }

  private final String symbol;

  Operation(String symbol) { this.symbol = symbol; }

  @Override public String toString() { return symbol; }
  public abstract double apply(double x, double y);
}
```

- 상수별 클래스 몸체를 구현하는 방식보다는 열거 타입에 인스턴스 필드를 두는것이 낫다.
- 람다를 이용하면 인스턴스 필드에 람다를 저장하여 간결하고 깔끔하게 개선 가능

인스턴스 필드와 람다 사용
```java
public enum Operation {
  PLUS("+", (x, y) -> x + y),
  MINUS("-", (x, y) -> x - y),
  TIMES("*", (x, y) -> x * y),
  DEVIDE("/", (x, y) -> x / y);

  private final String symbol;
  private final DoubleBinaryOperator op;

  Operation(String symbol, DoubleBinaryOperator op) {
    this.symbol = symbol;
    this.op = op;
  }

  @Override public String toString() { return symbol; }
  public doube apply(double x, double y) {
    return op.applyAsDouble(x, y);
  }
}
```

- 람다는 이름도 없고 문서화도 못하기 때문에 코드 자체로 동작이 명확하게 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다(길어야 세줄)
- 열거 타입 생성자에 넘겨지는 인수들의 타입은 컴파일타임에 추론되기 때문에, 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근 불가(인스턴스는 런타임에 생성)
- 상수별 클래스 몸체를 사용해야 하는 경우 : 줄 수가 많아지거나, 인스턴스 필드나 메서드를 사용해야 하는 경우

## 아이템 43. 람다보다는 메서드 참조를 사용하라

함수 객체를 람다보다 더 간결하게 만드는 방법 : 메서드 참조(::로 표현, static, 특정 인스턴스/클래스의 메서드를 사용하는 방법)

예시 - 임의의 키와 Integer 값을 매핑하는 프로그램의 일부(두 Integer 인수(count, incr)의 합을 반환)
```java
map.merge(key, 1, (count, incr) -> count + incr);
```

메서드 참조 사용
```java
map.merge(key, 1, Integer::sum);
```

- 어떤 람다는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되는 경우 -> 람다 사용
- 보통은 메서드 참조가 더 간결함

예시 - 람다가 더 나은 경우
```java
service.execute(GoshThisClassNameIsHumongous::action);  // 메서드 참조 사용

service.execute(() -> action());  // 람다 사용
```

메서드 참조의 유형

1. 정적 : 정적 메서드를 가리키는 메서드 참조 ``` Integer::parseInt  // str -> Integer.parseInt(str) ```
2. 한정적(인스턴스) : 수신 객체(참조 대상 인스턴스)를 특정(함수 객체가 받는 인수와 참조되는 메서드가 받는 객체가 동일) ``` Instant.now()::isAfter  // Instant then = Instant.now(); t -> then.isAfter(t) ```
3. 비한정적(인스턴스) : 수신 객체를 특정하지 않음(함수 객체를 적용하는 시점에 수신 객체를 명시) ``` String::toLowerCase   // str -> str.toLowerCase() ```
4. 클래스 생성자 :  ``` TreeMap<K,V>::new    // () -> new TreeMap<K,V>() ```
5. 배열 생성자  ``` int[]::new  // len -> new int[len]  ```
