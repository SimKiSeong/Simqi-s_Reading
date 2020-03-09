#29. 이왕이면 제네릭 타입으로 만들라



JDK가 제공하는 제네릭타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다. 그래도 배워보자.

## Stack 예제

~~~java
public class Stack{
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack(){
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e){
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop(){
    if(size == 0)
      threow new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
  }
  
  public void isEmpty(){
    return size == 0;
  }
  
  public void ensureCapacity() {
    if(elements.length == size)
      elements = Arrays.copyOf(elements,2*size+1);
  }
}
~~~

 이 클래스는 제네릭이어야 마땅하다. 제네릭으로 바꾼다고 해서 클라이언트는 스택에서 꺼낸 객체를 형변환 해야 하는데, 이떄 런타임 오류가 날 위험이 있다. 



### 제네릭 타입으로 변환

일반 클래스를 제네릭 클래스로 만드는 첫 단계는 선언타입 매개변수를 추가하는 일이다. 위의 코드는 스택이 담을 원소의 타입 하나만 추가하면 된다. 이때 타입 이름으로는 보통 E를 사용한다.(E는 타입 매개변수 중에 컬랙션 원소 타입을 지정한다.)  그런 다음에 Object를 적절한 타입 매개변수로 바꾸고 컴파일 해보자.

~~~java
public class Stack<E>{
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
    public Stack(){
    elements = new E[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(E e){
    ensureCapacity();
    elements[size++] = e;
  }
  
  public E pop(){
    if(size == 0)
      threow new EmptyStackException();
      E result = (E)elements[--size];
    elements[size] = null;
    return result;
  }
  
  public void isEmpty(){
    return size == 0;
  }
  
  public void ensureCapacity() {
    if(elements.length == size)
      elements = Arrays.copyOf(elements,2*size+1);
  }
  
}
~~~

 이 단계를 거치면 하나 이상의 오류나 경고가 뜨는데 E와 같은 실채화 불가 타입은 배열을 만들 수 없기 때문이다.  

### 해결책 1 : Object배열 활용

해결책은 2가지다. 첫번째는 Object배열을 생성한 다음 제네릭 배열로 형번환 하는 것이다. 컴파일러는 오류 대신 경고를 보낼것이다. 하지만 타입 안전하진 않다.

 컴파일러는 안전한지 알 방법이 없지만 우리는 할 수 있다. 따라서 이 비검사 형변환이 프로그램의 타입안전성을 해치지 않는 것을 우리 스스로 확인한다. Stack클래스 내에서는 배열에 저장되는 원소가 항상 E이므로 확실히 안전하다. 비검사 형변환이 안전함을 알았으면 @Suppress warning으로 해당 경고를 숨긴다.

~~~java
@SuppressWarnings("unchecked")
public Stack(){
 elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; 
}
~~~



### 해결책 2 : elements필드타입을 E[]에서 Object[]로

두번째 해결방법은 elements필드의 타입을 E[]에서 Object[]로 바꾸는 것이다. 이러면 다른 오류가 발생하는데 배열이 반환한 원소를 E로 형변환 하면 오류 대신 경고가 뜬다.

 E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. 이번에도 우리가 직접 증명하고 경고를 숨길 수 있다. 이럴경우 pop메소드에서 E로 형변환 할경우 오류가 생기는데 pop메서드 전체에서 경고를 숨기지 말고 비검사 형변환을 수행하는 할당문에서만 오류를 숨겨보자

~~~java
public E pop(){
     if(size == 0)
      threow new EmptyStackException();
  	@SuppressedWarnings("unchecked")
     E result = (E)elements[--size];
    elements[size] = null;
    return result;
}
~~~



### 장단점

 제네릭 배열 생성을 제거하는 두 방법 모두 날므의 지지를 얻는다. 첫번쨰 방법은 가독성이 좋다. 코드도 더 짧으며 코드 이곳저곳에서 배열을 자주 사용하게 된다. 첫 방식에서는 배열을 생성할 때만 형변환을 하게 되지만 두번째 방식은 원소를 읽을때 마다 처리 해줘야 한다. 단 배열의 런타임 타입이 컴파일 타입에 따라 힙 오염을 일으킬 수 있으므로 힙 오염을 걱정하는 프로그래머는 두번 째 방식을 고수하기도 한다.