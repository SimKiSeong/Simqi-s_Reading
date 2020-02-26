# 20. 추상클래스보다 인터페이스를 우선하라



 자바가 제공하는 다중구현 메커니즘은 인터페이스와 추상클래스만 존재한다. 자바 8부터 인터페이스도 디폴트 메서드를 제공할 수 있게 되어 두 매커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다. **둘의 가장 큰 차이는 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다.** 반면 인터페이스가 선언한 메서드를 모두 정의하고 그 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.



---

## 인터페이스의 장점



### 기존 클래스에 끼워 넣기

 **인터페이스는 기존 클래스에 끼워넣기 좋다.** 인터페이스가 요구하는 메소드를 추가하고 클래스 선언에 implements구문만 추가하면 끝이다. 자바에서 Comparable, Iterable, AutoCloseable인터페이스가 추가됐을때 표준 라이브러리의 수많은 기존 클래스가 이 인터페이스들을 구현한채 릴리스 됐다. 반면 **추상 클래스는 기존클래스에 끼워넣기는 어려운게 일반적이다.** 두 클래스가 같은 추상클래스를 확장하길 원한다면, 추상클래스는 계층 구조상 두 클래스의 공통 조상이어야 한다. 



### 믹스인(mixin)정의에 안성맞춤이다.

믹스인은 클래스가 자신의 "본래타입"에 추가하여 구현할 수 있는 타입으로써, 선택 가능한 기능을 제공하며, 그 기능을 제공 받고자 하는 클래스에서 선언한다. 예를 들어 Compareable은 믹스인 인터페이스 이다. 상호 비교 가능한 다른 객체와의 비교를 통해 클래스의 인터페이스를 정렬가능하다는 것을 그 클래스에 선언 할 수 있다. mixin은 이처럼 어떤 타입의 본래 기능에 선택 가능한 것을 섞는다는 의미이다. 추상 클래스로는 믹스인을 정의 할 수 없는데 기존 클래스에 덧씌울수 없기 때문이다.



### 계층구조가 없는 타입 프레임워크

 타입을 계층적으로 정의하면 많은 개념을 구조적으로 잘 표현할 수 있지만, 계층을 엄격하게 구분하기 어려운 개념도 있다.

~~~java
public interface Singer{
  AudioClip sing(Song s);
}

public interface Songwriter{
  Song compose(int chartPosition);
}

public interface SingerSongwriter extends Singer, Songwriter{
  AudioClip strum();
  void actSensitive();
}
~~~

아이유처럼 작곡도 하는 가수가 제법있다. 이 코드처럼 타입을 인터페이스로 정의하면 가수 클래스가 Singer와 Songwritwer모두를 구현해도 문제가 안된다. 심지어 모두를 확장하고 세로운 메서드를 추가한 제 3의 인터페이스도 정의할 수도 있다. 

같은 구조를 클래스로 만들려면 가능한 조합을 전부 각각의 클래스로 정의한 고도비만 계층구조가 만들어진다. 속성이 n개라면 2의 n승개의 계층 구조가 만들어지게 되며 조합폭발이라는 현상이 이러난다.



### 강력하고 안전

 래퍼클래스 관용구(아이템 18 참고) 와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이된다. 타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속 뿐이다. 상속해서 만든 클래스는 래퍼클래스보다 활용도가 떨어지고 깨지기 쉽다.



---

## 디폴트 메서드



 인터페이스의 메서드중 구현방법이 명백한 것이 있따면, 그 구현을 디폴트 메서드로 제공해 프로그래머들의 일감을 덜어 줄 수있다. 이 기법은 21번 아이템에 있는 removeIf코드를 보면 된다. 단 제공할때 상속하려는 사람을 위한 설명을 @implSpec를 통해 문서화 하자

 디폴트 메서드에도 제약은 있다. equals hashcode 같은 object메서드는 제공해서 안된다. 또한 인터페이스는 인스턴스 필드를 가질 수 없고 public이 아닌 정적 맴버도 가질수 없다.(단 private정적 메서드는 예외이다.) 마지막으로 여러분이 만들지 않은 인터페이스에는 디폴트 메서드를 추가 할 수 없다.



---

## 골격 구현 클래스

 

  인터페이스와 추상 골격 구현 클래스와 함께 제공하는 식으로 인터페이스와 추상클래스의 장점을 모두 취하는 방법도 있다. 인터페이스로 타입을 정의하고 필요하면 디폴트 메서드 몇게도 함께 재공한다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다. 이렇게 해두면 단순히 골격 구현을 확장하는 것만으로도 이 인터페이스를 구현하는데 필요한 일이 대부분 완료된다.

 관례상 인터페이스 이름이 Interface라면 골격 구현 클래스의 이름은 AbstractInterface로 짓는다. Skeletal이 더 어울릴지 모르지만 Abstract가 붙는 것이 확고히 자리를 잡았다. 제대로 설계했다면 골격 구현은 그 인터페이스 나름의 구현을 만들려는 프로그래머의 일을 상당히 덜어준다.

골격구현을 사용해 완성한 구체 클래스

 ~~~java
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // 오토박싱(아이템 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // 오토언박싱
                return oldVal;  // 오토박싱
            }

            @Override public int size() {
                return a.length;
            }
        };
    }

    public static void main(String[] args) {
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++)
            a[i] = i;

        List<Integer> list = intArrayAsList(a);
        Collections.shuffle(list);
        System.out.println(list);
    }
}
 ~~~



골격클래스의 아름다움은 추상클래스처럼 구현을 도와주는 동시에 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유로움에 있다. 골격구현을 확장하므로 추상클래스를 구현할 수 있지만, 상황이 여의치 못하면 인터페이스를 직접 구현해야한다.

시뮬레이티드 다중상속

인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고 각 메서드의 호출은 내부 클래스의 인스턴스에 전달하는 방법(아이템 18의 래퍼클래스 비슷) 이 있는데, 다중상속의 많은 장점을 제공하는 동시에 단점은 피하게 해준다.



골격구현 작성은 인터페이스를 잘 살펴 다른 메서드르의 구현에 상용되는 기반 메서드를 선정한다. ( 이 메서드들은 골격구현에서 추상 메소드가 될것이다.) 그 다음으로 기반 메서드를 사용해 사용자가 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다. 단 equals, hashCode, Object와 같은 애들은 디폴트로 제공하면 안된다.



추상 골격 구현 클래스

~~~java
public abstract class AbstractMapEntry<K,V>
        implements Map.Entry<K,V> {
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(),   getKey())
                && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override public int hashCode() {
        return Objects.hashCode(getKey())
                ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
~~~





---

## 그 외



골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 상속할때 해야하는 일들을 모두 잘 해야한다.



---

## 이해하기 위해 필요했던 것들



### 추상클래스

특징이 비슷한 클래스들의 특징을 모아 상속하게 만들어 쓰게 하는 클래스

추상클래스의 장점 -> 인터페이스에 메서드를 추가하면 오류가 나는데 추상클래스는 매서드를 추가할 수 있다. 하지만 이것두 옛말이 되어 버렸다. default를 사용하게 되면 인터페이스도 메서드를 추가 할 수 있다.