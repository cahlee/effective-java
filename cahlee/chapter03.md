## 아이템36. 비트 필드 대신 EnumSet을 사용하라

열거한 값들이 주로 집합으로 사용될 경우

### 기존 방식

#### 비트 필드 열거 상수

```java
public class Text {
  public static final int STYLE_BOLD           = 1 << 0; // 1
  public static final int STYLE_ITALIC         = 1 << 1; // 2
  public static final int STYLE_UNDERLINE      = 1 << 2; // 4
  public static final int STYLE_STRIKETHROUGH  = 1 << 3; // 8

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
  public void applyStyle(int styles) { ... }
}

// 사용 시
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);  // 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산 수행
```

단점
   
- 비트 필드 값이 그대로 출력되면 해석하기 어렵다. (ex. 3로 출력된 값의 의미를 찾으려면 2진수로 변환 필요?)
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다. (ex. 3로 출력된 값이 STYLE_BOLD | STYLE_ITALIC 임을 찾기 쉽지 않다)
- 최대 몇 비트가 필요한지 미리 예측하여 타입을 선택해야 한다.

### 대안 방식

#### 비트 필드 대신 EnumSet 사용

- java.util 패키지의 EnumSet 클래스는 열거 타입으로 구성된 집합을 효과적으로 표현
- Set 인터페이스를 완벽히 구현하여 타입 안전하고, 다른 Set 구현체와도 함께 사용 가능
- 내부는 비트 벡터로 구현되어 비트 필드에 비견되는 성능 보장

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

  // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
  public void applyStyle(Set<Style> styles) { ... }
}

// 사용 시
text.applyStyles(EnumSet.of(Style.BOLD, STYLE.ITALIC));  // EnumSet의 다양한 정적 팩터리 사용 가능
```

applyStyles 메서드가 EnumSet<Style>이 아닌 Set<Style>을 받은 이유 : 다른 Set 구현체 사용 가능

## 아이템37. ordinal 인덱싱 대신 EnumMap을 사용하라

### 상황

정원(garden)에 심은 식물(Plant)들을 배열 하나로 관리(Plant[] garden)하고 생애주기(LifeCycle) 별로 묶는다.
-> 생애주기별로 총 3개의 집합(Set)을 만들고, 정원을 한바퀴 돌며 각 식물을 해당 집합에 넣는다.

```java
class Plant {
  enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

  final String name;
  final LifeCycle lifeCycle;

  Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }

  @Override public String toString() {
    return name;
  }
}
```

#### 생애주기의 ordinal 값을 배열의 인덱스로 사용한 경우 - 나쁜 사례

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for (int = 0; i < plantsByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

문제점

- 배열은 제네릭과 호환되지 않아 비검사 형변환 필요
- 배열은 각 인덱스의 의미를 몰라 출력 결과에 직접 레이블 매핑 필요
- 배열 인덱스에 정확한 정숫값을 사용하는 것을 보증하지 못함(타입 안전하지 않음) - 런타임 오류 혹은 Exception 발생 가능성 존재

배열의 역할은 열거 타입 상수를 값으로 매핑 -> Map으로 대체 가능 -> 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체 = EnumMap

#### EnumMap을 사용하여 데이터와 열거 타입을 매핑

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden)
  plantsByLifeCycle.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```

EnumMap으로 바꾼 이점

- 더 짧고 명료하고 안전하고 성능도 비등
- 안전하지 않은 형변환 제거
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블 매핑 필요 없음
- 배열 인덱스를 계산하는 과정에서 오류 가능성 0
- 내부 구현 방식(배열 인덱스)을 안으로 숨겨서 Map의 타입 안전성 + 배열의 성능
- EnumMap의 생성자가 받는 키 타입의 Class 객체(Plant.LifeCycle.class)는 한정적 타입 토큰으로 런타임 제네릭 타입정보를 제공

#### 스트림1. 스트림을 사용해 맵을 관리하면 코드 단순화 가능

```java
System.out.println(Arrays.stream(garden)
      .collect(groupingBy(p -> p.lifeCycle)));
```

단점

- EnumMap이 아닌 고유한 Map 구현체를 사용하여, EnumMap의 공간과 성능 이점이 없음

#### 스트림2. 맵 구현체를 명시해 EnumMap을 이용하여 데이터와 열거 타입을 매핑

```java
System.out.println(Arrays.stream(garden)
      .collect(groupingBy(p -> p.lifeCycle,
        () -> new EnumMap<>(LifeCycle.class), toSet())));
```
