## 아이템72. 표준 예외를 사용하라

자바 라이브러리는 대부분 API에서 쓰기 충분한 예외를 제공하므로 이를 재사용하는 것이 좋다.

### 장점
- 많은 프로그래머에게 익숙해진 규약을 그대로 따르기 때문에, API가 다른 사람이 익히고 사용하기가 쉬워진다.
- API를 사용한 프로그램도 낯선 예외를 사용하지 않게 되어 읽기 쉽다.
- 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.

### 일반적으로 많이 사용하는 표준 예외
- IllegalArgumentException : 호출자가 인수로 부적절한 값을 넘겼을 때 ex. 반복 횟수를 지정하는 매개변수에 음수 전달
- IllegalStateException : 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 ex. 초기화되지 않은 객체를 사용하려 할 때
- NullPointerException : null 값을 허용하지 않는 메서드에 null을 건넬 때 관례 상 IleggalArgumentException 대신 사용
- IndexOutOfBoundsException : 어떤 시퀀스(인덱스)의 허용 범위를 넘는 값을 건넬 때 관례 상 IleggalArgumentException 대신 사용
- ConcurrentModificationException : 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때(가능성을 알려주는 정도)
- UnsupportedOperationException : 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때(구현하려는 인터페이스의 메서드의 일부를 구현할 수 없을 때) ex. 원소를 넣을 수만 있는 List 구현체에 remove 메서드를 호출할 때]

### 미사용 권고하는 표준 예외
-  Exception, RuntimeException, Throwable, Error는 여러 성격의 예외들을 포괄하는 클래스로 안정적으로 테스트할 수 없어 직접 재사용 하지 말자(다른 예외들의 상위 클래스)

### 기타
- 복소수나 유리수를 다루는 객체에서 사용 : ArithmeticExceptoin, NumberFormatException
- 상황에 부합하면 항상 표준 예외를 재사용
- API 문서를 참고해 예외가 어떤 상황에서 던져지는지 꼭 확인 필요
- 예외의 이름뿐 아니라 예외가 던져지는 맥락도 부합할 때만 재사용
- 더 많은 정보가 필요하면 표준 예외를 확장하여 사용
- 예외는 직렬화할 수 있어 많은 부담이 따르니 새로 만들지 않는 것이 좋다
- '주요 쓰임'이 상호 배타적이지 않아 예외를 선택하기가 어려울 경우 일반적인 규칙 : 덱에 남아있는 카드 수를 뽑아주는 메서드에 카드 수보다 큰 인수가 전달되면 -> 인수 값이 무엇이었던 어차피 실패했을 거라면 IllegalStateException, 그렇지 않으면 IllegalArgumentException

## 아이템 73. 추상화 수준에 맞는 예외를 던지라

- 메서드가 저수준 예외를 처리하지 않고 바깥으로 전파하면, 수행하려는 일과 관련이 없어 당황스럽고, 내부 구현 방식을 드러내어 윗 레벨 API를 오염시킨다.
- 다음 릴리스에서 구현 방식을 바꾸면 다른 예외가 튀어나와 기존 클라이언트 프로그램을 깨지게 할 수도 있다.

->  상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다 : 예외 번역

예외 번역
```java
try {
  ...  // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
  // 추상화 수준에 맞게 번역한다.
  throw new HigherLevelException(...);
}
```

### 예시

AbstractSequentialList는 List 인터페이스의 골격 구현 : 이 예외 번역은 List<E> 인터페이스의 get 메서드 명세에 명시된 필수사항임

```java
/**
  * 이 리스트 안의 지정한 위치의 원소를 반환한다.
  * @throws IndexOutOfBoundsException index가 범위 밖이라면,
  *         즉 ({@code index < 0 || index >= size()})이면 발생한다.
  */
public E get(int index) {
  ListIterator<E> i = listIterator(index);
  try {
    result i.next();
  } catch (NoSuchElementException e) {
    throw new IndexOutOfBoundsException("인덱스: " + index);
  }
}
```

### 예외 연쇄
예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄 사용 : 문제의 근본 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식

-> 별도의 접근자 메서드(Throwable의 getCause 메서드)를 통해 저수준 예외 확인 가능

예외 연쇄
```java
try {
  ...  // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
  // 저수준 예외를 고수준 예외에 실어 보낸다.
  throw new HigherLevelException(cause);
}
```

고수준 예외의 생성자는 (예외 연쇄용으로 설계된) 상위 클래스의 생성자에 '원인'을 건네주어 최종적으로 Throwable(Throwable) 생성자까지 건네지게 한다.

예외 연쇄용 생성자
```java
class HigherLevelException extends Exception {
  HigherLevelException(Throwable cause) {
    super(cause);
  }
}
```

- 대부분 표준 예외는 예외 연쇄용 생성자를 갖추고 있고, 그렇지 않은 예외라도 Throwable의 initCause 메서드를 통해 세팅 가능.
- 가능한 저수준 메서드가 반드시 성공하도록하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선 ex. 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전에 미리 검사
- 아래 계층에서의 예외를 피할 수 없다면 상위 계층에서 그 예외를 처리하여 문제를 API 호출자에까지 전파지 않는 것도 차선책(+ java.util.logging으로 기록하여 프로그래머가 로그 분석 가능)

## 아이템76. 가능한 한 실패 원자적으로 만들라

- 실패 원자적 : 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지

1. 불변 객체로 설계하는 것
2. 작업 수행에 앞서 매개변수의 유효성 검사(+ 실패할 가능성이 있는 모든 코드를 객체의 상태를 바꾸는 코드보다 앞에 배치)

```java
public Object pop() {
  if (size == 0)
    throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null;  // 다 쓴 참조 해제
  return result;
}
```

3. 객체의 임시 복사본에서 작업을 수행한 후 작업이 성공적으로 완료되면 원래 객체와 교체
4. 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌린다.(내구성이 보장되어야하는 자료구조에서 사용)

일반적으로 실패 원자성이 권장되지만 달성이 어려운 상황
- 두 스레드가 동기화 없이 같은 객체를 수정할 경우, ConcurrentModificationException을 잡아도 객체가 여전히 쓸 수 있는 상태가 아닐 수 있다.
- Error는 복구할 수 없으므로 AssertionError는 시도할 필요가 없다.
- 실패 원자성을 달성하기 위한 비용이나 복잡도가 높은 연산의 경우
- 지키지 못할 경우 실패 시의 객체 상태를 API 설명에 명시

## 아이템77. 예외를 무시하지 말라

API 설계자가 메서드 선언에 예외를 명시한 까닭은, 그 메서드를 사용할 때 적절한 조치를 요청하는 것

catch 블록을 비워두면 예외가 존재할 이유가 없어짐(화재 경보기 예시)
```java
// catch 블록을 비워두면 예외가 무시된다. -> 의심스러운 코드
try {
  ...
} catch (SomeException e) {
}
```

예외를 무시해야 할 때
- FileInputStream을 닫을 때 : 입력 전용 스트림이므로 파일의 상태를 변경하지 않았으니 복구할 것이 없고,
- 스트림을 닫는다는 건 필요한 정보는 이미 다 읽었다는 뜻이므로 작업을 중단할 이유가 없다.
  -> 파일을 닫지 못했다는 사실을 로깅
- 예외를 무시하기로 했다면 catch 블록 안에 이유를 주석으로 남기고 예외 변수도 ignored로 남겨놓는다.

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4;  // 기본 값. 어떤 지도라도 이 값이면 충분하다.
try {
  numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
  // 기본값을 사용한다.(색상 수를 최소화하면 좋지만, 필수는 아니다)
}
```

검사/비검사 예외 모두 해당되며, 무시하지 않고 바깥으로 전파되게만해도 최소한 디버깅 정보를 남길 수 있다.
