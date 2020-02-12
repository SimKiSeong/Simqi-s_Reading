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

 이 결과 새 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며 기존 클래스의 새로운 메서드가 추가되더라도 영향받지 않는다. 구체적인 코드를 보고 이야기 해보자

### 래퍼 클래스

우리가 아는 랩 하는 랩퍼들이 아니라 wrapper클래스 이다.

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



