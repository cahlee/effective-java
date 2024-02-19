object 명세 규약

equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 매소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.

equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다.

euqlas(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 
성능이 좋아진다.

hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두번째다. 즉, 논리적으로 같은 객체는 같은 해시 코드를 반환해야 한다.

ex) 
```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhonewNumber(707, 867, 5309), "제니");
```

이 코드 다음에 m.get(new PhoneNumber(707, 867, 5309))를 실행하면 "제니"가 아니라 null이 반환된다.
Why? 
이 코드는 두개의 인스턴스가 사용됨.
하나는 HashMap에 "제니"를 넣을때, 두번째는 이를 꺼낼때.

PhoneNumber 클래스는 hashCode를 재정의하지 않아서 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한다.
따라서 get 메소드는 엉뚱한 해시버킷에 가서 객체를 찾게됌.
두 인스턴스를 같은 버킷에 담아도 null 반환, 이유는 해시코드가 다른 앤트리끼리는 동치성 비교 조차 하지 않게 최적화 되어있기 때문.

=> PhoneNumber에 적절한 hashCode 메소드만 작성해주면 해결된다.

최악의(하지만 적법한) hashCode 구현
```java
@Override public int hashCode(){
	return 42;
}
```

좋은 해시코드 작성방법 : p68참고


PhoneNumber 클래스 예시

```java
@Override public int hashCode(){
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```

Object 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메소드인 hash를 제공한다.
단 한줄로 작성가능하지만 속도가 느려 성능에 민감하다면 지양한다.

@Override public int hashCode(){
	return Objects.hash(lineNum, prefix, areaCode);
}

만약 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보단 캐싱하는 방식을 고려한다.

해시 코드를 지연 초기화(lazy initialization)하는 hashCode 메소드 
private int hashCode; // 자동으로 0으로 초기화된다.

```java
@Override public int hashCode(){
	int result = hashCode;
	if(result == 0) {
		result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		hashCode = result;
	}
	return result;
}
```

성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.

hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

=> equals를 재정의할때 hashCode도 반드시 재정의해야한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을것이다.
재정의한 hashCode는 object의 api문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.
p68참고

https://100100e.tistory.com/352
