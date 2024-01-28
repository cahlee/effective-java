# 3장. 모든 객체의 공통 메서드
Object의 final이 아닌 메서드는 일반 규약에 맞게 재정의해야 한다.
- equals, hashCode, toString, clone, finalize
## 아이템10. equals는 일반 규약을 지켜 재정의하라
### 재정의 하지 않는게 좋은 case
- 각 인스턴스가 본질적으로 고유한 경우
  - Thread
- 인스턴스의 '논리적 동치성'을 검사할 일이 없는 경우
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 경우
  - AbstractSet, AbstractList, AbstractMap
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일는 경우
``` java
@Override
public boolean equals(Object o) {
  throw new AssertionError();  // 호출 금지!
}
```
- 값이 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스
  - Enum
### 재정의가 필요한 case
- 논리적 동치성을 확인해야 하는 경우
  - 값 클래스 (Integer, String...)
### equals 메서드는 반드시 동치관계(equivalence relation)을 구현하며, 다음을 만족한다.
#### 1. 반사성 (reflexivity): null이 아닌 모든 참조 값 x에 대해 x.equals(x) 는 true이다.
- 객체는 자기 자신과 같아야 한다.


*** 
작성중
#### 2. 대칭성 (symmetry): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면 y.equals(x)도 true이다.
- **대칭성 작성 필요**
#### 3. 추이성 (transitivity): null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true이면, x.equals(z)도 true이다.
- **결론: 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**
``` java
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override
	public boolean equals(Object o) {
		if (!(o instanceof Point))
			return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}
```
#### 새로운 필드를 하위 클래스에 추가하는 경우
``` java
```
#### 리스코프 치환 원칙 위배
- 리스코프 치환 원칙: 자식 클래스는 언제나 자신의 부모 클래스를 대체할 수 있다는 원칙
- Point의 equals를 아래와 같이 구현하면 같은 구현 클래스 객체와 비교할 때만 true를 반환하므로 리스코프 치환 원칙에 위배된다.
``` java
@Override
public boolean equals(Object o) {
  if (o == null || o.getClass() != getClass())
    return false;
  Point p = (Point) o;
  return p.x == x && p.y == y;
}
...
private static final Set<Point> unitCircle = Set.of(
        new Point( 1,  0), new Point(0,  1),
        new Point(-1,  0), new Point(0, -1));

public static boolean onUnitCircle(Point p) {
  return unitCircle.contains(p);
}

public static void main(String[] args) {
  Point p1 = new Point(1, 0);
  Point p2 = new CounterPoint(1, 0);  // CounterPoint는 Point를 상속한 클래스

  System.out.println(onUnitCircle(p1));  // Point끼리 비교하니 true
  System.out.println(onUnitCircle(p2));  // o.getClass는 CounterPoint, getClass는 Point 이므로 false
}
```
- 상속 대신 컴포지션 사용
``` java

```
- 참고
  - 추상 클래스를 활용하면, equals 규약을 지키면서도 값을 추가할 수 있다. 상위 클래스를 직접 인스턴스로 만드는게 불가능 하기 때문이다. [아이템23]


#### 일관성 (consistency): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환해야 한다.
#### null 아님 : null이 아닌 모든 참조 값 x에 대하여 x.equals(null)은 false이다.

