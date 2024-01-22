## 아이템4: 인스턴스화를 막으려거든 private 생성자를 사용하라
### 정적 메서드와 정적 필드만을 담은 클래스의 사용처
1. Math나 Arrays 처럼 특정 주제와 관련된 메서드들을 모아 놓는 유티리티 클래스
2. Collections처럼 특정 인터페이스를 구현하는 객체 생성해주는 정적 메서드 집합(?)
3. final 클래스와 관련된 메서드 집합
### 정적 멤버만 담은 클래스
1. 유틸리티 클래스의 인스턴스화를 막기위해서 private 기본 생성자 추가
2. 상속도 불가
``` java
public class UtilityClass {
  private UtilityClass() {
    throw new AssertionError();
  }
}
```
