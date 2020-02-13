# 18. 상속보다는 컴포지션을 사용하라



상속보다는 컴포지션을 이용하는 것이 좋다. 여기서 말하는 상속은 인터페이스 상속이 아니라 클래스가 다른 클래스를 확장하는 경우를 말한다. 릴리즈에 따라 상위 클래스가 변할 수 잇는데 그 여파로 하위 클래스가 오류가 날 수 있기 때문이다.  



---

## 상속의 잘못된 예



HashSet을 상속받아 사용하는 변형된 HashSet클래스 이다.

원소가 추가될때마다 addCount를 통해 추가된 원소의 개수를 새는 변형된 HashSet클래스에서

 메인메소드를 통해 3개의 원소를 넣고 getAddCount에서 들어간 원소만큼 개수를 출력하는 코드이다.

~~~java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
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
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
~~~

 addAll 매소드를 통해 " 틱", "탁탁", "펑" 3개의 원소를 추가하게 되면 s.getAddCount( )부분에서 3개가 반환 될거라 생각되지만 실제로는 6을 반환하게 된다. 그 원인은 HashSet의 addAll메소드가 add메소드를 사용해 구현되어 있기 때문이다. addAll메소드에서         addCount += c.size();를 통해 addCount를 추가시켜 놓은 후 HashSet의 addAll메소드를 통해서 각 원소를 add메소드를 통해 추가하게 되어 각 원소별로 addCount가 2번 실행되기 때문이다.

이럴 경우 addAll메소드를 재정의 하지 않으면 고칠수 있다. 하지만 당장은 재대로 동작할지 모르나 HashSet의 addAll이 add를 이용해 구현되었음을 가정한 해법이며 해당클래스의 내부 구현방식이다. **다음 릴리즈때 유지될지는 알 수 없다.**

 addAll을 다른식으로 재정의 할 수도 있다. 상위 메소드의 addAll을 활용하는 것이 아니라 add메소드를 활용하여 재정의 하는 것이다. 이 방식은 결과가 옳게 돌아갈 수 있기 때문에 해법으로 보이지만 **상위클래스의 메서드 동작을 다시구현하기 어렵고 자칫 오류 나거나 성능을 떨어뜨릴 수 도 있으며 하위 클래스의 private필드에 접근 할 수 없다.**  **또한 상위클래스가 다음 릴리즈에서 새로운 메서드를 추가하면 오류가 날 수 있다.** 

 위에서 메서드의 재정의 때문에 오류가 난거라 생각해서 **매서드를 재정의 하지 않고 새로 추가하는 경우를 생각 할 수있다.** 위 방식들보다 안전할 수는 있지만 안전을 보장할수 없는데 상위클래스에서 **하위클래스와 시그니처가 같고 반환타입만 다르다면 컴파일 되지 않는다.**



---

## 상속 문제를 피하는 방법 : 컴포지션



 위에 생긴 모든 문제를 피해갈 묘안이 있다. **기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private필드로 기존 클래스의 인스턴스를 참조하면 된다.** 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 **컴포지션**이라 한다. 새 클래스의 인스턴스 매서드들은 (private 필드로 참조하는)기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 **전달(fowarding)**이라 하며, 새 클래스의 메서드들은 **전달 메서드(fowarding method)**라 부른다.

 이 결과 새 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며 기존 클래스의 새로운 메서드가 추가되더라도 영향받지 않는다. 구체적인 코드를 보고 이야기 해보자. 개선된 코드는 2개의 클래스로 구현되었다.

### 래퍼 클래스

~~~java
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
~~~



### 전달 클래스

~~~java
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
~~~



 InstrumentedSet은 HashSet의 모든 기능을 정의한 Set인터페이스를 활용해 설계되어 견고하고 유연하다. 구체적으로는 Set을 인터페이스로 구현했고 Set인스턴스를 인수로 받는 생성자 하나가 있다. **임의의 Set에 계측 기능을 덧씌워 새로운 set으로만드는 것이 이 클래스의 핵심이다.**  상속방식은 구체클래스를 각각 따로 확장해야하고 각각의 생성자에 대응하는 생성자를 별도로 정의 해줘야 한다.

**다른 Set 인스턴스를 감싸고(wrap)있다**고 해서 InstrumentedSet같은 클래스를 **래퍼 클래스**라 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 **데코레이터 패턴(Decorator pattern)**이라고 한다. 컴포지션과 전달클래스의 조합은 넓은 의미로 위임이라고 부른다. (단, 엄밀히 말하면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다.)



---

## 래퍼 클래스의 단점



 래퍼 클래스는 단점이 거의 없다. 단점은 한 가지 **래퍼 클래스가 콜백 프레임워크와 어울리지 않는다**는 점만 주의하면 된다. 콜백 프레임 워크에서는 자기 자신의 ㅏㅁ조를 다른 객체에 넘겨서 다음 호출 때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 자신의 참조를 넘기고 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다.

 전달 메서드가 성능에 주는 영향이나 래퍼 객체가 메모리 사용량에 주는 영향을 걱정하는 사람도 있지만, 실전에서는 둘 다 별다른 영향이 없다고 밝혀졌다.



---

## 그 외

 **상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.** B클래스가 A와 is_a관계 일때만 클래스 A를 상속해야 한다. ( 컴포지션은 has_a관계일때 쓴다고 생각하면 된다. ) 이런 관계가 아니라면 상속해야하는 관계가 아니다고 할 수 있다. 

 자바라이브러리에서도 잘못된 클래스들이 있는데 Stack은 Vector가 아니므로 Vector를 확장해서는 안됐었다.

 컴포지션을 써야할 상황에서 상속을 사용하는건 내부구현을 불필요하게 노출하는 꼴이다. 그 결과 API가 내부구현에 묶이고 클래스 성능도 영원히 제한된다. 더 심각한 문제는 클라이언트가 노출된 내부에 직접 접근 할 수 있다는 점이다

 컴포지션대신 상속을 사용하기로 결정하기전에 마지막으로 자문해야할 질문이 있다. 확장하려는 클래스의 API에 아무런 결함이 없는가? 결함이 있다면 여러분의 클래스의 API까지 전파돼도 괜찮은가?

 **컴포지션으로는 결함을 숨기는 새로운 API를 설계 할 수 있지만, 상속은 상위 클래스의 API를 결함까지도 그대로 숭계한다.**



---

## 몰랐던 것들



계측 - 여러 방법과 장치를 이용하여 어떤 사실을 양적으로 포착하는 일로 측정보다 더 넓은 내용을 함축한 것이다. 측정 값을 어디에 쓸 것인지, 어떻게 측정할 것인지, 무슨 방법으로 계산하여 다른 값을 얻을 것인지등을 다룬다.

**[네이버 지식백과]** [계측](https://terms.naver.com/entry.nhn?docId=1061041) [instrumentation, 計測] (두산백과)



