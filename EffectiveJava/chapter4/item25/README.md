# 25. 톱 레벨 클래스는 한 파일에 하나만 담으라



 소스파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않는다. 하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 한다. 이렇게 하면 한 클래스를 여러가지로 정의 할 수 있으며, 그중 어느것을 사용할지는 어느 소스파일을 먼저 컴파일 하냐에 따라 달라지기 때문이다.



## 예시

다음 소스파일은 메인클래스를 하나 담고 Utensil과 Dessert라는 2개의 다른 톱레벨 클래스를 참조한다.

~~~java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
~~~

집기(Utensil)와 디저트(Dessert) 클래스가 Utensil.java라는 한 파일에 정의 되어 있다고 해보자

~~~java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
~~~

실행하면 pancake가 출력된다.

 이제 우연히 똑같은 두 클래스를 담은 Dessert.java라는 파일을 만들었다고 해보자

~~~java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
~~~



 운 좋게 javac Main.java Dessert.java명령으로 컴파일 한다면 컴파일 오류가 나고 Utensil과 Dessert클래스를 중복 정의 했다고 알려줄 것이다.  한편 javac Main.java나 javac Main.java Utensil.java 명령으로 컴파일 하면 pancake를 출력한다. 그러나 javac Dessert.java Main.java명령으로 컴파일 하면 potpie를 출력한다. 컴파일러에 어떤 파일을 먼저 보내느냐에 따라 동작이 달라지게 된다.



### 해결책

 가장 간단한 해결책은 톱레벨 클래스를 서로 다른 소스파일로 분리 하면 그만이다. 굳이 여러 톱레벨 ㅡㄹ래스를 한 파일에 담고싶다면 정적 멤버 클래스를 사용하는 방법도 있다.

~~~java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
~~~

