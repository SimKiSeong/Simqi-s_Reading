# 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라



## 테그달린 클래스

 테그달린 클래스란 두가지 이상의 의미를 표현할 수 잇으며, 그중 현재 표현하는 의미를 태그값으로 알려주는 클래스이다.  다음은 원과 사각형을 표현할 수 있는 클래스다.

~~~java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
~~~

 

 태그 달린 클래스에는 단점이 한가득이다. 우선 열거타입선언, 태그필드, 스위치문 등 쓸대없는 코드가 많고, 가독성이 나쁘며 다른의미를 위한 코드로 메모리도 많이 사용한다. 필드들을 final로 선언하면 사용하지 않는 필드들도 생성자에서 초기화 하게 되는데 오류가 터져 나올 수도 있다. **테그달린 클래스는 장황하고, 오류를 내기 쉽고 비효율 적이다.**

 다행히 자바는 타입하나로 다양한 의미의 객체를 표현하는 더 나은 수단을 제공하는데 바로 클래스 계층구조를 활용하는 서브타이핑 이다. **테그달린 클래스는 클래스 계층구조를 어설프레 흉내낸 아류일 뿐이다**

 

## 태그달린 클래스를 계층 구조 클래스로

 태그달린 클래스를 계층구조로 바꾸는 방법을 알아보자.

1. 가장먼저 계층구조의 루트가 될 추상클래스를 정의하고 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상메서드로 선언한다. (위의 코드에서는 area 메서드가 해당)
2. 그런다음 태그값에 상관없이 동작이 링정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드도 루트로 올린다.
4. 다음으로 루트클래스를 확장한 구체 클래스를 의미별로 하나씩 정의 한다.



~~~java
abstract class Figure {
    abstract double area();
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}
~~~



 계층구조는 태그달린 클래스의 단점을 모두 날려버린다. 

* 쓸대없는 데이터 필드가 모두 사라지고 살아남은 필드의 모든 부분은 final이다.  
* 각 클래스 생성자가 모든 필드를 남김없이 초기화 하고 추상메서드를 모두 구현했는지는 컴파일러가 확인해준다. 
* 태그달린 클래스에서 실수로 case를 빼먹어서 생기는 오류는 이곳에서는 생길일 없다.
* 루트 클래스를 건드리지 않고 다른 개발자가 계층구조를 확장하여 사용할 수도 있다.
* 타입이 의미별로 따로 존재해서 의미를 명시하거나 제한할 수도 있고 특정의미만 매개변수로 받을 수 있다.
* 타입사이의 자연스러운 계층 관계를 반영할 수 있어 유연성과 컴파일타임 타입검사도 높아진다.

~~~java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
~~~

