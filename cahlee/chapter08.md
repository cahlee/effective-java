## 아이템48. 스트림 병렬화는 주의해서 적용하라

### 자바는 동시성 측면에서 항상 앞서갔다.
- 최초 릴리스 : 스레드, 동기화, wait/notify 지원
- 자바5 : java.util.concurrent 라이브러리(동시성 컬렉션), Executor(실행자) 프레임워크 지원
- 자바7 : 고성능 병렬 분해 프레임워크인 포크-조인(fork-join) 패키지 추가
- 자바8 : parallel 메소드 지원(파이프라인을 병렬 실행할 수 있는 스트림)

### 자바로 동시성 프로그램을 작성하기는 쉬워지고 있지만, 올바르게 작성하는 일은 여전히 어렵다.
- 동시성 프로그래밍을 할 때는 안전성(safety)와 응답 가능(liveness) 상태 유지 필요
- 병렬 스트림 파이프라인 프로그래밍도 동일

스트림을 사용해 처음 20개의 메르센 소수를 생성하는 프로그램
```java
public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
      .filter(mersenne -> mersenne.isProbablePrime(50))
      .limit(20)
      .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
  return Stream.interate(TWO, BigInteger::nextProbablePrime);
}
```

위 코드의 경우 속도를 높이기 위해 스트림 파이프라인의 parallel()을 호출하면 아무것도 출력하지 못하면서 CPU는 90%의 상태가 무한히 계속됨(응답 불가)
-> 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문
-> 데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.

파이프라인 병렬화는 limit를 다룰 때 CPU 코어가 남으면 원소를 몇 개 더 처리한 후 결과를 버린다.
-> 새로운 메르센 소수를 찾을 때 비용은 이전 원소 전부를 계산한 비용을 합친것보다 크다.

### 병렬화의 효과가 좋은 경우
- 스트림의 소스가 ArrayList, HashMap, ConcurrentHashMap의 인스턴스인 경우나 배열, int 범위, long 범위 일때
- 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어 일으 다수의 스레드에 분배하기 좋다
- 나누는 작업은 Spliterator가 담당 : Stream이나 Iterable의 spliterator 메서드로 획득 가능
- 원소들을 순차적으로 실행할 때 참조 지역성(이웃한 원소의 참조들이 메모리에 연속해서 저장)이 좋음 : 데이터가 주 메모리에서 캐시 메모리로 전송되는 시간이 적음
- 기본 타입의 배열은(참조x) 데이터 자체가 메모리에 연속해서 저장되기 때문에 참조 지역성이 가장 뛰어나다.

### 스트림 파이프라인의 종단 연산의 동작 방식은 병렬 수행 효율에 영향
- 종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하면서 순차적인 연산이라면 파이프라인 병렬 수행 효과는 제한

병렬화에 가장 적합한 종단 연산
- 축소(reduction) - 모든 원소를 하나로 합치는 작업 : Stream의 reduce 메서드 중 하나(min, max, count, sum)
- 조건에 맞으면 바로 반환되는 메서드 : anyMatch, allMatch, noneMatch

적합하지 않은 종단 연산
- 가변 축소(mutable reduction) : Stream의 collect 메서드(컬렉션을 합치는 부담이 큼)

직접 구현한 Stream, Iterable, Collection이 병렬화의 이점을 제대로 누리려면?
-> spliterator 메서드를 반드시 재정의하고 결과 스트림의 병렬화 성능 테스트 필요 : 어렵다

스트림을 잘못 병렬화 할 경우 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작 발생(안전 실패)
- 병렬화한 파이프라인이 사용하는 mappers, filters, 다른 함수 객체가 명세대로 동작하지 않음

Stream 명세는 사용되는 함수 객체에 대한 엄중한 규약 정의
- 예시 : Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 결합법칙을 만족, 간섭받지 않고, 상태를 갖지 않아야한다.
- 규약을 지키지 못해도 파이프라인을 순차적으로 수행하면 문제가 없을 수 있으나 병렬로 수행하면 실패

위 모든 조건들을 만족해도 병렬화에 드는 추가 비용을 상쇄하지 못하면 성능 향상은 미미함
- 간단하게 추정해보는 방법 : 스트림 안의 원소 수 x 원소당 수행되는 코드 줄 수 -> 최소 수십만 이상이면 성능 향상 가능
- 변경 전후로 반드시 성능 테스트를 진행하여 사용할 가치가 있는지 확인 필요
- 병렬 스트림 파이프라인도 공통의 포크-조인 풀에서 수행(같은 스레드 풀)되므로 시스템의 다른 부분에도 영향

### 조건이 잘 갖춰지면 parallel 메서드 호출 하나로 프로세서 코어수에 비례하는 성능 향상 가능

소수 계산 스트림 파이프라인(n보다 작거나 같은 소수의 개수를 계산) - 병렬화에 적합
```java
static long pi(long n) {
  return LongStream.rangeClose(2, n)
              .mapToObj(BigInteger::valueOf)
              .filter(i -> i.isProbablePrime(50))
              .count();
}
```

소수 계산 스트림 파이프라인 - 병렬화 버전
```java
static long pi(long n) {
  return LongStream.rangeClose(2, n)
              .parallel()
              .mapToObj(BigInteger::valueOf)
              .filter(i -> i.isProbablePrime(50))
              .count();
}
```

### 무작위 수들로 이뤄진 스트림을 병렬화 할 때
- ThreadLocalRandom : 단일 스레드에서 사용하고자 개발 - 쓰지 말것
- SplittableRandom : 멀티 스레드에서 사용하고자 개발
- Random(구식) : 모든 연산을 동기화하기 때문에 최악 - 쓰지 말것

## 아이템49. 매개변수가 유효한지 검사하라

### 메서드(+생성자) 대부분은 입력 매개변수의 값이 특정 조건을 만족해야함
- ex. 양수이어야한다, null이 아니어야한다. 등
- 반드시 문서화 필요
- 메서드 몸체가 시작되기 전에 검사 필요(오류는 가능한 한 빨리 발생한 곳에서 잡아야 한다) - 즉각적으로 예외 throw

### 매개변수 검사를 제대로 하지 않을 경우
1. 메서드가 수행되는 중간에 모호한 예외 발생
2. 메서드는 문제없이 수행되었으나 잘못된 결과(객체) -> 미래의 알 수 없는 시점에 관련 없는 오류 발생

### public, protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외 문서화 필요(@throws 자바독 태그)
- ex. IllegalArgumentException, IndexOutOfBoundsException, NullPointerException 등

제약을 어겼을 때 발생하는 예외도 함께 기술 필요

```java
/**
  * (현재 값 mod m) 값을 반환한다. 이 메서드는
  * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
  *
  * @param m 계수(양수여야 한다.)
  * @return 현재 값 mod m
  * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
  */
public BigInteger mod(BigInter m) {
  if (m.signum() <= 0)
    throw new ArithmeticException("계수(m)은 양수여야 합니다. " + m);
  ... // 계산 수행
}
```

m.signum() 호출 시 NPE가 던져지는데 해당 설명이 없는 이유
- BigInteger 클래스에 있다(클래스 수준 주석) : 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는것보다 깔끔함
- @Nullable(+다른 애너테이션) 사용 가능 : 표준 x

자바 7부터는 java.util.Objects.requireNonNull 메서드 사용 가능
- 유연 + 편함
- 원하는 메시지 지정
- 입력을 그대로 반환(값을 사용하는 동시에 null 검사 수행 가능)

``` java
this.strategy = Objects.requireNonNull(strategy, "전략");
```

자바 9에는 Objects에 범위 검사 기능 추가
- checkFromIndexSize, checkFromToIndex, checkIndex : 예외 메서드 지정 불가, 리스트, 배열 전용으로 설계, 닫힌 범위 사용 불가

### 공개되지 않은 메서드는 패키지 제작자가 호출되는 상황 통제
- 유효한 값만이 메서드에 넘겨지리라는 것을 보증해야 한다.
- 단언문(assert) 사용하여 매개변수 유효성 검증

```java
private static void sort(long a[], int offset, int length) {
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >= 0 && length <= a.length - offset;
  ... // 계산 수행
}
```

단언문 : 자신이 단언한 조건이 무조건 참이라고 선언
- 실패하면 AssertionError throw
- 런타임에 아무런 효과나 성능 저하가 없다(-ea 혹은 --enableassertions 플래그 설정 제외)

### 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 메서드는 더 신경써서 검사 필요

```java
static List<Integer> intArrayAsList(int[] a) {
  Objects.requireNonNull(a);

  // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
  // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
  return new AbstractList<>() {
    @Override public Integer get(int i) {
      return a[i];
    }
    ... (생략)
  }
}
```

생략했으면 새로 생성한 List 인스턴스를 반환하는데, 클라이언트가 돌려받은 List를 사용할 때 NPE 발생
-> 어디서 가져왔는지 추적하기 어려워 디버깅이 힘들어진다.

생성자는 "나중에 쓰려고 저장하는 메서드의 유효성을 검사하라"의 특수 사례
- 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않도록 하는 데 꼭 필요

### 메서드 몸체 실행 전에 매개변수 유효성 검사 규칙의 예외
- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 수행될 때 : ex. Collections.sort(List)의 경우 상호 비교 검사 시 ClassCastException 발생
- 암묵적 유효 검사에 너무 의존하면 실패 원자성을 해칠 수 있으니 주의 필요
- 계산 과정에서 필요한 유효성 검사가 이뤄지지만 실패했을때 잘못된 예외 발생 가능 - 실제 발생한 예외가 API에 명시된 예외 상이(예외 번역 관용구 사용 필요)

### 매개변수 제약은 적을수록 좋다 - 메서드는 최대한 범용적으로 설계 필요(특정한 제약을 내재한 경우)

## 아이템54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

null이 반환하면, 클라이언트에서 null 상황을 처리하는 코드 추가 작성 필요

방어 코드가 없으면 오류 발생 가능성이 높아지고 반환 하는 쪽에서도 특별 취급해줘야하여 코드 복잡도가 높아짐

컬렉션이 비었으면 null을 반환 - 나쁜 사례
```java
private final List<Cheese> cheesesInStock = ...;

/**
  * @return 매장 안의 모든 치즈 목록을 반환한다.
  * 
  */
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheeseInStock);
}
```

상황을 처리하는 클라이언트 방어 코드
```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
  System.out.println("좋았어, 바로 그거야.");
```

- 빈 컨테이너를 할당하는 비용으로 인한 성능 차이는 미미함
- 빈 컬렉션과 배열은 새로 할당하지 않고 반환 가능 -> 불변 객체(Collections.emptyList, Collections.emptySet, Collections.emptySet) 또는 길이가 0인 배열 반환

빈 컬렉션을 반환하는 올바른 예
```java
public List<Cheese> getCheese() {
  return new ArrayList<>(cheesesInStock);
}
```

최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 함
```java
public List<Cheese> getCheese() {
  cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheeseInStock);
}
```

길이가 0일 수도 있는 배열을 반환하는 올바른 방법
```java
public Cheese[] getCheese() {
  return cheesesInStock.toArray(new Cheese[0]);
}
```

최적화 - 빈 배열을 매번 새로 할당하지 않도록 했다.(길이가 0짜리 배열은 모두 불변)
```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
  return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

나쁜 예 - 배열을 미리 할당하면 성능이 나빠진다.
```java
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

## 아이템55. 옵셔널 반환은 신중히 하라

### 메서드가 특정 조건에서 값을 반환할 수 없을 때

- 자바8 이전
1. 예외 throw - 예외는 예외적인 상황에서만 사용해야하며, 예외 생성 시 스택 추적 전체를 캡처하여 고비용
2. null 반환(객체) - NPE를 방어하기 위한 코드 작성 필수

- 자바8 이후 : Optional<T> 반환 가능 - null이 아닌 T 타입 참조를 하나 담거나 빈 Optional 반환

#### Optional<T>
원소를 최대 1개 가질 수 있는 불변 컬렉션(Collection<T>를 구현한건 아님)

- 아무것도 담지 않은 Optional : 비었다
- 어떤 값을 담은 Optional : 비지 않았다
- 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉽고, null을 반환하는 메서드보다 오류 가능성이 낮음

컬렉션에서 최댓값을 구한다(컬렉션이 비었으면 예외를 던진다)
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("빈 컬렉션");

  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);

  return result;
}
```

컬렉션에서 최댓값을 구해 Optional<E>로 반환한다.
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty())
    return Optional.empty();    // 빈 옵셔널

  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);

  return Optional.of(result);    // 값이 든 옵셔널
}
```

- 빈 옵셔널은 Optional.empty()로 생성
- 값이 든 옵셔널은 Optional.of(value)로 생성 : value가 null이면 NPE 발생
- null값도 허용하는 옵셔널 : Optional.ofNullable(value) 사용
- Optional을 반환하는 메서드에서는 절대 null을 반환하지 말것
- 스트림의 종단 연산 중 상당수가 옵셔널 반환

컬렉션에서 최댓값을 구해 Optional<E>로 반환한다. - 스트림 버전
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  return c.stream().max(Comparator.naturalOrder());
}
```

null을 반환하거나 예외를 던지는 대신 옵셔날 반환을 선택해야하는 기준
- 검사 예외와 취지가 비슷(반환 값이 없을 수도 있음을 API 사용자에게 명확히 전달)
- 비검사 예외를 던지거나 null을 반환하면 API 사용자가 인지하지 못해 오류 발생
- 옵셔널이 반환되면 클라이언트는 값을 받지 못했을 때 행동 선택 필요

옵셔널 활용 1 - 기본값 설정
```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

옵셔널 활용 2 - 원하는 예외 발생
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

옵셔널 활용 3 - 항상 값이 채워져 있다고 가정한다.
```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();  // NoSuchElementException 발생 가능
```

- 기본값을 설정하는 비용이 커서 부담스러우면 Supplier<T>를 인수로 받는 orElseGet을 사용 - 값이 처음 필요할 때 Supplier<T>를 사용해 생성
- filter, map, flatMap, ifPresent 사용
- isPresent : 옵셔널이 채워져 있으면 true, 비어있으면 false 반환 - 신중히 사용(filter, map, flatMap으로 대체 가능)

부모 프로세스의 프로세스 ID를 출력하거나, 부모가 없다면 "N/A" 출력
```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));
```

Optional의 map 사용 예시
```java
System.out.println("부모 PID: " + ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

스트림을 사용하면 옵셔널들을 Stream<Optional<T>>로 받아서 채워진 옵셔널들에서 값을 뽑아 Stream<T>에 건네 담아 처리하는 경우가 많다.
````java
streamOfOptionals
    .filter(Optional::isPresent)
    .map(Optional::get)
```

- 자바9 : Optional에 stream() 메서드 추가(Optional을 Stream으로 변환) - 옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환
```java
steramOfOptionals
    .flatMap(Optional::stream)
```

옵셔널이 해가되는 경우
- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입을 옵셔널로 감싸면 안된다 : Optional<List<T>>를 반환하기보다는 빈 List<T>를 반환
- 옵셔널을 맵의 값으로 사용하지 말 것 : 맵 안에 키가 없다는 사실을 나타내는 방법이 2가지가 됨(키 자체가 없는 경우, 키는 있지만 키가 빈 옵셔널인 경우)
- 컬렉션의 키, 값, 원소나 배열의 원소로 사용하지 말 것
- 박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무겁기 때문에 반환하지 말것 - int, long, double 전용 옵셔널 사용(OptionalInt, OptionalLong, OptionalDouble)

메서드 타입을 T 대신 Optional<T>로 선언되어야 하는 규칙
- 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야할 경우
- Optional<T>를 반환 비용도 고려 필요(성능)

옵셔널을 인스턴스 필드에 저장하는 것이 필요한 경우
- 필수 필드를 갖는 클래스와 선택적 필드를 추가한 하위 클래스 -> 리팩토링 필요
- 필수 필드들이 기본 타입이라 값이 없음을 나타낼 수 있는 방법이 없음 -> 선택적 필드를 옵셔널로 선언하고 getter 메서드들이 옵셔널을 반환하도록 가능
