# 16. 변경 가능성을 최소화 하라



불변 클래스란 인스턴스 내부 값을 수정할 수 없는 클래스다. BigInteger, BigDecimal이 이에 속하며 이 클래스들을 불변으로 설계한 데는 그만한 이유가있다. 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.



---

## 클래스를 불변으로 만드는 다섯가지 규칙

1. 객체의 상태를 변경하는 메서드를 제공하지 않는다.
2. 클래스를 확장할 수 없도록 한다.
   * 하위 클래스에서 객체의 상태를 변하게 만드는 사태를 막아준다. 상속을 막는 대표적인 방법은 클래스를 final로 선언하는 것이지만 다른 방법도 뒤에 있다고 한다.
3. 모든 필드를 final로 선언한다.
   * 시스템이 강ㅇ제하는 수단을 이용해 설계자의 의도를 명확히 들어내는 방법이라고 한다. 새ㅐ로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제 없이 동작하게끔 보장한는데도 필요하다.
4. 모든 필드를 private으로 선언한다.
   * 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다. 기술적으로는 기본 타입 필드나 불변객체를 참조하는 필드를 public final로 선언해도 불변객체가 되지만 이렇게 하면 다음 릴ㄹ리스에서 내부 표현을 바꾸지 못하므로 권하지는 않는다.
5. 자신 외에는 내부의 가변 컴포넌트에 접근 할 수 없도록 한다.
   * 클래스의 가변객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야한다. 클라이언트가 제공한 객체 참조를 가리키게 해서는 안되며, 접근자 메서드가 그 필드를 그대로 반환해서는 안된다. 생성자 접근자 readObject모두에서 방어적 복사를 수행해야한다.



---

## 예제를 통해 알아보는 불변 클래스



다음은 불면 복소수 클래스 이다.

~~~java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
~~~



### 함수형 프로그래밍

 위의 복소수 클래스에서는 사칙연산 메소드들이 존재한다. 이 사칙연사 메소드 들은 자신은 수정하지 않고 새로운 Complex인스턴스를 만들어 반환한다. 이처럼 피 연산자에 함수를 적용해 그 결과를 반환하지만, 피 연산자 자체는 그대로인 프로그래밍 패턴을 ***함수형 프로그래밍*** 이라고 한다.

 이와 달리 절차형 혹은 명령형 프로그래밍에서는 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다.

 또한 메서드 이름으로 add와 같은 동사 대신 plus같은 전치사를 사용한 점 또한 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도이다. 이 명명규칙을 따르지 않은 BigInteger와 BigDecimal클래스를 사람들이 잘못 사용해 오류가 발생하는 일이 자주 있다.



### 불변 클래스의 장점

* 불변객체는 단순하다.

* 코드에서 불변되는 영역의 비율이 높아진다.

* 생성된 시점의 상태를 파괴될 때 까지 그대로 간직한다.

* 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않아도 불변이다.

  -> 가변상태의 객체는 복잡한 상태에 놓일 수 있다. 변경자 메서드가 일으키는 상태전이를 정밀하게 문서로 남겨놓지 않은 경우 믿고 사용하기 어려울 수 있다.

* 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.

  -> 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.

* 안심하고 공유할 수 있다. 

  -> 따라서 한번 만든 인스턴스를 재활용 하기도 쉽다

  -> 가장 쉬운 재활용 방법은 자주 쓰이는 값들을 상수(public static final)로 제공하는 것이다.

* 불변 객체는 자유롭게 공유 할수 있음은 물론 불변 객체끼리는 내부 데이터를 공유 할 수 있다.

  -> BigInteger클래스에서는 내부값이 부호와 크기를 따로 표현한다.

  -> BigInteger클래스에서는 크기가 같고 부호만 반대인 negate메서드가 있는데 배열은 복사하지 않고 원본과 공유한다.

* 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.

  -> 

* 불변객체는 그 자체로 실패 원자성을 제공한다.

  * 









