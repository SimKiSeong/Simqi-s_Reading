#33. 타입 안전 이종 컨테이너를 고려하라



 제네릭은 컬랙션뿐만아니라 단일원소 컨테이너에도 흔히 쓰인다. 이런 모든 쓰임에서 매개변수화 되는 대상은 원소가 아니라 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다. set의 경우 원소의 타입을 뜻하는 하나의 타입매개변수만 있으면 되며, Map에는 키와 값의 타입을 뜻하는 2개만 필요한 식이다.



## 이종컨테이너 패턴

 하지만 좀더 유연한 수단이 필요할때도 있다. 데이터배이스의 행은 임의 개수의 열을 가질수 있는데, 모든 열을 타입안전하게 이용할수 있다면 좋을 것이다. 이때 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺼 때 매개변수화한 키를 함께 제공하면 된다. 이러한 설계 방식을 이종 컨테이너 패턴이라 한다.

##예 시

 간단한 예로 타입별로 즐겨찾는 인스턴스를 저장하고 거머색할 수 있는 Favorites클래스를 생각해보자. 각 타입의 Class객체를 매개변수화한 키 역할로 사용하면 된다. 이 방식이 통하는 이유는 class리터럴의 타입이 Class\<T\>이기 때문이다.  String.class의 타입은 Class\<String\> 인것이다. 

 다음은 Favorites클래스의 API로, 클라이언트는 즐겨찾기르 저장하거나 얻어올때 Class객체를 알려주면 된다.

 ~~~java
public class Favorites{
  public <T> void putFfavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}
 ~~~



그리고 이 Favorites클래스를 사용하는 예시다.

~~~java
public static void main(String[] args) {
        Favorites f = new Favorites();
        
        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);
       
        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);
        
        System.out.printf("%s %x %s%n", favoriteString,
                favoriteInteger, favoriteClass.getName());
}
~~~

 기대한대로 이 프로그램은 Java Cafebabe Favorites를 출력한다.

###Favorites 구현

 Favorites 인스턴스는 타입 안전하다. String을 요청했는데 Integer를 반환하는 일은 없다. 또한 모든 키의 타입이 제각각이라, 일반적인 맵과 달리 여러가지 원소가 담긴다. 따라서 Favorites는 타입안전 이종컨테이너라 할만하다. Favorites의 구현은 단순하다.

~~~java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
~~~

 favorites의 타입인 Map\<Class\<?\>,Object\> 는 비 한정적 와일드카드 타입이라 아무것도 넣을수 없다고 생각하지만 사실은 그 반대다. 와일드 카드가 주업되어 있어서 키가 와일드 카드이다. 이는 모든 키가 서로 다른 매개변수화 타입아라는 뜻으로 Class\<String\> , Class\<Integer\> 이렇게 지원한다.

 favorites맵의 값타입은 Object라는 점을 알아야한다. 키와 값 사이의 타입 관계를 보증하지 않기 때문이다. 사실 자바 타입에는 이 관계를 명시할 방법이 없다. 하지만 우리는 이 관계가 성립함을 알고 있고 즐겨찾기를 검색할때 그 이점을 누릴 수 있다.

 putFavorites의 구현은 아주 쉽다. Class객체와 즐겨찾기 인스턴스를 favorites에 추가해 관계를 지으면 끝이다. 그 값이 그 키타입의 인스턴스라는 정보가 사라지지만 getFavorite에서 관계를 살릴 수 있다.

 getFavorite 코드는 주어진 Class객체에 해당하는 값을 favorites 맵에서 꺼낸다. 이 객체의 타입은 favorites맵의 값타입인 Object이나, 우리는 이를 T로 바꿔야 한다. 따라서 Class의 cast메서드를 통해 동적형변환 한다. Class의 cast메소드는 인수가 타입의 인스턴스를 검사해 맞다면 그대로 반환할건데 우리는 인수가 타입의 인스턴스인지 알고 쓰기 때문에 걱정할 필요가 없다. 또한 cast메소드는 Class클래스가 제네릭인점을 이용해 Class의 객체의 타입 매개변수와 같은점을 이용하기 때문에 사용한다.

### 제약사항

 Favorites클래스에는 제약이 2가지 있다. 첫번재는 악의적인 클라이언트가 Class객체를 제네릭이 아닌 로타입으로 너머기면 안정성이 쉽게 깨진다. 하지만 이러한 클라이언트 코드는 컴파일 단계에서 검출된다. 

 두번째 제약은 실체화 불가타입에는 사용할수 없다는 것이다. String이나 String[]은 저장할 수 있어도, List\<String\> 은 불가능하다. 