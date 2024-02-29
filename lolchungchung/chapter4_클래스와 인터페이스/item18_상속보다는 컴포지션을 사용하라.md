
일반적인 쿠체 클래스를 패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.

메소드 호출과 달리 상속은 캡슐화를 깨뜨린다.
	상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길수 있다.
	
예시 :

```java

public class InstrumentedHashSet<E> extends HashSet<E> {
        // 추가된 원소의 수
        private int addCount = 0;

        public InstrumentedHashSet() {
        }

        @Override
        public boolean add(E e) {
            addCount++;
            return super.add(e);
        }

        @Override
        public boolean addAll(Collection<? extends E> c) {
            addCount += c.size();
            return super.addAll(c);
        }

        public int getAddCount() {
            return addCount;
        }
    }
	
```


```java
	InstrumentedHashSet<String> languages = new InstrumentedHashSet<>();
	languages.addAll(Arrays.asList("틱", "탁탁", "펑"));
```

일반적으로 위 코드 실행 후 addCount가 3이 될 것이라 예상할 것이다. 
하지만 실제로는 6이다. 이유는 부모 클래스인 HashSet의 addAll 메서드 안에서 add메서드를 호출하기 때문이다.

HashSet의 addALL은 각 원소를 add 메소드를 호출해 추가하는데 
이때 불리는건 재정의한 메소드
addCount값이 중복으로 더해져 6이 나옴.
addAll로 추가한 원소 하나당 2씩 늘어남.

상위클래스의 메소드 동작을 다시 구현하는 방식은 어렵고 시간도 더들고 오류를 발생해 성능 저하를 일으킬수도 있다.

매소드 재정의가 원인.
새로운 메소드만 추가하는것 또한 하위 크래스에 추가한 매소드와 상위클래스에 메소드가 똑같을 경우도 있을수 있다.

결론 : 기존 클래스를 확장하는대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.
=> 컴포지션(composition; 구성) 기존 클래스가 새로운 클래스의 구성 요소로 쓰인다는 뜻.

새클래스의 인스턴스 매소드들은 기존 클래스의 대응하는 매소드를 호출해 반환.(forwarding; 전달)
새 클래스 메소드를 forwarding method(전달 메소드) 라 한다.

예시) 상속 대신 컴포지션
1) wlqgkq zmffotm 
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}

```

전달 매소드만 이뤄진 재사용 가능한 클래스
```java
/ 코드 18-3 재사용할 수 있는 전달 클래스 (118쪽)
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}

```

InstrumentedSet은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다. 구체적으로는 Set 인터페이스를 구현했고, Set의 인스턴스를 인수로 받는 생성자를 하나 제공한다. 임의의 Set에 계측 기능을 덧씌어 새로운 Set으로 만드는 것이 이 클래스의 핵심이다. 이 컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들도 함께 사용할 수 있다.

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet 같은 클래스를 래퍼 클래스라 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다. 컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 부른다.

상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다. 다르게 말하면, 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다.
클래스 A를 상속하는 클래스 B를 작성하려 한다면 "B가 정말 A인가?"를 자문하고 '그렇다'고 확신할수 없다면 B는 A를 상속하면 안된다.


컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 꼴이다. 그 결과 API가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한된다. 더 심각한 문제는 클라이언트가 노출된 내부에 직접 접근할 수 있다는 점이다.
