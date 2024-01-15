# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.15             |
| PART | 33. 타입 안전 이종 컨테이너를 고려하라 |

# 5장 Generic
* 자바 5부터 지원된 generic - 지원 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다. 
* 제네릭 사용 시 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주게 된다: 컴파일 타임에서 형변환 오류를 차단하므로 더 안전하고 명확한 프로그램을 만들어준다. 
* 컬렉션이 아니더라도 관련 이점을 누릴 수 있다. 
* 이번 장에서는 제네릭의 이점을 최대로 살리고 단점을 최소화하는 방법을 이야기한다.


## Item33. 타입 안전 이종 컨테이너를 고려하라

### Common uses of generic
* `Set<E>`, `Map<K, V>`와 같은 collections
* `ThreadLocal<T>`, `AtomicReference<T>`와 같은 single-element container
* 이는 모두 ***Parameterized container***들이며, type parameter의 수가 제한된다.
* 컨테이너의 일반적인 용도에 맞춰 설계되었으니 문제될 것은 없다.

### Parameterize the key instead of the container 
* More flexibility - 좀 더 유연한 케이스를 사용해야 될 때 
* 더 많은 type parameter를 사용할 때는 없을까? e.g. 많은 column을 가진 Database, 이 column들이 Typesafe 하려면? (`Column<T>`)
* 컨테이너 대신 Key를 Parameterize한 다음, 이를 제공하여 값을 삽입하거나 뺀다. 
* 이를 **type safe heterogeneous container pattern** 이라고 한다. 


### Type safe heterogeneous container pattern 
#### Example
* type 별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 클래스 
* 각 Type의 Class 객체: Parameterized key 
* `Class` 클래스가 generic이기 때문에 동작한다: `Class<T>`
* compile time, runtime의 type 정보를 커뮤니케이션하기 위해 메서드들은 Class literal을 주고 받는다. 이를 *type token* 이라고 한다. 

##### Typesafe heterogeneous container pattern - API
```java
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> getFavorite(Class<T> type);
}
```

##### Typesafe heterogeneous container pattern - client
* 즐겨찾는 `String`, `Integer`, `Class` Instance를 저장, 검색, 출력하는 프로그램이다. 
```java
public static void main(String[] args) {
  Favorites f = new Favorites();
  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, 0xcafebabe);
  f.putFavorite(Class.class, Favorites.class);

  String favoriteStr = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Integer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);

  // Console: Java cafebabe Favorites
  System.out.printf("%s %x %s%n", favoriteStr, favoriteInteger, favoriteClass);
}
```

##### Typesafe heterogeneous container pattern - implementation
* `Favorites` instance는 *typesafe* 하다. `String`을 요청했는데 `Integer`를 return 하는 일 같은 건 없다.
* 일반적인 맵과 달리, 각 Key 들은 모두 타입이 다를 수 있다. 
```java
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<>();

  public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
  }

  public <T> T getFavorite(Class<T> type) {
    // 참조 Class<T> - T cast(Object obj); 
    // Class 클래스가 generic이라는 점을 완벽하게 활용한다. 
    return type.cast(favorites.get(type));
  }
}
```

* `Map<Class<?>, Object>`에서 unbounded wildcard type이라 맵 안에 넣을 수 없을 것 같지만, wildcard type은 Nested된 점에 주목하자. 즉, 이는 map의 type(wildcard type인)이 아니라, 그 Key의 type이다. e.g. `Class<String>, Class<Integer>` 그러므로 다양한 타입이 가능해진다. 
* `Map<Class<?>, Object>`에서 value type이 `Object`인 점을 보자. 단순 `Object`로, key와 value 사이의 타입 관계를 보증하지 않는다. (명시적으로 표현할 방법이 없다.) 그러나 프로그램적으로 이 관계가 성립함을 알고 있다. 


#### 제약
1. 악의적인 client가 `Class` 객체를 raw type으로 넘기면 타입 안정성이 쉽게 깨진다. (컴파일 시 Unchecked warning이 뜰 것이다.) -> 일반 컬렉션 구현체도 같은 문제가 있다.  
  ```java
  f.putFavorite((Class) Integer.class, "Integer의 인스턴스가 아닙니다.");
  int favoriteInteger = f.getFavorite(Integer.class); // ClassCaseException
  ```

  ```java
  public <T> void putFavorite(Class<T> type, T instance) {
    // dynamic cast를 통한 runtime type safety 확보
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
  }
  ```

2. 실체화 불가 타입(Item 28)에는 사용할 수 없다. e.g. `List<String>`
  * `List.class`에 `List<String>.class`, `List<Integer>.class`를 허용해서 같은 타입의 객체 참조를 반환하면 아수라장이 될 것이다.
  * [Neal Gafter - Super type token](https://gafter.blogspot.com/2007/05/limitation-of-super-type-tokens.html)를 활용할 수는 있는데, 이 방식도 한계가 있다. 


#### Bounded type token 
* unbounded type token에서 type를 한정하고 싶을 때 - bounded type parameter(Item 29), bounded wildcard (Item 31)를 사용하여 표현 가능한 타입을 제한하는 Type token 
* e.g. Annotation API
  ```java
  public <T extends Annotation>
    T getAnnotation(Class<T> annotationType);
  ```
