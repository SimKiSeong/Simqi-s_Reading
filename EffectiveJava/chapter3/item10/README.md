# 10. equals는 일반 규약을 지켜 재정의 하라



## equals를 재정의 하지 말아야 할 때

equals 메서드는 재정의 하기 쉬워보이지만 함정이 도사리고 있어 자칫하면 끔찍한 결과를 초래한다. 다음중 하나에 해당한다면 재정의 하지 않는 것이 최선이다.

1. 각 인스턴스가 본질적으로 고유하다. 
   * 값을 표현하는게 아닌 동작하는 개체를 표현하는 클래스 
   * ex) 쓰레드 -> equals는 이러한 클래스에 딱 맞게 구현되어 있다.
2. 인스턴스의 논리적 동치성을 검사할 일이 없다.
3. 상위 클래스에서 재정의한 equals가 하위클래스에도 딱 맞는다.
   * Set이나 List, Map의 경우 상위 클래스의 것을 그대로 받아서 쓴다.
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
   * package-private이란 패키지에서만 접근 할 수 있는 접근자를 뜻한다.
   * 내부 클래스로 사용하기 위해 클래스를 private로 선언할 경우 equals를 사용할 일이 없다고 한다.( -> 몰랐던 것 )
   * 혹시라도 호출될게 두렵다면 equals매소드 호출시 에러가 뜨도록 제정의 하면 된다.
5. Enum과 같이 값으로 이루어진 클래스이나 같은 값이 둘 이상 만들어 지지 않을때 재정의 할 필요가 없다.



---

## equals를 재정의 해야 할때

두 객체각 물리적으로 같은가를 판별하는 것이 아닌 논리적 동치성을 확인해야 할때 (equals가 논리적 동치성을 비교할때가 아닐때) 제정의 한다.

* Integer와 String의 경우 값을 표현하는 클래스이며 equals는 값을 비교하도록 만들어져 있다.(논리적 동치성 확인)



---

## equals를 재정의 할때 따라야 할 일반 규약

equals를 재정의 할 때는 반드시 일반 규약을 따라야 한다.

1. 반사성 : null이 아닌 모든 참조값 x에 대해, x.equals(x)는 true다.
2. 대칭성 : null이 아닌 모든 참조값 x,y에 대해 x.equals(y)가 true면, y.equals(x)도 true다.
3. 추이성 : null이 아닌 모든 참조값 x,y,z에 대해 x.equals(y)가 true고, y.equals(z)도 true면 x.equals(z)도 true다. 
4. 일관성 : null이 아닌 모든 참조값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 같은 값이 나와야 한다.
5. null아님 : null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false다.



---

## 대칭성 위배

~~~java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);

        System.out.println(list.contains(s));
    }
~~~

* s와 cis를 비교하게 되면 cis.equals는 true지만 s.equals는 false를 나타내게 됨
* List.contains은 어떤 값을 반환할지 예측 할 수 없는 상황이 생김 (지금 버전의 OpenJDK에서는 false를 반환)



---

## 추이성 위배

~~~java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}

public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    // 코드 10-3 잘못된 코드 - 추이성 위배! (57쪽)
		@Override public boolean equals(Object o) {
          if (!(o instanceof Point))
              return false;
  
          // o가 일반 Point면 색상을 무시하고 비교한다.
          if (!(o instanceof ColorPoint))
              return o.equals(this);
  
          // o가 ColorPoint면 색상까지 비교한다.
          return super.equals(o) && ((ColorPoint) o).color == color;
      }
  
  public static void main(String[] args) {
    
        // 두 번째 equals 메서드(코드 10-3)는 추이성을 위배한다. (57쪽)
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
        System.out.printf("%s %s %s%n",
                          p1.equals(p2), p2.equals(p3), p1.equals(p3));
    }
  
}
~~~

* p1.equals(p2), p2.equals(p3)의 경우 위치만 비교하여 true를 반환하지만 p1.equals(p3)는 색깔도 판별하기 때문에false를 반환
* 자칫하면 무한 루프에 빠질수도 있는 방식이다.
* 구체클래스를 확장해 새로운 값을 추가하면서 equals규약을 만족시킬 방법은 존재하지 않는다. 
  * getclass를 사용하여 instanceof를 대신할경우 같은 구체클래스만 비교할 수 있다.
  * 하위 클래스가 상위 클래스의 객채로서 활용 되지 못한다 (리스코프 치환 원칙 위배)



**리스코프 치환 원칙**

어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.

* Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용 될 수 있어야 한다.



---

## 일관성 위배

클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들어서는 안된다. 

* java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교한다. 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 알 수 있는데 결과가 항상 같다고 보장되지 않는다.
* 이러한 문제를 피하려면 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.



---

##양질의 equals를 구현하는 방법

1. ==연산자를 통해 자기 자신의 참조인지 확인
2. instanceof 연산자를 통해 입력이 올바른 타입인지 확인
3. 입력을 올바른 타입으로 형 변환
4. 입력 객체와 자기자신의 대응되는 '핵심' 필드들이 모두 일치하는지 검사



---

## 추가

* equals를 구현하였다면 대칭적인가 추이성이 있는가 일관적인가 단위테스트 해볼 것
* equals를 재정의 할땐 hashcode도 반드시 재정의 하자(아이템 11)
* 필드들의 동치성만 검사해도 equals의 규약을 어렵지 않게 지킬수 있다.
* object외의 타입을 매개변수로 받는 equals는 선언하지 말자



---

몰랐던 것들

* 논리적 동치성 : 논리적으로 같은 값인지 비교
* 정규 표현식 : 특정한 규칙을 가진 문자열의 집합을 **표현**하는 데 사용하는 형식 언어
* Package-private : 패키지 안에서만 쓸 수 있는 접근제어자
* Instanceof : 참조변수가 참조하고 있는 인스턴스의 실제 타입을 알아보기 위해 instanceof 연산자를 사용합니다. 주로 조건문에 사용되며, instanceof의 왼쪽에는 참조변수를 오른쪽에는 타입(클래스명)이 피연산자로 위치합니다. 그리고 연산의 결과로 boolean값인 true, false 중의 하나를 반환 합니다. instanceof를 이용한 연산결과로 true를 얻었다는 것은 참조변수가 검사한 타입으로 형변환이 가능하다는 것을 뜻합니다.

