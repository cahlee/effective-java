■명명 패턴
-----
전통적으로, 도구나 프레임워크가 특별이 다뤄야 할 프로그램 요소에는 구분되는 명명 패턴을 적용해왔다. 

예를 들어, JUnit은 버전 3까지 테스트 메서드 이름을 test로 시작하게끔 하였다. 

하지만 이러한 명명 패턴 방식은 여러 단점을 지닌다.

#### 명명 패턴의 단점
1) 오타에 민감하다.

2) 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.

[ 예를 들어 메서드가 아닌 클래스 이름을 TestSafety로 지어 JUnit에게 줘도, 테스트는 수행되지 않으며 Junit은 경고 메시지조차 출력하지 않는다. ] 

3) 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

애너테이션은, 이러한 명명 패턴의 문제들을 해결해준다.

■ 애너테이션(Annotation)
-----
Test라는 자동으로 수행되는 간단한 테스트용 애너테이션으로, 예외가 발생하면 테스트를 실패로 처리한다.

####  마커(marker) 애너테이션 타입 선언

``` java
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```
@Test 에너테이션 타입 선언 자체에 두 가지의 다른 애너테이션이 달려 있는데, 

이를 **메타 애너테이션(meta-annotation)** 이라 한다.

1) @Retention(RetentionPolicy.RUNTIME) 메타 애너테이션은 @Test가 런타임에도 유지되어야 한다는 의미이며, 만약 이를 생략하면 테스트 도구는 @Test를 인식할 수 없게 된다.

2) @Target(ElementType.METHOD) 메타 애너테이션은 @Test가 반드시 메서드 선언에만 사용되어야 한다고 알려주는 것이다. ( 클래스 선언, 필드 선언 등 다른 프로그램 요소에는 달 수 없음 )

이처럼 @Test와 같은 애너테이션을, "아무 매개변수 없이 단순한 대상에 마킹한다"라는 뜻에서 
**마커 애너테이션**이라 한다. 

즉 프로그래머가 Test이름에 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내준다.

####  마커 애너테이션을 사용한 프로그램 예시

``` java
public class Sample {
    @Test
    public static void m1() { }        // 성공해야 한다.
    public static void m2() { }
    @Test public static void m3() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m4() { }  // 테스트가 아니다.
    @Test public void m5() { }   // 잘못 사용한 예: 정적 메서드가 아니다.
    public static void m6() { }
    @Test public static void m7() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m8() { }
}
``` 


@Test 애너테이션은 Sample 클래스의 의미에 직접적인 영향을 주지 않고, 

단지 관심 있는 프로그램에게 추가 정보를 제공한다. 

즉 대상 코드의 의미는 그대로 둔 채 그 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 주는 것이다. 


#### 마커 애너테이션을 처리하는 프로그램
``` java
import java.lang.reflect.*;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
           // 클래스 판단
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
``` 


이 테스트 러너는 명령줄로부터 완전 정규화된 클래스 이름을 받아, 클래스에서 @Test 애너테이션이 달린 메서드를 찾아 차례로 호출한다. 

그리고 애너테이션을 잘못 사용해 예외가 발생한다면 오류 메세지를 출력한다.

■ 매개변수를 받는 애너테이션 타입
-----
만약 특정 예외를 던져야만 성공하는 테스트를 지원하려면, 다음과 같은 새로운 애너테이션 타입이 필요하다.

#### 매개변수 하나를 받는 애너테이션 타입

``` java
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();  // 매개변수
}
```

이 애너테이션의 매개변수 타입은 Class<? extends Throwable> 이며(한정적 타입 토큰), 

이는 **"Throwable을 확장한 클래스의 Class 객체 "** 라는 뜻이다. 

따라서 모든 예외와 오류 타입을 수용한다.

####  매개변수 하나짜리 애너테이션을 사용한 프로그램, ArithmeticException -> 0 나눌때 발생하는 exception
```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
``` 
#### 테스트 도구 수정
``` java
if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
         try {
               m.invoke(null);
               System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
         } catch (InvocationTargetException wrappedEx) {
               Throwable exc = wrappedEx.getCause();
               // 아래 부분 추가 
               Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                if (excType.isInstance(exc)) {
                     passed++;
                } else {
                     System.out.printf(
                                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                }
           } catch (Exception exc) {
                System.out.println("잘못 사용한 @ExceptionTest: " + m);
           }
}
```

앞의 코드와 한 가지 차이라면, 

이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는데 사용한다.


■  다수의 예외를 명시하는 애너테이션(1) : 배열 매개변수
-----

예외를 여러개 명시하고 그중 하나가 발생하며 성공하게 만들 수도 있다. 

@ExceptionTest 애너테이션의 매개변수 타입을 Class 객체의 배열로 수정하면 된다.

#### 배열 매개변수를 받는 애너테이션 타입
```java 
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();   // Class 객체의 배열
}
``` 

원소가 여럿인 배열일 지정할 때는 다음과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해주기만 하면 된다.

#### 배열 매개변수를 받는 애너테이션을 사용하는 코드
```java
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
public static void doublyBad() {   // 성공해야 한다.
     List<String> list = new ArrayList<>();

     // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
     // NullPointerException을 던질 수 있다.
     list.addAll(5, null);
}
```

#### 배열 매개변수를 받는 애너테이션을 처리하는 코드
```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
             m.invoke(null);
             System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
         } catch (Throwable wrappedExc) {
             Throwable exc = wrappedExc.getCause();
             int oldPassed = passed;
             // 매개 배열 체크
             Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
             for (Class<? extends Throwable> excType : excTypes) {
                 if (excType.isInstance(exc)) {
                       passed++;
                       break;
                 }
              }
              if (passed == oldPassed)
                  System.out.printf("테스트 %s 실패: %s %n", m, exc);
              }
          }
 }
```

■  다수의 예외를 명시하는 애너테이션(2) : @Repeatable
---- 

하지만 위의 코드를 더 간단하게 개선하고 싶다면, 

자바 8에서는 여러개의 값을 받는 애너테이션을, 배열 매개변수를 사용하는 대신 

**@Repeatable 메타 애너테이션** 을 다는 방식을 선택하여 코드 가독성을 높일 수 있다.

@Repeatable를 단 애너테이션은 하나의 프로그램 요소에 여러번 달 수 있다.

#### 주의 사항
1) @Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, 
@Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.

2) 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.

3) 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.(그렇지 않으면 컴파일 X)

####  반복 가능한 애너테이션 타입
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)  //컨테이너 애너테이션 class 객체
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```
#### 반복 가능한 애너테이션의 컨테이너 애너테이션
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();     // value 메서드 정의
}
```
#### 반복 가능한 애너테이션을 두 번 단 코드
```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
      List<String> list = new ArrayList<>();

      // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나 NullPointerException을 던질 수 있다.
      list.addAll(5, null);
 }
```


#### 반복 가능 애터네이션을 처리할 때 주의 사항

먼저, 애너테이션을 여러개 달면 

하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용되기 때문에 

m.isAnnotationPresent() 에서 둘을 명확히 구분하고 있는 것을 볼 수 있다.

하지만, 해당 메서드로 반복 가능 애너테이션이 달렸는지 검사한다면 검사에 실패할 것이고 (애너테이션을 여러 번 단 메서드들을 무시) 

컨테이너 애너테이션이 달렸는지만 검사하여도 

반복 가능 애너테이션을 한번만 단 메서드를 무시하고 지나치기 때문에 둘을 따로따로 확인해야 한다.

<br>

#### 반복 가능 애너테이션 다루기
``` java
if (m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
        tests++;
        try {
             m.invoke(null);
             System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (Throwable wrappedExc) {
             Throwable exc = wrappedExc.getCause();
             int oldPassed = passed;
             // repeactable exception 부분
             ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
             for (ExceptionTest excTest : excTests) {
                 if (excTest.value().isInstance(exc)) {
                     passed++;
                     break;
                 }
             }
             if (passed == oldPassed)
                 System.out.printf("테스트 %s 실패: %s %n", m, exc);
            }
      }
}
```

■ 정리 
-----
애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없으며, 

자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다. (아이템 40, 27)
