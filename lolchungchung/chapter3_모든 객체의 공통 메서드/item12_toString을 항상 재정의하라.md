# Item12 : toString을 항상 재정의하라 
### 목적 : 모든 구체 클래스에서 Object의 toString을 잘! 재정의하자.

Object의 기본 toString 매소드는 단순히 '클래스 이름@16진수로_표시한_해시코드' 를 반환할 뿐이다.

간결하고 읽기 쉬운 정보를 위해선 반드시 모든 하위 클래스에서는 toString 매소드를 재정의해야한다.

실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.
이상적으로는 스스로를 완벽히 설명하는 문자열이어야 한다.

toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야한다.
포맷을 한번 명시하면 그 포맷에 얽매이게 되므로 잘 선택해야한다.

포맷을 명시하든 아니든 작성자의 의도는 명확히 밝혀야 한다.

예시1) 포맷명시 - 전화번호
```java
@Override public String toString(){
	return String.format("%03d-%03d-%04d", areaCode, prefix lineNum);
}
```

예시2) 포맷비명시 - 약물
```java
@Override public String toString(){...}
```

포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올수 있는 API를 제공하자.

(AutoValue 프레임워크는 toString도 생성해준다)

