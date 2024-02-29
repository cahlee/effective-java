
불변 클래스란 ? 그 인스턴스의 내부 값을 수정할 수 없는 클래스
불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.

불변 클래스의 종류 : String, BigInteger, BigDecimal

클래스를 불변으로 만들기 위한 다섯가지 규칙

1. 객체의 상태를 변경하는 메소드(변경자)를 제공하지 않는다.

2. 클래스를 확장할 수 없도록 한다. (대표적으로 클래스를 final로 선언)

3. 모든 필드를 final로 선언한다.

4. 모든 필드를 private으로 선언한다.
	필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아줌. 
	기술적으로 기본 타입필드나 불변 객체를 참조하는 필드를 public final로만 선언해도 불변 객체가 되지만, 
	이렇게 하면 내부 표현을 바꾸지 못하므로 권장하지 않는다. (item 15, 16)  
	
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
   
	클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야한다.
	
예시) 불변 복소스 클래스
```java
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}


```

실수부와 허수부, 
접근자 메소드(realPart, imaginaryPart)
사칙연산(plus, minus, times, divideBy)

	피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라고 한다    
	절차적 혹은 명령형 프로그래밍에서는 메소드에서 피 연산자인 자신을 수정해 자신의 상태가 변하게 된다.  
	메소드이름으로 동사대신 전치사를 사용, 해당 메소드가 객체의 값을 변경하지 않는다는 사실을 강조(add -> plus)    
	
이점 : 

- 코드에서 불변이 되는 영역의 비율이 높아진다.   
- 불변객체는 생성된 시점의 상태를 파괴될때까지 그대로 간직해서 단순하다.  
- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.
- 여러스레드가 동시에 사용해도 절대 훼손되지 않는다. 다른 스레드에 영향을 줄 수 없어서 안심하고 공유 할 수 있다.
	- 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용 하기 권한다. => 재활용 방법 : 자주 쓰이는 값들을 상수(public static final)로 제공.
	
ex) 
```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```


불변 클래스는자주 사용되는 인스턴스를 캐싱하여 중복 생성 안하도록 정적 팩터리를 제공할 수 있다.  
자유롭게 공유한다는 뜻 = 방어적 복사(item50)도 필요없다.     
원본과 똑같기 때문이다.     
so, 불변 클래스는 clone 메소드나 복사 생성자를 제공하지 않는게 좋음.   

불변 객체는 자유롭게 공유할수 있고 불변 객체 끼리 내부 데이터 공유가 가능하다.   
ex) BigInteger 클래서는 내부에서 값의 부호(int 변수)와 크기(int 배열 )를 따로 표현.   
negate 메소드는 크기는 값고 부호만 반대이나 원본 인스턴스와 공유해도 되서 내부 배열을 그대로 가리킨다.   

- 객체를 만들때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.   

- 불변 객체는 그 자체로 실패 원자성을 제공한다.(item76)  
	실패 원자성(failure atomicty) : 메소드에서 에외가 발생한 후에도 그 객체는 여전히(메소드 호출 전과 똑같은) 유효한 상태여야 한다.

단점 : 값이 다르면 반드시 독립된 객체로 만들어야 한다.   
	ex) 백만 비트짜리 BigInteger에서 비트 하나 바꾸게 되면 백만 비트짜리 새로운 인스턴스를 생성.   
 ```java
		BigInteger moby = ...;
		moby = moby.flipBit(0);
```		
Bitset클래스 에서 원하는 비트 하나만 상수 시간안에 바꿈   

  ```java
		BitSet moby = ...;
		moby.flip(0);
```
		
성능 문제에 대해서    
다단계 연산들을 예측하여 기본 기능으로 제공한다.   
다단계 연산을 기본으로 제공하면 각 단계마다 객체 생성을 하지 않아도 된다. 내부적으로 구현 가능.   
ex) BigInteger는 모둘러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스(companion class) 를 package-prviate로 두고있다.   

그렇지 않다면 public으로 구현하자. 이의 예시가 String 클래스 (가변동반 클래스는 StringBuilder(StringBuffer))
	
 불변 클래스 만드는 또다른 설계 방법.  
 불변임을 보장하기 위해 자신을 상속하지 못하게 하도록 final 클래스로 선언하는거지만 더 유연한  
	모든 생성자를 private 혹은 package-private로 만들고 public 정적 팩터리를 제공하는 방법(item1)  
	
EX) BigIteger와 BigDecimal을 설계할 당시엔 불변객체가 사실상 final 이어야 한다는 생각이 널리 퍼지지지 않아서 
두 클래스 메소드 모두 재정의가 가능하고 하위 호환성이 발목을잡는다. 그래서 주의해야 한다.   
이 값들이 불변이어야 클래스의 보안을 지킬 수 있다면 인수로 받은 객체가 진짜 BigInteger인지 반드시 확인해야한다.   
	신뢰할수 없는 하위 클래스의 인스턴스라고 확인되면 가변이라 가정하고 방어적 복사 사용(item50)   

	
### 결론 : 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.  
but, 모든 클래스를 불변으로 만들 수는 없다. 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.   
다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.   
생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.  
