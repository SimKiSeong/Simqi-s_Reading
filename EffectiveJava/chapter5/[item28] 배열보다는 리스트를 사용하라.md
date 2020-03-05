#28. 배열보다는 리스트를 사용하라



## 배열과 제네릭의 차이

### 공변 불공변

 배열과 제네릭타입에는 2가지 차이가 있다. 그중 첫번째는 **배열은 공변 이고 제네릭은 불공변 이다는 것이다.** Sub가 Super의 하위 타입이라면 Sub[]는 Super[]의 하위 타입이 된다. (공변은 함께 변한다는 뜻이다.) 반면 List\<Type1\>은 List\<Type2\>의 하위타입도 상위 타입도 아니다. 언뜻보면 문제가 있는 건 리스트 같지만 실제 문제는 배열에 있다.

~~~java
// 런타임에 실패 한다.
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."
  
 // 컴파일 되지  않는다.
Object[] al = new ArrayList<Long>();
ol.addd("타입이 달라 넣을 수 없다.");
~~~

 어느쪽이든 long타입에 String을 넣을 수는 없다. 다마 배열에서는 실수를 런타임에 알게 되지만 **리스트를 사용하면 컴파일 단계에서 알수 있다.** 



### 배열의 실체화

 두번째 차이는 배열은 실체화 된다는 것이다. 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 그래서 Long배열에 String을 넣으려 하면 ArrayStoreException이 발생한다. 반면 제네릭은 타입정보가 런타임에는 소거된다.



## 제네릭 배열

 배열은 제네릭타입, 매개변수화 타입, 타입매개변수로 만들수 없다. 이렇기 떄문에 제네릭 배열을 만들수 없는데, 제네릭 배열을 생성하지 못하게 한 이유는 바로 타입 안전하지 않기 때문이다.

 배열을 제네릭으로 만들수 없어서 귀찮을 수도 있다. 제네릭 컬랙션에서는 자신의 원소타입을 담은 배열을 반환하는게 보통 불가능 하다.( 완벽하지는 않지만 이부분은 아이템 33을 통해서 해결이 가능하다. ) 또한 가변인수 메서드와 제네릭 타입을 함께 쓰면 해석하기 어려운 경고 매세지를 받는다. 이때는 @SafeVarargs애너테이션으로 대처할 수 있다.( 이것은 아이템 32에서 배운다.)

 배열로 형변환 할때 제네릭 배열 생성오류나 비검사 형변환 경고가 뜨는 것은 E[]모양의 배열대신 List\<E\>르ㄹ 사용하면 해결된다. 성능이 나빠지거나 복잡해 질수 있지만, 타입안정성과 상호운용성은 좋아진다.



## 예시

생성자에 컬랙션을 받는 Chooser클래스를 예로 살펴보자.

~~~java
public class Chooser{
  private final Object[] choiceArray;
  
  public Chooser(Collection choices){
    choiceArray = choices.toArray();
  }
  
  public Object choose(){
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
  
}
~~~



 이 클래스를 사용하려면 choose메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환 해야한다. 혹시나 다른 타입이 들어있다면 런타임 오류가 날 것이다. 이 클래스를 제네릭으로 만들면 아래와 같이 고칠수 있다.

~~~java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }

    public static void main(String[] args) {
        List<Integer> intList = List.of(1, 2, 3, 4, 5, 6);

        Chooser<Integer> chooser = new Chooser<>(intList);

        for (int i = 0; i < 10; i++) {
            Number choice = chooser.choose();
            System.out.println(choice);
        }
    }
}
~~~

이렇게 바꿀 경우 런타입 오류가 나지 않는다는 보장이 된다.