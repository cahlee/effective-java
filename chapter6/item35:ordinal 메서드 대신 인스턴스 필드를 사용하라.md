# 아이템35: ordinal 메서드 대신 인스턴스 필드를 사용하라
- 열거 타입에서 몇 번째 위치인지 반환하는 ordinal 메소드
## oridnal 잘못 사용한 예
``` java
enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET,
	SEXTET, SEPTET, OCTET, NONET, DECTET;

	public int numberOfMusicians() {
		return ordinal() + 1;
	}
}
```
- 상수 선언 순서를 바꾸는 순간 numberOfMusicians 오동작
- 이미 사용 중인 정수와 값이 같은 상수는 추가할 수 없음
- 값을 중간에 비워둘 수도 없다. 굳이 쓰려면 더미 상수를 추가해야 함
## 해결책: 인스턴스 필드에 저장
``` java
// 인스턴스 필드에 정수 데이터를 저장하는 열거 타입 (222쪽)
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```
## 결론
- ordinal은 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계 됨
  - 참고
    enum 타입만을 사용하도록 만든 Map 구현체
    EnumMap 뜯어보면, 내부 데이터를 array로 저장 함
    enum은 길이가 정해져 있기 때문에, index로 데이터 저장 및 접근 가능
    그래서 ordinal을 쓰는 EnumMap은 성능이 좋음(HashMap처럼 hash를 계산할 필요도, hash값 충돌로 LinkedList 복잡도로 값을 찾을 필요도, 저장소(Node)를 resize 할 필요도 없음)
- (위와 같은 용도가 아니면) **쓰지말자!**
