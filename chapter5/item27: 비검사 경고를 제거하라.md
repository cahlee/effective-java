# 아이템27: 비검사 경고를 제거하라
- 비검사 경고를 제거하지 않으면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있음
## 다이아몬드 연산자 누락
``` java
Set<Lark> exaltation = new HashSet();  // 다이아몬드 연산자(자바7부터 지원) 누락
Set<Lark> exaltation = new HashSet<>();  // <Lark> 혹은 다이이몬드 연산자(<>) 추가
```
## @SuppressWarnings("unchecked")
- 경고를 제거할 수 없지만 타입 안전을 확신할 수 있는 경우
- 좁은 범위에만 적용하자(변수, 짧은 메서드, 생성자). 클래스 전체는 절대 X
- 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 함
``` java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        @SuppressWarnings("unchecked")
        T[] result = Arrays.copyOf(elements, size, a.getClass());  // return문을 지역 변수로 바꿔서 @SuppressWarnings("unchecked") 애너테이션 추가
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) {
        a[size] = null;
    }
    return a;
}
``` 
## 정리
- 할 수 있는 한 모든 비검사 경고를 제거하라
- 경고를 없앨 방법을 찾지 못하겠다면, @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨긴 후, 근거를 주석으로 남겨라
