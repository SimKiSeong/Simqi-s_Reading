# 14. Comparable을 구현할지 고려하라

## 

 Compareable인터페이스의 유일무이한 메서드는 compareTo이다. 두가지만 빼고 equals와 똑같지만 다른 2가지는 동치성 뿐만 아니라 순서 또한 비교할 수 있으며 제네릭하다는 것이다. compareTo를 구현한다는 것은 해당 클래스의 순서가 있다는 것이고 Arrays.sort()를 사용 할 수 있다는 뜻이 된다. 또한 검색, 극단값 계산, 자동 정렬되는 컬랙션 관리도 잘 된다.



---

## CompareTo의 일반 규약

compareTo의 일반 규약은 equals의 규약과 비슷하다.

>  이 객체와 주어진 객채의 순서를 비교할때 주어진 객체보다 작으면 음의 정수, 같으면 0 크면 양의 정수를 반환하고 비교할수 없는 경우 ClassCastExcetpion을 던진다.
>
>  sgn표기는 수학에서 말하는 부호함수를 뜻하며, 표현식의 값이 음수,0,ㅇ양수일떄 -1,0,1을 반환하도록 정의했다.
>
> 
>
> 1. sgn(x.compareTo(y)) == -sgn(y.compareTo(x)) 따라서 x.compareTo(y)가 예외를 던질때 y.compareTo(x)도 예외를 던져야 한다.
> 2. x.compareTo(y) > 0 && y.compareTo(x) > 0 일 경우 x.compareTo(z) > 0 이다. (추이성)
> 3. x.compareTo(y) == 0 일 경우 x.compareTo(z) == y.compareTo(z) 이다.
> 4. ( x.compareTo(y) == 0 ) == (x.equals(y)) 의 경우 필수는 아니지만 지키는 게 좋다. 지키지 않을경우 명시해야한다.
>
> 
>



---

## compareTo 작성 요령



compareTo메소드는 equals와 비슷하지만 몇가지 차이점만 주의하면 된다. 

* compareable은 타입을 인수로 받는 제너릭 인터페이스 이므로 인수타입을 확인하거나 형변환 할필요는 없다. 인수가 잘못되면 컴파일 자채가 안된다.
* compareTo메서드는 각 필드가 동치인지 비교하는게 아니라 순서를 비교한다. 객체참조필드를 비교하려면 compareTo를 재귀적으로 호출한다. Compareable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 Comparator를 대신 사용한다. 직접 만들거나 자바가 제공하는 것을 쓰면 된다.
* compareTo메서드에서 관계 연산자 <와 >를 사용하는 이전 방식은 거추장스럽고 오류를 발생하니 이제는 추천하지 않는다.
* 클래스에 핵심필드가 여러개라면 어느것을 먼저 비교하느냐가 중요해진다. 가장 핵심적인 필드부터 비교해 나가자.



---

## 정적 임포트 기능



자바 클래스의 static 메소드는 클래스에 대한 인스턴스의 생성없이 메소드를 사용할 수 있습니다. 예로 절대값을 구하는 java.lang.Math 클래스의 abs() 메소드는 다음과 같이 클래스명.메소드로 바로 사용한다.

~~~java
int i = Math.abs(-3);
~~~



JDK 1.5부터는 이러한 정적(static) 메소드를 더욱 쉽게 사용하기 위해서 static import 를 지원한다.

~~~java
import static java.lang.Math.abs;

int i = abs(-3);
~~~

정적 메소드를 import static 을 사용해서 import 한후에 클래스명 없이 abs(); 처럼 바로 사용할 수 있다. 주의해야 할 것은 같은 클래스 내에 동일한 이름의 메소드가 있으면 클래스 자신의 메소드가 우선한다.

참조 : https://offbyone.tistory.com/283



**이러한 정적 임포트 기능을 이용하면 정적 비교자 생성매서드 들을 그 이름만으로 사용할 수 있어 코드가 깔끔해진다.**



다음은 phoneNumber용 compareTo를 구현한 모습이다.

~~~java
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
~~~

이 메소드는 클래스를 초기화 할때 비교자 생성 메소드 2개를 이용해 비교자를 생성한다.

* comparingInt는 람다를 인수로 받아 Comparator<PhoneNumber>를 반환한다.
* 이후 비교해야할 인자들은 thenComparingInt를 통해 원하는 만큼 연달아 호출 할 수있다.
* Comparator는 수많은 보조 생성 메소들을 지원하며 숫지용 기본타입을 모두 커버한다.



---

## 잘못된 경우



이따금 값의 차를 기준으로 첫번째 값이 두번째 값보다 작으면 음수 같으면 0 첫번째 값이 크면 양수를 반환하는 compare메서드가 있다.

~~~java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
  public int compare(Object o1, Object o2){
    return o1.hashCode() - o2.hashCode();
  }
}
~~~

이럴경우 오버플로나 부동소수점 계산 방식에 따른 오류를 낼 수 있다. 아래의 2개 방식을 활용하자



1. 정적 compare메소드를 활용한 비교자

~~~java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
  public int compare(Object o1, Object o2){
    return Integer.compare(o1,hashCode(),o2.hashCode());
  }
}
~~~



2. 비교자 생성 매소드를 활용한 비교자

~~~java
static Comparator<Object> hashCodeOrder = 
  Comparator.comparingInt(o->o.hashCode());
~~~



---

## 몰랐던 것들

* **Comparable** : 객체 간의 **일반적인 정렬**이 필요할 때, Comparable 인터페이스를 확장해서 정렬의 기준을 정의하는 compareTo() 메서드를 구현한다.
* **Comparator** : 객체 간의 **특정한 정렬**이 필요할 때, Comparator 인터페이스를 확장해서 특정 기준을 정의하는 compare() 메서드를 구현한다.