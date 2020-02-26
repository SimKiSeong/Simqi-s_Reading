# 22. 인터페이스는 타입을 정의하는 용도로만 사용하라



 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 잇는 타입역할을 한다. 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는 지를 클라이언트에 얘기해주는 것이다. 인터페이스는 오직 이용도로만 사용되어야 한다.



---

## 잘못된 예



 잘못된 예로는 대표적으로 상수 인터페이스가 있다. 메서드 없이 static final 필드로만 가득 찬 인터페이스를 말한다.

~~~java
public interface PhysicalConstants{
	// 아보가드로 수 (1/몰)
	static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  
  // 볼츠만 수 (J/K)
	static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
  
  // 전자 질량 (kg)
	static final double ELECTRON_MASS =9.109_383_56e-31;
}
~~~



 상수인터페이스 안티페턴은 인터페이스를 잘못 사용한 예다. 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. 따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위다. 

 java.io.ObjectStreamConstants등 자바 플랫폼 랑이브러리에도 상수 인터페이스가 몇개 있으나, 인터페이스를 잘못 활용한 예이니 따라 해서는 안된다.

 상수를 공개할 목적이라면 더 합당한 선택지들이 있다.

1. 특정클래스와 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야한다. (Integer.Min_Value 등)
2. 열거 타입으로 나타내기 적합한 상수라면 열거타입으로 만들어 공개하면 된다. (아이템 34)
3. 그것도 아니라면 인스턴스화 할 수 없는 유틸리티 클래스를 담아 공개하자.

~~~java
public class PhysicalConstants{
  private PhysicalConstants(){} // 인스턴스화 방지
  
  // 아보가드로 수 (1/몰)
	public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  // 볼츠만 수 (J/K)
  public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
  // 전자 질량 (kg)
	public static final double ELECTRON_MASS =9.109_383_56e-31;
}
~~~

