#26. 비검사 경고를 제거하라



 제네릭을 사용하면 컴파일러의 수많은 경고를 보게 된다. 비검사 형변환 경고, 비검사 메서드 호출경고, 비검사 매개변수화 가변인수 타입경고, 비검사 변환 경고 등이다. 대부분의 비검사 경고는 쉽게 제거 할 수 있다.



## 비검사 경고를 없에야 하는 이유

첫줄의 코드와 같은 코드를 작동하면 컴파일러는 올바른 타입을 제시해 주긴 한다. 아무튼 **비검사 경고를 모두 제거하는 이유는 그 코드는 타입 안정성이 보장된다는 것을 뜻하기 때문이다.** 

~~~java
// 컴파일 하면 컴파일러가 HashSet이 잘못된걸 알려준다.
Set<Lark> exaltation = new HashSet();

// 컴파일러가 알려준대로 고치지 않고 <>모양의 연산자 만으로 고칠수 있다.
Set<Lark> exaltation = new HashSet<>();
~~~



##@SuppressWarnings

 **경고를 제거할 수는 없지만 타입 안전 하다고 확신할수 있다면 @Suppress Warnings("unchecked")어노테이션을 달아 경고를 숨기면 된다.** 단 타입이 안점함을 검증해야 한다. 안전하지 않을경우 컴파일은 되지만 런타임에서는 ClassCastException을 던질 수 있다. 이럴경우 진 짜 문제가 되는 새로운 경고가 나와도 눈치 채기 힘들 수 도 있다.

 @SuppressWarning어놑이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 붙일 수 있다. **하지만 @SuppressWarnings어노테이션은 항상 가능한 한 좁은 범위에 적용하자.** 보통은 변수 선언, 아주 짧은 메서드, 혹은 생성자가 될것이며 클래스 전채에 사용할 경우 심각한 경고를 놓칠 수 도 있다. 한줄이 넘는 메서드나 생성자에 달려 있으면 지역변수 선언 쪽으로 옮기는 것이 좋다.

~~~java
public <T> T[] toArray(T[] a){
  if(a.length < size){
    // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
    @SuppressWarnings("unchecked") T[] result = 
      (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
  system.arraycopy(elements,0,a,0,size);
    if(a.length>size)
      a[size] = null;
    return a;
 }
~~~

 위 코드에서 ArrayList를 컴파일 하면 비검사 경고가 발생한다. 어노테이션은 선언문에만 달수 있기 때문에 return문에는 달수 없으며 메서드 전체에 담기에는 필요 이상으로 넓어진다. 그러므로 반환값을 담을 지역변수에 다는것이 좋다. 또한 **이 비검사 경고를 무시해도 좋은 이유를 주석으로 남겨야 한다.**

## 그 외

* 2판에서는 비검사 경고를 무점검이라고 한거 같다

* 비검사 경고란 warning : [unchecked]를 말하는데 casting할때 검사 안했다고 뜨는 경고다.