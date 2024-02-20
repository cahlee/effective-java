
cloneable은 복제해도 되는 클래스임을 명시하는 용도의 mixin interface 지만 
clone 메소드가 선언된 곳은 cloneable이 아닌 object 이고 protected다.
+ Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메소드를 호출 할 수 없다.

해당 객체가 clone 메소드를 제공한다는 보장이 없기 때문이다.

Cloneable 인터페이스가 하는일 = Object의 protected 메소드인 clone 동작방식을 결정.

- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며
그렇지 않은 경우는 CloneNotSupportedException을 던진다.
(이는 상당히 이례적인 경우이므로 재현 하지 않는다)

Object 명세 :


