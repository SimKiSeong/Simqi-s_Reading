# 13. Clone 재정의는 주의해서 진행하라

## 

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스지만, 아쉽게도 의도한 목적을 제대로 이루지 못했다. clone메소드는 Cloneable에 명시되어 있는 것이 아니라 Object메소드에 있고 Cloneable을 통해서 clone메소드를 동작하는 방식을 결정해주게 된다. 



---

## Clone메서드의 일반 규약

어떤 객체 x에 대하여 다음의 식들은 참이다.

> x.clone( ) != x
>
> x.clone.getClass( ) != x.getClass( )
>
> x.clone( ).equals(x)
>
> x.clone.getClass( ) == x.getClass( )



---

## super.clone이 아닌 생성자 호출로 구현할 때의 문제



clone메소드가 super.clone이 아닌 생성자 호출을 통해 얻은 인스턴스를 반환해도 컴파일러는 문제 없다고 하겠지만 하위 클래스에서 clone을 호출하게 된다면 하위 클래스에서도 상위 타입의 객체를 반환할 수 밖에 없다.



---

## 가변상태를 참조하지 않는 클래스용 clone메소드



제대로 동작하는 clone메소드는 먼저 super.clone을 호출한다. 불변상태의 객채라면 모든 필드가 완벽한 복재가 되었을 것이며 더이상 손볼 것이 없다.

~~~java
@Override public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // 일어날 수 없는 일이다.
        }
    }

~~~



Object의 clone메소드는 Object를 반환 하지만 형변환을 통해 PhoneNumber는 phoneNumber를 반환하게 한다. 자바는 공변반환타이핑을 지원하므로 이렇게 하는 것이 가능하고 권장하는 방식이기도 하다.



---

## 가변상태를 참조하는 클래스용 clone메소드



Stack의 경우 super.clone의 결과를 그대로 반환한다면 어떻게 될까??? 스택 인스턴스의 size는 올바른 값을 갖지만 elements필드는 원본 Stack과 똑같은 배열을 참조하게 될것이고 둘중 하나를 수정하게 된다면 다른 하나도 수정이 되어 불변식을 해치게 된다. clone메소드는 사실상 생성자와 같은 효과를 내야한다. 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.



~~~java
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
~~~



배열을 복제할 떄는 배열의 clone메소드를 사용하라고 권장되는데, 사실 배열은 clone기능을 제대로 사용하는 유일한 예다. 



---

## 복사 생성자 복사 팩터리

사실 복사 생성자와 그 변형인 복사 팩터리는 cloneable/clone 방식보다 나은 면이 많다. 언어 모순적이고 위험 천만한 객체 생성 메커니즘을 사용하지 않으며 엉성하게 문서화 된 규약ㅇ에 기대지 않고, 정상적인 final필드 용법과도 충돌하지 않으며, 불필요한 예외를 던지지 않고 형변환도 필요하지 않다.

* 복사 생성자

~~~java
public Yum(Yum yum) {...};
~~~



* 복사 팩터리

~~~java
public static Yum newInstancfe(Yum yum){...};
~~~





 새로운 인터페이스를 만들때 Cloneable을 확장해서는 안되며, 새로운 클래스도 이를 구현해서는 안된다. 단 배열만은 clone메소드 방식이 가장 깔끔한 예외이다.



---

## 몰랐던 것들

* getClass( )메소드는 객체가 속하는 class의 정보를 토해내는 매소드이다.