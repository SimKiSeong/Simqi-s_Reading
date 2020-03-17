#31. 한정적 와일드 카드를 사용해 API유연성을 높이라



 아이템 28에서 이야기 했든 매개변수화 타입은 불공변이다. 즉, 서로다른 타입 Type1과 Type2에 대해서 List\<Type1\> 은List\<Type2\> 의 하위타입도 상위 타입도 아니다. List\<Type1\>은 List\<Type2\> 의 하위타입이 아니라는 뜻이다. List\<String\>은 List\<Object\>의 하위타입이 아니라는 것인데 List\<String\>은 List\<Object\> 가 하는 일을 제대로 수행하지 못하기 때문이다. (리스코프 치환 원칙) 하지만 떄론 불공병 방식보다 유연한 무언가가 필요하다. 



## 스택 예제

 아이템 29의 스택 예제에서 일련의 원소를 넣느느다고 해보자.

~~~java
public class Stack<E>{
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
  
  //이곳을 보자
  public void pushAll(Iterable<E> src){
    for(E e : src ){
      push(e);
    }
  }
}
~~~

이 메서드는 깨끗이 컴파일 돼지만 완벽하진 않다. Iterable src의 원소타입이 이 스택의 타입과 일치하면 잘 작동한다. 하지만 만약 Stack\<Number\> 로 선언한 후 pushAll(intVal)을 호출하면 어떻게 될까? 여기서 intVal은 Integer타입인데 Integer는 Number의 하위타입이기때문에 오류가 나지 않아야 된다고 생각하지만, 오류가 난다. 매개변수화 타입이 불공변이기 때문이다.



## 한정적 와일드 카드

 위와 같은 상황을 타개할 방법이딨다. 바로 한정적 와일드 카드라는 타입ㅇ이다. pushAll의 입력 매개변수 타입은 'E의 Iterable'이 아니라 E의 하위타입의 Iterable이어야 한다. 와일드 카드타입 Iterable<? extends E>가 정확히 이런뜹이다. (사실 extends가 어울리는 표현은 아니다.) 수정한 코드는 다음과 같다.

~~~java
  public void pushAll(Iterable<? extends E> src){
    for(E e : src ){
      push(e);
    }
  }
~~~



이제 이를 사용하는 클라이언트도 말끔히 컴파일 된다. 이제 pushAll과 짝을 이루는 popAll을 작성해보자. popAll메서드는 Stack안의 모든 원소를 주어진 컬렉션으로 옮겨 담든다. 다음과 같이 작성했다고 해보자.

~~~java
public void popAll(Collection<E> dst){
  while(!isEmpty()){
    dst.add(pop());
  }
}

// 클라이언트 코드
void exampleClient(){
  Stack<Number> numberStack = new Stack<>();
  Collection<Object> objects = ...;
  numberStack.popAll(objects);
}
~~~

 이번에도 타입이 동일하다면 문제가 없다. 하지만 Stack\<Number\> 의 원소를 Object용 컬랙션에 담는다고 해보자 모두 문제 없어 보이지만 오류가 난다.

 이 클라이언트 코드를 앞의 popAll코드와 함께 컴파일 하면 Collection\<Object\>는 Collection\<Number\> 의 하위타입이 아니다는 오류가 난다. 이번에도 와일드 카드 타입으로 해결할 수 있다. 이번에는 popAll의 입력 매개변수가 상위타입이어야 한다는 와일드 카드를 써보자. Collection<? super E>가 정확히 이런의미다.

~~~java
public void popAll(Collection<? super E> dst){
  while(!isEmpty()){
    dst.add(pop());
  }
}
~~~



## 펙스 ( PECS )

 메세지는 분명하다. **유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.** 한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드타입을 안쓰는것이 좋다.(타입을 정확히 지정해야 하므로) 다음 공식을 외우면 좋다.

> 펙스 (PECS) : producer-extends, consumer-super

 즉, 매개변수화 타입 T가 생산자라면 \<? extends T\>를 사용해야 하고, 소비자라면 \<? super T\> 를 사용하라. 앞선 Stack의 코드를 보면 이해 될것이다.

 이번엔 item30의 union메서드를 봐보자.

~~~java
// 기존코드
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
  
// 수정코드
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
~~~

 여기서 수정된 코드의 반환타입을 보면 여전히 Set\<E\> 임을 주목해보자. **반환타입에는 한정적 와일드 카드를 사용하면 안된다.** 유연성을 높여주기는 커녕 클라이언트 코드에서도 와일드 카드 타입을 사용해야 한다.

수정한 선언을 사용하면 다음코드도 말끔히 컴파일 된다.

~~~java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(1,0, 3.0, 5.0);
Set<Number> numbers = union(integers,doubles);
~~~

 제대로만 사용한다면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실조차 의식하지 못할 것이다. 받아들여야 할 매개변수르 ㄹ 받고 거절해야 할 매개변수는 거절하는 작업이 알아서 이뤄진다. **클래스 사용자가 와일드카드 타입을 신경써야한다면 그 API에 무슨 문제가 있을 가능성이 크다.** 

 하지만 위의 코드는 자바 7이전에서는 오류가 나는데 타입 추론능력이 떨어지기 때문이다. 이럴땐 컴파일러가 올바른 타입을 추론할 수 있도록 명시적 타입 인수를 사용하면 된다.

~~~java
Set<Number> numbers = Union.<Number>union(integers,doubles);
~~~



## PECS공식의 여러번 적용

 조금 복잡한 경우가 있다. item30의 7번째 코드이다.

~~~java
//원래 코드
public static <E extends Comparable<E>> E max(List<E> list)
  
// 한정적 와일드 카드를 이용해 다듬은 모습
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
~~~

 이번에는 PECS공식이 두번 사용된다. 간단한 입력 매개변수 부분부터 보면 입력 매개변수는 E인스턴스를 생산하므로 List\<E\> 를 List\< ? extends E\> 로 수정했다.

 더 난해한 타입 매개변수 E가 있다. 원래선언에서는 E가 Comparable\<E\> 를 확장한다고 정의했는데 이떄 Comparabe\<E\> 는 E인스턴스를 소비한다. 그래서 Comparable<? super E>로 대채한다. **Comparable는 언제나 소비자이므로, 일반적으로 Comparable\<E\> 보다는 Comparator\<? super E \> 를 사용하는 편이 낫다. Comparator 또한 마찬가지 이다.** 책에서 아마 가장 복잡한 메소드라고 하는데 이렇게 만들만한 가치가 있을까 라고 한다면 그렇다 이다. 다음코드는 수정된 max로만 처리할 수 있다.

~~~java
List<ScheduledFuture<?>> scheduledFutures = ...;
~~~

 수정 전 max가 이 리스트를 처리할 수 없는 이유는 ScheduledFuture가 Comparable\<ScheduledFuture\> 를 구현하지 않았기 때문이다. SchduledFuture는 Delayed의 하위 인터페이스 이고 Delayed는 Comparable\<Delayed\> 를 확장했다. ***다시말해 ScheduledFuture의 인스턴스는 다른 ScheduledFuture 인스턴스 뿐만 아니라 Delayed인스턴스와도 비교할 수 있어서 수정 전 max가 이 리스트를 거부하기 때문이다.*** 이런 일을 막기 위해서는 와일드 카드가 필요하다.



## 타입매개변수와 와일드 카드

 타입 매개변수와 와일드카드에는 공통되는 부분이 있어서 메서드를 정의할 떄 둘 중 어느것을 사용해도 괜찮을 때가 있다. swap메서드를 봐보자.

~~~java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
~~~

 public API라면 간단한 두번째가 낫다. 어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해 줄것이다. 기본규칙은 **메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체하라.** 이다. 이때 비 한정적 타입 매개변수라면 비한정적와일드 카드로, 한정적 타입 매개변수라면 하나정적 와일드카드로 바꾸면 된다.

 하지만 두번째 swap선언에는 문제가 하나 있는데 다음과 같이 직관적으로 구현한 코드가 컴파일 되지 않는다는 점이다.

~~~java
public static void swap(List<?> list, int i, int j){
  list.set(i,list.set(j,list.get(i)));
}
~~~

 이 코드를 컴파일 하면 방금 꺼낸 원소를 리스트에 다시 넣을수 없다는 오류가 난다. 이 문제를 해결하기 위해서는 와일드 타입의 실제 타입을 알려주는 메서드를 private도우미로 따로 작성하여 활용하는 방법이다. 실제 타입을 알아내려면 도우미 메서드는 제네릭메서드여야 한다.

~~~java
public static void swap(List<?> list, int i, int j){
  swapHelper(list,i,j);
}

private static<E> void swapHelper(List<E> list, int i, int j){
  list.set(i, list.set(j, list.get(i)));
}
~~~

 swapHelper 메서드는 리스트가 List\<E\>임을 알고있다. 즉 이 리스트에서 꺼낸 타입은 E이고 E타입의 값이라면 이 리스트에 넣어도 안전함을 알 고 있다. 





