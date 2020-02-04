# 16. Public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라



이따금 인스턴스 필드들을 모아 놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있다.

~~~JAVA
class Point{
  public double x;
  public double y;
}
~~~



이 경우 데이터 필드에 직접접근 할 수 있으나, 캡슐화의 이점을 제공하지 못한다.

* API를 수정하지 않고 내부 표현을 바꿀 수 없다.
* 불변식을 보장 할 수 없으며, 외부 필드에 접근할 때 부수작업을 수행 할 수도 없다.



철저한 객체지향 프로그래머는 이런 클래스를 상당히 싫어하므로 필드를 모두 private로 바꾸고 public 접근자를 추가한다.

~~~java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
~~~

 public 클래스에서라면 이 방식이 확실히 맞는데, 패키지 바깥에서 접근할 수 있는 클래스라면 접근자로 클래스 내부 표현 방식을 언제든 바꿀수 있는 유연성을 제공하게 된다.

하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출 한다 해도 하등의 문제가 없다.



public 클래스의 필드가 불변이라면 직접노출할 때의 단점이 조금 줄어들지만 결코 좋은 생각은 아니다.

* API를 변경하지 않고는 표현방식을 바꿀 수 없다.
*  필드를 읽을때 부수 작업을 수행 할 수 없다는 단점은 여전하다. 

다음 코드를 보고 생각해 보자.

~~~java
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }

}
~~~

