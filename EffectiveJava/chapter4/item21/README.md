# 21. 인터페이스는 구현하는 쪽을 생각해 설계 하라.



자바 8 전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 방법이 없었다. 디폴트 메서드를 통해서 이게 해결되었다곤 하지만 위험이 완전히 사라진 것은 아니다. 자바 8에서는 핵심 컬랙션 인터페이스들에 다수의 디폴트 메소드가 추가되었다. 대부분 잘 작동하지만 **생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어렵다.** 



---

## 예 시



자바 8의 Collection인터페이스에 추가된 removeIf 메서드를 보면 범용적으로 구현되어있다.

~~~java
default boolean removeIf(Predicate<? super E> filter){
  Objects.requireNonNull(filter);
  boolean result = false;
  for(Iterator<E> it = iterator(); it.hasNext();){
    if(filter.test(it.next())){
      it.remove();
      result = true;
    }
  }
  return result;
}
~~~



이 코드보다 더 범용적이게 구현하기 어렵겠지만, 현존하는 모든 Collection구현체와 잘 어우러지는 것은 아니다. 대표적인 예가 apache.commons.collections4.collection.SynchronizedCollection이다. SynchronizedCollection은 활발히 관리되고 있지만 removeIf기능을 재정의 하지 않고 있다. 락 부분과 관련해서 오류가 난다고 한다.

 

### 이와 관련해서 인터페이스 설계시 주의해야 할 사항.

* 디폴트 메서드는 기존 구현체에 런타임 오류를 일으킬 수 있다.
* 디폴트 메서드가 생겼지만 인터페이스를 할때는 주의를 기울어야 한다.
* 인터페이스를 릴리즈 한 후에도 결함을 수정 가능 할 수도 있지만, 가능성이 기대서는 안된다.