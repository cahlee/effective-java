## 아이템5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
1. 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식
``` java
public class SpellChecker {
  private final Lexicon dictionary;
  public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }
}
```
2. 생성자에 자원 팩터리를 넘겨주는 방식
Tile의 하위 클래스들을 생성하는 Supplier 팩터리를 넘겨주면, create 메서드에서 Tile의 하위 클래스들의 객체를 인스턴스화 할 수 있다.
``` java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```
### 결론
> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.
> 이 자원들을 클래스가 직접 만들게 해서도 안 된다. 대신 필요한 자원을(혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자.
> 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.
