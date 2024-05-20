## 아이템60: 정확한 답이 필요하다면 float와 double은 피하라
float와 double 타입은 금융 관련 계산과 맞지 않다
``` java
// 코드 60-1 오류 발생! 금융 계산에 부동소수 타입을 사용했다. (356쪽)
public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```
### 결과
```
3개 구입
잔돈(달러): 0.3999999999999999
```
금융 계산에는 BigDecimal, int 혹은 long을 사용해야 한다
``` java
// 코드 60-2 BigDecimal을 사용한 해법. 속도가 느리고 쓰기 불편하다. (356쪽)
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS;
         funds.compareTo(price) >= 0;
         price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```
### 결과
```
4개 구입
잔돈(달러): 0.00
```
하지만 BigDecimal은 기본 타입보다 쓰기가 불편하고 느리다

### 해결 방법
1. int 혹은 long 타입을 사용
- 다룰 수 있는 값이 제한되고, 소수점을 직접 관리해야 한다.
2. 접근 방법의 변화. 달러를 센트로 수행
``` java
// 코드 60-3 정수 타입을 사용한 해법 (357쪽)
public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(센트): " + funds);
}
```
### 결과
```
4개 구입
잔돈(센트): 0
```
### 결론
1. 성능이 중요하고 소수점을 직접 추적할 수 있는 경우 중, 아홉자리 십진수를 표현할 수 있다면 int를, 열여덟 자리 십진수로 표현할 수 있다면 long을 사용!
2. 성능이 별로 중요하지 않거나, 열여덟자리 이상을 표현하려면 BigDecimal 사용! 
