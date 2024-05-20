## 아이템62. 다른 타입이 적절하다면 문자열 사용을 피하라

### 1. 문자열은 다른 값 타입을 대신하기에 적합하지 않다
- 입력받을 데이터가 진짜 문자열일 때만 문자열(String)을 사용
- 받은 데이터가 수치형이라면 -> int, float, BigInteger
- '예/아니오' 질문의 답이라면 -> boolean
- 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 작성

### 2. 문자열은 열거 타입을 대신하기에 적합하지 않다
- 상수를 열거할 때는 문자열보다 열거 타입 사용

### 3. 문자열은 혼합 타입을 대신하기에 적합하지 않다

혼합 타입을 문자열로 처리한 부적절한 예
```java
String compoundKey = className + "#" + i.next();
```
- 구분자(#)가 두 요소 중 하나에서 쓰이면 잘못된 결과
- 개별 요소로 접근하려면 파싱 필요 -> 느리고, 귀찮고, 오류 가능성
- 적절한 equals, toString, compareTo 메소드 제공 불가능
- 전용 클래스(private 정적 멤버 클래스) 생성

### 4. 문자열은 권한을 표현하기에 적합하지 않다

잘못된 예 - 문자열을 사용해 권한을 구분하였다.
```java
public class ThreadLocal {
  private ThreadLocal() { }  // 객체 생성 불가

  // 현 스레드의 값을 키로 구분해 저장한다.
  public static void set(String key, Object value);

  // (키가 가리키는) 현 스레드의 값을 반환한다.
  public static Object get(String key);
}
```

- 스레드 구분용 문자열 키가 전역 이름공간에서 공유
- 의도적으로 같은 키를 사용하여 다른 클라이언트의 값 접근 -> 보안 취약점
- 권한(capacity) : 위조할 수 없는 키 사용

Key 클래스로 권한을 구분했다.
```java
public class ThreadLocal {
  private ThreadLocal() { }  // 객체 생성 불가

  public static class Key {
    Key() { }
  }

  // 위조 불가능한 고유 키를 생성한다.
  public static Key getKey() {
    return new Key();
  }

  public static void set(Key key, Object value);
  public static Object get(Key key);
}
```

- set/get이 정적 메서드일 이유가 없음

-> 인스턴트 메서드로 변경

-> Key 자체가 스레드 지역변수가 됨

-> ThreadLocal을 지우고 Key 이름을 ThreadLocal로 변경

리팩터링하여 Key를 ThreadLocal로 변경
```java
public final class ThreadLocal {
  public ThreadLocal();
  public void set(Object value);
  public Object get();
}
````

- 실제 타입으로 형변환이 필요하여 타입 안전하지 않음(문자열, 키 둘다 마찬가지)

-> ThreadLocal을 매개변수화 타입으로 선언하여 해결

매개변수화하여 타입안전성 확보
```java
public final class ThreadLocal<T> {
  public ThreadLocal();
  public void set(T value);
  public T get();
}
```

java.lang.ThreadLocal과 흡사
- 문자열 기반 API의 문제를 해결해주고, 키 기반 API보다 빠르고 우아하다

## 아이템63. 문자열 연결은 느리니 주의하라

문자열 연결 연산자(+)로 문자열 n개를 잇는 시간은 $n^2$ 에 비례한다.
- 문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사

문자열 연결을 잘못 사용한 예 - 느리다
```java
public String statement() {
  String result = "";
  for (int i = 0; i < numItems(); i++)
    result += lineForItem(i);  // 문자열 연결
  return result;
}
```

String 대신 StringBuilder 사용

StringBuilder를 사용하면 문자열 연결 성능이 크게 개선된다.
```java
public String statement2() {
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
  for (int i = 0; i < numItems(); i++)
    b.append(lineForItem(i));
  return b.toString();
}
```

- statement 메서드의 수행 시간은 품목수의 제곱에 비례
- statement2 메서드의 수행 시간은 선형으로 증가
