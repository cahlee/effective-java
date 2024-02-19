
아이템 11
equals를 재정의하려거든 hashCode도 재정의하라.
equals를 재정의한 클래스 모두에서 hashCode도 재정의해야한다.
그렇지 않으면 hashCode 일반 규약을 어기게 되어 
해당 클래스의 인스턴스를 hashMap이나 hashSet 같은 컬렉션의 원소로 사용할떄 문제를 일으킨다.

