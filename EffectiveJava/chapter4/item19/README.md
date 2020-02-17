# 19. 상속을 고려해 설계하고 문서화 하라. 그러지 않았다면 상속을 금지하라.



 지난 시간, 상속을 염두해 두지 않고 설계 되었고, 문서화 해놓지도 않은 외부 클래스에 대해서 위험을 경고 했다. 여기서 설명한 상속을 염두한 설계와, 문서화에 대해서 알아본다.



---

## 상속을 고려한 문서



 메서드를 재정의 하면 어떤 일이 일어나는지를 정확히 정리하는 문서로 남겨야 한다. 한마디로, **상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.** 여기서 말하는 재정의는 public과 protected메서드중 final이 아닌 모든 메서드를 뜻한다. 덧붙여서 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다.



### 내부동작 방식을 설명하는 방법

 API문서의 메서드 설명 끝에서 종종 "Implementation Requirements"로 시작하는 절을 볼 수 있는데, 매서드의 내부 동작 방식을 설명하는 곳이다. @implSpec태그를 붙여주면 자바독 도구가 생성해준다.

java.util.AbstractCollection의 예시

![image-20200215182127217](/Users/simqi/Library/Application Support/typora-user-images/image-20200215182127217.png)

설명을 보면 iterator 메서드를 재정의 하면 remove에 영향을 줌을 알 수 있다.

 "좋은 API 문서란 어떻게가 아닌 무엇을 하는지를 설명해야한다"라는 격언과 대치된다. 상속이 캡슐화를 해치기 때문에 일어나는 현실이지만 클래스를 안전하게 상속할 수 있도록 하려면 내부 구현 방식을 설명해야 한다.



### protected

 효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 **클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected형태로 공개해야 할 수도 있다.** 



java.util.AbstractList의 예시

![image-20200215183601383](/Users/simqi/Library/Application Support/typora-user-images/image-20200215183601383.png)

 List 구현체의 최종 사용자는 removeRange 메소드에 관심이 없다. 그럼에도 메서드를 제공하는 이유는 단지 하위 클래스에서 부분리스트의 clear메소드를 고성능으로 만들기 쉽게 하기 위해서 이다.

 그렇다면 상속용 클래스를 설계할 떄 어떤 메서드를 protected로 노출해야할지 결정할 수 있을까? 안타깝게 마법은 없다. **상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다.**



### 재정의 가능 메서드와 생성자

 **상속용 클래스의 생성자는 재정의 가능 메서드를 직접적으로든, 간접적으로든 호출해서는 안된다.** 상위 클래스의 생성자가 하위 클래스으의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의 한 메서드가 하위 클래스의 생성자보다 먼저 호출 된다. 아래의 코드를 보고 이해해보자.


~~~java
public class Super {
    // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
}

public final class Sub extends Super {
    // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
    @Override public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
~~~



이 프로그램이 instance를 두번 출력 할것 같지만 첫번재는 null을 출력한다. 하위클래스의 생성자가 인스턴스 필드를 초기화 하기도 전에 overrideMe를 호출 하기 때문이다.



### Cloneable과 Serializable

 cloneable과 serializable 인터페이스는 상속에 어려움을 더해준다. 둘중 하나라도 구현한 클래스를 상속할 수 있게 설계하는 것은 일반적으로 좋지 않은 생각이다. 이 아이템들은 아이템 13(cloneable)과 86(serializable)에서 설명된다.

 clone과 readObject 메서드는 생성자와 비슷한 효과를 낸다. 새로운 객체를 만들기 때문에 이들을 구현할 때 따르는 제약도 생성좌와 비슷하다는 점에서 주의하자. **clone과 readObject 모두 직접적으로든 간접적으로든 재정의 메서드를 호출해선 안된다.**

 Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace메서드를 갖는다면 이 메서드들은 private이 아닌 prortected로 선언해야 한다. private로 선언하면 하위 클래스에서 무시되기 때문이다.



### 일반적인 구체 클래스

 일반적인 구체 클래스의 상속과 관련해서는 **상속용으로 설계되지 않은 클래스는 상속을 금지하는 것이 좋다.** 상속을 금지하는 방법은 둘중 더 쉬운 쪽은 클래스를 final로 선언하는 방법이다. 다른 하나는 모든 생성자를 private이나 package-private로 선언하고 public 정적 팩터리 매서드를 만드는 방법이다. 

  



