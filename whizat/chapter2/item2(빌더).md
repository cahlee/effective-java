## 아이템2: 생성자에 매개변수가 많다며 빌더를 고려하라
- 매개변수가 많을 때 객체를 생성 하는 방법을 알아보자
### 1. 점증적 생성자 패턴
- 필수 매개변수만 필요한 생성자, 필수 매개변수 + 선택 매개변수 1개가 필요한 생성자...(늘려감)
```java
// 옷장 클래스
public class Closet {
  private int hoodie;
  private int jacket;
  private int pants;
  private int hat;
  private int accesary;
  private int ...

  public Closet(int hoodie) {
    this.hoodie = hoodie;
  }
  public Closet(int hoodie, int pants...) {
    ...
  } 
}
```
- 설정하길 원하지 않는 매개변수도 설정해 줘야 한다.
- 매개변수가 많아지면 코드를 읽기 힘들다.
- 매개변수 순서가 바뀌어도 컴파일시에는 알아채기 어렵다.

### 2. 자바 빈즈
- 매개변수 없는 생성자로 객체를 만든 후, setter 매서드를 호출해 매개변수 값을 세팅하는 방식
```java
Closet closet = new Closet();
closet.setHoodie(10);
closet.sethat(3);
...
```
- 세터 메서드를 여러개 호출해야한다.
- 객체가 완전히 생성되기 전까지는 일관성이 없다(불변이 아니다)

### 3. 빌더 패턴
- 필요한 매개변수만으로 생성자를 호출해서 빌더 객체를 얻고, 세터 메서드로 값을 설정한 후, 매개변수 없는 build 메서드를 호출해 객체를 생성한다.
``` java
public class Closet {
  private final int hoodie;
  private final int jacket;
  private final int pants;
  private final int hat;
  private final int accesary;
  private final int ...

  public static class Builder {
    private final int hoodie;
    private final int jacket;

    private int pants = 0;

    public Builder(int hoodie, int jacket) {
      this.hoodie = hoodie;
      this.jacket = jacket;
    }
    public Builder pants(int pants) {
      this.pants = pants;
      return this;
    }
    ...
    public Closet build() {
      return new Closet(this);
    }
  }

  private Closet(Builder builder) {
    this.hoodie = builder.hoodie;
    ...
  }
}
```
``` java
Closet closet = new Closet.Builder(10, 5)
                      .pants(6)
                      .hat(3)
                      .build();
```
- 읽고 쓰기 쉽다.
- 생성자와 메서드에 입력 매개변수를 검사하면 잘못된 매개변수를 일찍 발견할 수 있다.
``` java
public Builder pants(int pants) {
  if (pants < 5) {
    throw new RuntimeException("옷장에 바지를 5벌 이상 넣어주세요.");
  }
  this.pants = pants;
  return this;
}
```
#### 계층적으로 설계된 클래스를 빌더로 만들기
``` java
// 창고
public abstract class Storage {
	final Set<Integer> itemNo;
	
	abstract static class Builder<T extends Builder<T>> {
		Set<Integer> set = new HashSet<>();
		// 아이템번호
		public T itemNo(int itemNo) {
			set.add(itemNo);
			return self();  // 여기에 return this를 하면 하위 클래스에서 형변환을 해줘야 한다.
		}

		abstract Storage build();  // 공변반환 타이핑(covariant return typing)
    protected abstract T self();  // 시뮬레이트한 셀프 타입(simulated self-type) 관용구
	}
	
	Storage(Builder<?> builder) {
		itemNo = builder.set;
	}
}
```
``` java
// 신선식품 창고
public class FreshFoodStorage extends Storage {

	private final LocalDate stockDate;  // 입고 날짜
	
	public static class Builder extends Storage.Builder<Builder> {
		private final LocalDate stockDate;
		
		public Builder(LocalDate stockDate) {
			this.stockDate = stockDate;
		}

		@Override
		FreshFoodStorage build() {  // FreshFoodStorage를 반환하므로서 형변화 없이 builder를 사용할 수 있다.
			return new FreshFoodStorage(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}
	
	private FreshFoodStorage(Builder builder) {
		super(builder);
		stockDate = builder.stockDate;
	}
```
``` java
FreshFoodStorage freshFoodStorage = new FreshFoodStorage.Builder(LocalDate.now())
                                      .itemNo(1000)
                                      .build();
```
- 시뮬레이트한 셀프 타입(simulated self-type) 관용구: 하위 클래스에서 형변환 하지 않고 메서드 연쇄 지원하기 위해 사용한다.
- 공변반환 타이핑(covariant return typing): 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 타입이 아닌, 그 하위 타입을 반환하는 기능으로, 형변환 하지 않고 빌더를 사용할 수 있게 해준다.
- 가변인수 매개변수 사용: 메서드 여러번 호출하거나, (???: 적절한 메서드로 나눠 선언)

### 생각해봐야할 점
1. 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.
3. 코드 양이 많다
   - 하지만 API는 시간이 지날수록 매개변수가 많아지므로, 시작부터 빌더로 하는게 나을 때가 많다.

### 결론
매개변수가 많다면 빌더를 사용하는게 낫다
 - 점층적 생성자보다 읽고 쓰기가 간결하고, 자바빈즈보다 안전하다.
