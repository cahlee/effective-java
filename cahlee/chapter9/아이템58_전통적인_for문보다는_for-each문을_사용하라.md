## 아이템 58: 전통적인 for문보다는 for-each문을 사용하라
### 전통적인 for문
``` java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    // e로 무언가를 한다.
}
for (int i = 0; i < a.length; i++) {
   // a[i]로 무언가를 한다.
}
```
while문 보다는 낫지만 좋은 방법은 아님
1. 관심 외의 변수가 증가함(반복자, 인덱스)
2. 잘못된 변수를 사용했을 때 컴파일러가 잡지 못할 수 있음
3. 컬렉션이나 배열이냐에 따라 코드 형태가 달라짐

### 전통적인 for문의 버그 예1
``` java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

Collection<Face> faces = EnumSet.allOf(Face.class);

for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
    for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
        System.out.println(i.next() + " " + j.next());
```
- 주사위를 두번 굴렸을 때 나오는 모든 경우의 수를 출력하는 프로그램
- 예외를 던지지는 않지만 "ONE ONE" 부터 "SIX SIX"까지 여섯 쌍만 출력(36개 조합이 나와야 함)

### 전통적인 for문의 버그 예2
``` java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
    NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next()));
```
- i.next()가 Suit당 한 번씩만 불려야 하는데, Rank 하나당 한번씩 불리고 있음
- Suit의 숫자가 바닥나면 NoSuchElementException을 던짐

### 전통적인 for문을 사용한 문제해결
``` java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(suit, j.next()));
}
```
바깥 반복문에 변수를 추가하여 해결(더 나은 방법이 있음)

### for-each 문
``` java
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```
- 컬렉션과 배열 모두 속도는 전통적인 for문과 같음
- Iterable 인터페이스를 구현한 객체라면 모두 for-each문을 사용하여 순회 가능

### for-each문을 사용할 수 없는 경우
1. 파괴적인 필터링
   - 컬렉션을 순회하면서 선택된 원소를 제거해야 할 경우
   - 자바 8부터 Collection의 removeIf 메서드를 사용하여 원소를 제거할 경우 순회하지 않아도 됨
2. 변형
   - 순회하면서 원소 값 일부나 전체를 교체해야 할 경우
   - 인덱스로 접근해서 특정 원소를 변경시킬 수 없음
3. 병렬 반복
   - 여러 컬렉션을 병렬로 순회해야 할 경우
