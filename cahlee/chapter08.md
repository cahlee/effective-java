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
