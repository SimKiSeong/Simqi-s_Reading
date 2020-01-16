# 11. equals를 재정의 하려거든 hashCode도 재정의 하라



## HashMap이나 HashSet이 무엇인가?

HashMap이란 Map인터페이스의 한종류로써 Key와 Value 값으로 데이터를 저장하는 형태를 가지고 있다,

Map이란 녀석을 무엇일까요? Map이란 놈은 키(Key) , 값(Value) 을 하나의 쌍으로 묶어서 저장하는 컬렉션 클래스들을 구현하는 데 사용 된다. 쉽게 말해 key, value 값으로 저장하는 List 형태의 조상이라고 생각 하시면 된다.

**HashMap이나 HashSet에서 객채를 같은지 비교하는 방법**

![img](https://i.imgur.com/dShPCEh.png)

객채에 equals를 하기 전에 hash코드로 비교해서 다를경우 equals를 하지 않기 때문에 논리적으로 같은지 확인 하기 위해서는 equals를 재정의 해야 한다.



---

## Object의 API문서에 기술된 규약(중 일부)

이 규약을 지키도록 hashCode를 재정의 해야한다.

* equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇번을 호출해도 일관되게 항상 같은 값응ㄹ 반환해야 한다. 단, 어플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
* equals가 두 객체를 같다고 판단 했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
* equals가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필욘는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.



---

## HashCode 구현법

1. 최악의 hashCode구현 (사용금지 -> 바로위 규약중 3번째 위배 )

   ~~~java
   @Overrride public int hashCode(){return 42;}
   ~~~

   * 모든 객체에서 똑같은 해시코드를 반환하니 적법한것처럼 보이지만 모든 객체에게 똑같은 값만 반환하므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트 처럼 동작한다.
   * 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다. 이상적인 해시 함수는 서로다른 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

   

2. 좋은 hashCode를 작성하는 간단한 요령

   1. int변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다.

   2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.

      a. 해당 필드의 해시코드 c를 계산한다.

      	1. 기본 타입 필드라면, Type.hashCode(f)를 수해ㅐㅇ한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.
       	2. 참조 타입필드면서 이 클래스의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산적으로 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다.(다른 상수도 가능하지만 전통적으로 0을 사용한다.)
       	3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심원소의 해시코드를 계산한다음 2.b 방식으로 갱신한다. 배열에 핵심원소가 하나도 없다면 단순히 상수(0이 추천된다.)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.

      b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.

~~~java
result = 31 * result + c;
~~~

​		3. result를 반환한다.



3. Object클래스가 제공하는 hashCode 함수
   * 2번과 비슷한 수준의 hashCode함수를 단 한줄로 작성할 수 있다.
   * 속도가 조금 더 느리다.
     * 입력 인수를 담기위한 배열, 박싱 언박싱으로 인한
   * 성능이 민감하지 않을땐 ok

~~~java
    @Override public int hashCode() {
        return Objects.hash(lineNum, prefix, areaCode);
    }
~~~



---

## 2번 방식으로 구현된 hashCode 메소드

~~~java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix   = rangeCheck(prefix,   999, "prefix");
        this.lineNum  = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    // 전형적인 hashCode 메서드
    @Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
}
~~~



___

## 지연 초기화 하는 hashCode

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다. 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어 질 때 해시코드를 계산해 둬야한다. 해시의 키로 사용되지 않을 경우 hashCode가 처음 불릴때 계산하는 지연초기화 전략이 있다.

~~~java
   // 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다.
   private int hashCode; // 자동으로 0으로 초기화된다.

    @Override public int hashCode() {
       int result = hashCode;
       if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
~~~



---

## 그 외

* 성능을 높이기 위해 핵심필드를 생략하면 안된다. -> 해시 품질 저하
* Hashcode가 반환하는 값의 생성규칙을 API사용자에게 자세히 공표하지 말자. -> 클라이언트가 이 값에 의지 하지 않고 추후에 계산 방식을 바꿀수도 있다.



---

참고

* https://minwan1.github.io/2018/07/03/2018-07-03-equals,hashcode/
* 