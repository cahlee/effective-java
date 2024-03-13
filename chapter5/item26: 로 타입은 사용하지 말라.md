# 아이템26: 로 타입은 사용하지 말라
## 용어정리
|한글 용어|영문 용어|예|아이템|
|---|---|---|---|
|매개변수화 타입|parameterized type|List<String>|아이템26|
|실제 타입 매개변수|actual type parameter|String|아이템26|
|제네릭 타입|generic type|List<E>|아이템26, 29|
|정규 타입 매개변수|formal type parameter|E|아이템26|
|비한정적 와일드카드 타입|unbounded wildcard type|List<?>|아이템26|
|로 타입|raw type|List|아이템26|
|한정적 타입 매개변수|bounded type parameter|\<E extends Number>|아이템29|
|재귀적 타입 한정|recursive type bound|<T extends Comparable<T>>|아이템30|
|한정적 와일드카드 타입|bounded wildcard type|List<? extends Number>|아이템31|
|제네릭 메서드|generic method|static <E> List<E> asList(E[] a)|아이템30|
|타입 토큰|type token|String.class|아이템33|
## 제네릭 타입
- 제네릭 클래스, 제네릭 인터페이스
- 클래스와 인터페이스 선언에 타입 매개변수가 쓰인 것
- List<E>의 E는 정규 타입 매개변수
- List<String>의 String은 실제 타입 매개변수 
## 로 타입
- 제네릭 타입에서 타입 매개변수를 사용하지 않은 경우
- List<E>의 로 타입은 List
- 제네릭 도입 전 코드와 호환을 위해
``` java
// Stamp 인스턴트만 취급하기로 마음 먹은 로 타입
private final Collection stamps = ... ;
// 실수로 Coin을 넣어도 컴파일 에러가 나지 않음
stamps.add(new Coin(...));
for (Iterator i = stamps.iterator(); i.hasNext()) {
    Stamp stamp = (Stamp) i.next()  // ClassCastException 발생
}
```
- 제네릭을 사용하면 컴파일시 오류 찾을 수 있음
``` java
// Stamp 인스턴스만 넣을 수 있음
private final Collection<Stamp> stamps = ... ;
```
- List<Object> 임의 객체를 혀용하는 매개변수화 타입은 괜찮음
  - List를 받는 메서드에 List<String>은 넘길 수 있지만 List<Object>는 못 넘김
  - 제네릭의 하위 타입 규칙 때문
``` java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    // unsafeAdd의 매개변수를 List<Object>로 받으면 컴파일 에러.
    // List<String>은 List<Object의 하위 타입이 아니기 때문
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0);  // 컴파일러가 자동으로 형변환 코드를 넣어준다. ClassCastException 발생
}
private static void unsafeAdd(List list, Object o) {
    list.add(o);  
}
```
## 비한정적 와일드카드 타입
- 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때 사용
- 예) List<?>
- null 외에 어떤 원소도 넣을 수 없음.
  - 불변식 훼손 불가
``` java
// s1, s2는 비한정적 와일드카드 타입을 사용해서 중간에 불변식을 훼손할 수 없다?
private static int numElementsInCommon(Set<?> s1, Set<?> s2) {
    int result = 0;
    for (Object o : s2) {
        if (s1.contains(o)) {
            result++;
        }
    }
    return result;
}
```
## 예외
1. class 리터럴
  - List.class (O), List<String>.class (X)
2. instanceof
  - if (o instanceof Set) {...}
## 정리
- 안전하지 않은(런타임 에러가 발생하는) 로 타입을 쓰지 말자
