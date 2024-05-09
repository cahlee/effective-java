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
