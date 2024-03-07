## 선요약 : 태그 달린 클래스를 작성하기보다는 태그를 없애고 계층구조로 대체하자.<br>기존에 그렇게 쓰고 있으면 계층구조로 리펙토링 하자.

#### 예시코드) 태그 달린 클래스 -클래스 게층구조보다 훨씬 나쁘다.
```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE};

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    public FigureWithTag(double radius) {
        this.shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    public FigureWithTag(double length, double width) {
        this.shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
```

### 태그 달린 클래스의 단점 

 - 열거 타입, 태그 필드, switch문 등 쓸데 없는 코드가 많다.<br>
 - 여러 구현이 섞여 있어 가독성이 나쁘다.<br>
 - 다른 의미를 위한 코드도 함께하니 메모리 비효율적이다.<br>
 - 인스턴스 타입만으로 현재 의미를 알 길이 없다.<br>

### 계층 구조로 대체하는 방법

추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언하면 된다.<br>
태그에 상관없이 동작이 일정한 메서드는 루트 클래스에 일반 메서드로 구현하면된다. <br> 
공통으로 사용하는 데이터 필드도 루트 클래스로 올린다. <br>

#### 예시코드) 태그 달린 클래스를 클래스 계층 구조로 변환
```java
public abstract class Figure {
    abstract double area();
}

public class Circle extends Figure {
    final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

public class Rectangle extends Figure {
    final double length;
    final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```
