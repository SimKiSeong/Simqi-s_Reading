#31. 제네릭과 가변인수를 함께 쓸 때는 신중하라



##제네릭과 가변인수

  가변인수메서드와 제네릭은 자바 5에 함께 추가가 되었다. 서로 잘 어우러질것 같지만 그렇지 않다. 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주지만 구현방식에 허점이 있다. 가변인수 메서드를 홏출하면 가변인수를 담기 위한 배열이 만들어지게 되는데 내부로 감춰야 할 배열을 클라이언트에 노출하기 때문이다. 그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다. 

 거의 모든 제네릭과 매개변수화 타입은 실체화 불가 타입임으로 런타임에는 타입관련 정보를 적게 가지고 있다. 메서드를 선언할때 실체화 불가 타입으로 varags매개변수를 선언하면 컴파일러가 경고를 보낸다. 다른 객체를 참조하는 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있기 때문이다.



## 예 시

~~~java
static void dangerous(List<String>... stringLists){
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;
  String s = stringLists[0].get(0);
}
~~~

 이 메서드에서는 형 변환하는 곳이 보이지 않는데도 ClassCastException을 던진다. 마지막줄에 컴파일러가 생성한(보이지 않는) 형변환이 숨어 있기 때문이다.



## 제네릭 varargs 허용 이유

 제네릭 배열을 프로그래머가 직접 생성하는 건 허용하지 않으며녀서 제네릭 varargs매개변수를 받는 메서드를 선언할 수 있게 만든 이유는 무엇일까??? 그 답은 제네릭이나 매개변수화 타입의 varargs매개변수를 받는 메서드가 실무에서 매우 유용하기 때문이다. 그래서 언어 설계자는 이 모순을 수용하기로 했다.



##SafeVarargs

 자바 7이전에는 제네릭 가변인수 메서드 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었다. 사용자는 이 경고들을 그냥 두거나 호출하는 곳마다 @SuppressWarnings("unchecked")를 달아 경고를 숨겨야 했다. 지루하며, 가독성이 떨어지고, 진짜 문제를 알리는 경고마저 숨겨 버렸다.

 자바 7에서는 @SafeVarargs에너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길수 있다. **SafeVarargs 에너테이션은 메서드 작성자가 그 메서드가 ㅌ타입 안점함을 보장하는 장치다. 메서드가 안전한게 확실하지 않다면 절대 SafeVarargs에너테이션을 달면 안된다.**

 그렇다면 안전한지는 어떻게 확신 할 수 있을까? 메서드가 varargs 배열에 아무것도 저장하지 않고 그 배열의 참조가 밖으로 노출되지 않는다면 타입안전하다. 달리 말하자면 이 **varargs매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.**



##예 시

~~~java
static <T> T[] toArray(T... args){
  return args;
}
~~~

 이 메서드는 가변인수로 넘어온 매개변수를 배열에 담아 내보내는 제네릭 메서드 이다. 얼핏 보면 편리한 유틸리티로 보이지만 위험하다. 메서드가 반환하는 타입은 컴파일 타임에 결정되는데, 그 시점에는 컴파일러에게 정보가 충분하지 않아 타입을 잘못 판단할 수 있다. 이 상태로 반환하면 힙오염을 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 나을 수 있다.



다음 메서드를 한번 보자.

~~~java
static <T> T[] pickTwo(T a, T b, T c){
  switch(ThreadLocalRandom,current().nextInt(3)){
    case 0: return toArray(a,b);
    case 1: return toArray(a,c);
    case 2: return toArray(b,c);  
  }
}

public static void main(String[] args){
  String[] attributes = pickTwo("좋은","빠른","저렴한");
}
~~~

 이 메서드는 제네릭 가변인수를 받는 toArray메서드를 호출한다는 점만 뺴면 위험하지 않고 경고도 내지 않을 것이다. 이 메서드는 항상 Object[] 타입 배열을 반환한다. pickTwo에 어떤 타입의 객체를 넘기더라도 담을 수 있는 가장 구체적인 타입이기 때문이다.

 이제 매인메소드를 실행해보면 별다른 경고없이 컴파일 된다. 하지만 실행하려 들면 ClassCastException을 던진다. pickTwo메서드의 반환값을 attributes에 저장하기 위해 String[]으로 형변환 하는 코드가 생성되는것 때문이다. 이 예는 **제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다** 는 점을 다시 상기 시키다. 단 예외는 2가지가 있다. SafeVarargs로 제대로 에노테이트된 또다른 varargs메서드에 넘기는 것은 안전하다. 다음으로 이 배열 내용의 일부 함수를 호출만하는 일반 메서드에 넘기는 것도 안전하다.

 ~~~java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists){
  List<T> result = new ArrayList<>;
  for(List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
 ~~~

위 코드는 varargs매개변수를 안전하게 사용사는 전형저거인 예다. 이 메서드는 @SafeVarargs를 써서 선언하는 쪽과 사용하는 쪽 모두에서 경고를 내진 않는다. @SafeVarargs를 사용하는 규칙은 간단하다. **제네릭과 매개변수화 타입의 varargs매개변수를 사용하는 모든 메서드에 @SafeVarargs를 다는 것이다.** 단 varargs매개변수를 사용하여 힙 오염경고가 뜨는 메서드가 있다면 점검해야한다. 메서드는 다음 두 조건을 모두 만족해야 한다.

> varargs 매개변수 배열에 아무것도 저장하지 않는다.
>
> 그 배열(혹은 복재본)을 신뢰 할 수 없는 코드에 노출하지 않는다.



## List

 @SafeVarargs어노테이션 만이 꼭 정답은 아니다. 아이템 28의 조언에 따라 varargs매개변수를 List매개변수로 바꿀수도 있다. 위에있는 flatten메서드에 적용하면 다음과 같이 된다.

~~~java
@SafeVarargs
static <T> List<T> flatten(List<List<? extends T>>... lists){
  List<T> result = new ArrayList<>;
  for(List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
~~~

 정적 팩터리 메서드인 List.of를 활용하면 다음 코드와 같이 이 매서드에 임의 개수의 인수를 넘길수 있다. 이게 가능한 이유는 List.of에도 @SafeVarargs가 붙어있기 때문이다.

 이 방식의 장점은 컴파일러가 이 메서드의 타입 안정성을 검증 할 수 있다는 점에 있다. 우리가 @SafeVarargs를 직접달지 않아도 되며 실수로 안전하다고 판단할 걱정도 없다. 다나저ㅓㅁ이라ㄹ면 클라이언트가 살짝 지저분 해지고 속도가 조금 느려질수도 있다는 점이다.

 이 방식은 pickTwo코드에도 적용할 수 있다. toArray의 List버전이 바로 List.of로, 적용하면 다음처럼 된다.

~~~java
static <T> List<T> pickTwo(T a, T b, T c){
  switch(ThreadLocalRandom,current().nextInt(3)){
    case 0: return List.of(a,b);
    case 1: return List.of(a,c);
    case 2: return List.of(b,c);  
  }
}

public static void main(String[] args){
 List<String> attributes = pickTwo("좋은","빠른","저렴한");
}
~~~

 이 코드는 배열 없이 제너릭만 사용하므로 타입안전하다.

 