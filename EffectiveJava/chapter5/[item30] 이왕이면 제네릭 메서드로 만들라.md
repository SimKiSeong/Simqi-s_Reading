#30. 이왕이면 제네릭 메서드로 만들라



 메서드 또한 클래스처럼 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메소드는 보통 제네릭이다. 제네릭 메서드 작성은 제네릭 타입 작성법과 비슷하다.



## 로 타입을 제네릭으로

 우선 로타입으로 된 다음 코드를 보자.

~~~java
public static Set union(Set s1, Set s2){
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
~~~

 로타입으로 된 코드는 2개의 경고가 난다. Set result = new HashSet(s1)부분과  result.addAll(s2)에서 오류가 난다. 로타입을 기억하고 있는 사람들은 알겠지만 타입 안전하지 않다. 오류를 없애려면 **입력과 반환하는 것의 타입 매개변수를 명시하고 메서드 안에서도 이 타입 매개변수만을 사용하게 수정하면 된다** 타입 매개변수 목록은 메서드의 제한자와 반환타입 사이에 온다. 타입매개변수 목록은 \<E\>설정하면 된고 반환타입은 Set\<E\>로 바꾸면 된다.

~~~java
public static <E> Set<E> union(Set<E> s1, Set<E> s2){
  Set<E> result = new HashSet(s1);
  result.addAll(s2);
  return result;
}

public static void main(String[] args){
  Set<String> guys = Set.of("톰","딕","해리");
  Set<String> stooges = Set.of("래리","모에","컬리");
  Set<String> aflCio = union(guys, stooges);
  System.out.println(aflCio);
}
~~~

이렇게 만들경우 형변환을 직접 하지 안아도 어떠한 오류나 경고 없이 컴파일 된다.



## 제네릭 싱글턴 팩터리

 때때로 불변 객체는 여러타입으로 활용할 수 있게 만들어야 할 때가 있따. 제네릭은 런타임에 타입정보가 사라짐으로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있지만 요청한 타입매개 변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이 패턴을 **제네릭 싱글턴 팩터리**라고 한다.

### 예시

 항등함수(-> 입력한 값을 그대로 반환하는 함수)를 담은 클래스를 만들고 싶다고 하자. 항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은 낭비다. 제네릭이 실체화 된다면 타입별로 항등함수를 만들어야 하지만, 소거방식을 통해 제네릭 싱글턴 하나면 된다.

~~~java
private static UnarayOperator<Object> IDENTITY_FN = (t) -> t;

@SuppresseWarnings("unchecked")
public static <T> UnarayOperatio<T> identityFunction(){
  return (UnarayOperator<T>)IDENTITY_FN;
}
~~~

 입력하는 값을 수정없이 그대로 내뱉으므로 T가 어떤 타입이든 UnarayOperator\<T\>로 형변환 하여도 안전하다.



## 재귀적 한정타입

 드물긴 하지만 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 형용범위를 한정할 수 있다. 바로 재귀적 타입 한정이라는 개념이다. 주로 Comparable과 같이 쓰인다. 

~~~java
public interface Comparable<T>{
  int compareTo(T o);
}
~~~

 여기서 T는 Comparable\<T\> 를 구현한 타입이 비교할 수 있는 원소의 타입을 정의 한다. 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다. Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그원소들을 정렬 혹은 검색하거나, 최솟 값이나 최댓값을 구하는 식으로 사용된다. 이 기능을 사용하려면 모든 원소가 상호 비교되어야 하며 아래 코드와 같이 표현할 수 있다.

~~~java
public static <E extends Comparable<E>> E max(Collection<E> c)
~~~

\<E extends Comparable\<E\>\> 는 모든 타입 E는 자신과 비교할 수 잇다 라고 읽을 수 있다. 









