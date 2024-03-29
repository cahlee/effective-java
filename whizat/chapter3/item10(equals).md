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
#### 2. 대칭성 (symmetry): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면 y.equals(x)도 true이다.
- 두 객체의 서로에 대한 동치 여부에 똑같이 대답해야 한다.
``` java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    // 대칭성 위배
    @Override
    public boolean equals(Object o) {
	if (o instanceof CaseInsensitiveString)
	    return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
	if (o instanceof String)  // 한 방향으로만 작동한다!
	    return s.equalsIgnoreCase((String) o);
	return false;
    }

    public static void main(String[] args) {
        // cis.equals(s) -> true
        // s.equals(cis) -> false => 대칭성 위배
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);

        System.out.println(list.contains(s));
    }
    ...
    // 수정한 equals 메소드 => CaseInsensitiveString의 equals를 String과 연동할 수 없다.
    // 해당 케이스는 대칭성을 만족 시킬 수 없음
    @Override 
    public boolean equals(Object o) {
	return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
}
```
#### 3. 추이성 (transitivity): null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true이면, x.equals(z)도 true이다.
- **구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**
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
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ...
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
    public static void main(String[] args) {
        // Point의 equals는 색상을 무시하고, 좌표만을 가지고 판단하여 true
        // ColorPoint의 equals는 매개변수 클래스 종류가 달라서 매번 false
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp) + " " + cp.equals(p));
    }
    ...
}
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
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    public static void main(String[] args) {
        ColorPoint cp1 = new ColorPoint(1, 2, Color.RED);
        ColorPoint cp2 = new ColorPoint(1, 2, Color.RED);
        System.out.println(cp1.equals(cp2));
    }
}
```
- 참고
  - 추상 클래스를 활용하면, equals 규약을 지키면서도 값을 추가할 수 있다. 상위 클래스를 직접 인스턴스로 만드는게 불가능 하기 때문이다. [아이템23]
#### 일관성 (consistency): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환해야 한다.
- 가변 클래스의 객체는 비교 시점에 따라 같을수도 다를수도 있다.
- 불변 클래스의 객체는 한번 같으면 영원히 같고, 한번 다르면 영원히 다르다고 답하도록 만들어야 한다.
- equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.
- equals는 항시 메모리에 존대하는 객체만을 상요한 결정적(deterministic) 계산만 수행해야 한다.

#### null 아님 : null이 아닌 모든 참조 값 x에 대하여 x.equals(null)은 false이다.
- o.equals(null)은 false
- NullPointException을 던지면 안된다.
- 
``` java
@Override
public boolean equals(Object o) {
if (!(o instanceof MyType))
    return false;
...
}
```
### equals 구현 방법 단계별 정리
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
- float와 double을 제외한 기본 타입 필드는 == 연산자로 비교하고, 참조 타입 필드는 각각의 equals  메서드로, float와 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compareTo(double, doulbe)로 비교한다. Float.equals와 Double.equals 메서드는 오토박싱을 수반할 수 있어 성능상 좋지 않다.
- 때로는 null도 정상 값으로 취급하는 참조 타입 필드도 있는데, 정적 메서드인 Object.equals(Object, Object)로 비교해서 NullPointerException을 예방해야한다.
- 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자. 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다. 하지만 파생 필드가 객체 전체의 상태를 대표하는 경우 파생 필드를 비교하는 쪽이 더 빠를 때도 있다.
- equals를 재정의할 땐 hashCode도 반드시 재정의하자[아이템11]
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
- AutoValue 프레임워크를 사용하면 equals와 hashCode를 작성하고 테스트해준다.
``` java
@AutoValue
public abstract class AutoValueMoneyWithBuilder {
    public abstract String getCurrency();
    public abstract long getAmount();
    static Builder builder() {
        return new AutoValue_AutoValueMoneyWithBuilder.Builder();
    }
    
    @AutoValue.Builder
    abstract static class Builder {
        abstract Builder setCurrency(String currency);
        abstract Builder setAmount(long amount);
        abstract AutoValueMoneyWithBuilder build();
    }
}
```
