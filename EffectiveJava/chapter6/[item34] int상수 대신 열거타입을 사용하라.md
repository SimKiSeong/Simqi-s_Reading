#24. int상수 대신 열거타입을 사용하라



###열거 패턴

 열거 타입은 일정개수의 상수값을 정의한 다음 그 외의 값은 허용하지 않는 타입이다. 대표적인 예로 정수 열거가 있는데 단점이 많다. 타입 안전을 보장할 방법이 없으며 표현력좋지 못하기 때문이다.

~~~java
public static final int APPLE_FUJI;
public static final int APPLE_PIPPIN;
public static final int APPLE_GRANNY_SMITH;
~~~



 이러한 정수 열거 패턴은 깨지기 쉬운데 평법한 상수를 나열한것 뿐이라 그 값이 클라이언트 파일에 그대로 새겨지기 때문이다. 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야한다. 또한 문자로 출력하기도 까다로우며 출력하거나 디버깅 할때 숫자로만 보여 도움이 되지 않는다. 



### 열거 타입

 이러한 문재점을 해결하기 위해 자바는 열거 타입을 재안하는데 C,C++,C#과는 다른 차이를 보인다. 자바의 열거는 완전한 클래스이기 때문이다. 열거 타입 자채는 클래스이며 상수 하나당 public static final 필드로 공개하기 때문에 인스턴스가 확장할 수 없고 딱 하나만 존재하는것이 보장된다.

~~~java
public enum Apple {FUJI,PIPPIN,GRANNY_SMITH}
~~~

 열거 타입은 컴파일 타입 안정성을 제공하는데 위 코드의 열거 타입을 매개변수로 받는 메서드를 선언했다면 참조는 Apple의 3가지 값중 하나임이 확실하다. 다른 타입의 경우 컴파일 오류가 나기 때문이다.

 열거 타입은 임의의 인터페이스를 구현 할 수 잇게 되어 있다. Object메서드들을 높은 품질로 구현해 놨으며 Comparable과 Serializable을 구현했고 직렬화 형태도 웬만큼 변형을 가해도 문제 없이 동작하게 끔 구현되어 있다. 그렇기에 추상 개념을 완벽히 표현해낼 수도 있다.

 태양계의 8개의 행성은 열거타입을 설명 하기 좋은 예이다.

~~~java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
~~~



### 상수별 메서드 구현

 상수별 메서드를 구현하게 되면 상수끼리 코드를 공유하기 어렵다는 코드를 공유하기 어렵다는 단점이 있다.  다른 상수도 static이므로 열거타입 생성자에서 정적필드에 접근할 수 없는 재약 또한 있다. 다음 예를 보면서 생각해보자

~~~java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

		private static final int MINS_PER_SHIFT = 8*60;
         int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
           swich(this){
             case SATURDAY : case SUNDAY:
             	overtimePay = basePay / 2;
             	break;
             default:
             overtimePay = minuㅅㄷㄴ째갇ㅇ <= MINS_PER_SHIFT ?
               0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
           }
            return basePay + overtimePay(minsWorked, payRate);
        }
}
~~~

 코드는 간결하고 좋은데 문재가 있다. 관리의 관점에서 보면 매우 위험한데, 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case문을 잊지 말고 쌍으로 넣어줘야 한다. 

 이를 해결하는 방법은 2가지 가 있다. 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣으면 된다. 두번째로 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한다음 상수가 자신에게 필요한 메서드를 호출하면 된다. 두 방식 모두 가독성이 떨어지고 오류 발생 가능성이 높아진다. 

 두번째 방법이 그나마 나은데 잔업 수당을 계산하는 enum을 private 중첩 열거 타입으로 옮기고 상수가 적당한것을 선택하는 것이다.

~~~java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    // PayrollDay() { this(PayType.WEEKDAY); } // (역자 노트) 원서 4쇄부터 삭제
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
~~~

이 패턴은 swich문보다 복잡하지만 더 안전하고 유연하다.



###열거타입은 언제 쓰는것이 좋나

열거 타입은 언제 쓰는 것이 좋을까??? 열거타입은 필요한 원소를 컴파일 타임에 다 알수 있는 상수 집합이면 사용하는 것이 좋다. 