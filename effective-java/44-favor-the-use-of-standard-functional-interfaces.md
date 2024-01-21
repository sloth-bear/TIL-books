# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.21             |
| PART | 44. 표준 함수형 인터페이스를 사용하라  |

# 7장 Lambdas and Streams
* Java 8 - Functional interface, lambda, method reference 개념이 추가되면서 함수 개체를 더 쉽게 만들 수 있게 되었다. 
* 그리고 Stream API까지 추가되어 Data element의 시퀀스 처리를 라이브러리 차원에서 지원하기 시작했다. 
* 이 기능들을 효과적으로 사용하는 방법을 알아보자. 


## Item44. 표준 함수형 인터페이스를 사용하라

### 들어가기 전에...
* 자바가 람다를 지원하면서 API를 작성하는 모범 사례가 크게 바뀌었다. 
    * 템플리 메서드 패턴(상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현)의 매력이 크게 줄었다. 
    * 대신 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것이 현대적인 해법이다. 
* 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다. 이때 함수형 매개변수 타입을 올바르게 선택해야 한다. 

#### Example
* `LinkedHashMap` - 맵에 새로운 키를 추가하는 `put` 메서드에서 이 메서드를 호출하여 `true`가 반환되면 맵에서 가장 오래된 원소를 제거한다.
```java
propected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
  return size() > 100;
}
```

* 람다를 사용하면 훨씬 잘 해낼 수 있다.
```java
// BiPredicate<Map<K,V>, Map.Entry<K,V>>
@FunctionalInterface
interfcae EldestEntryRemovalFunction<K,V> {
  boolean remove(Map<K,V map, Map.Entry<K,V> eldest);
}

// least recently used cache factory
MapUtils.lruCache(cacheSize, (map, eldest) -> map.size() > cacheSize);
```


### 기본 인터페이스 - `java.util.function`
  | **Interface**     | **Function Signature** | **Example**         |
  |-------------------|------------------------|---------------------|
  | UnaryOperator<T>  | T apply(T t)           | String::toLowerCase |
  | BinaryOperator<T> | T apply(T t1, T t2)    | BigInteger::add     |
  | Predicate<T>      | boolean test(T t)      | Collection::isEmpty |
  | Function<T, R>    | R apply(T t)           | Arrays::asList      |
  | Supplier<T>       | T get()                | Instant::now        |
  | Consumer<T>       | void accept(T t)       | System.out::println |
  * 총 43개의 인터페이스가 담겨 있다. 전부 기억하긴 어렵겠지만, 기본 인터페이스 6개만 기억하면 나머지를 충분히 유추해낼 수 있다. 
* `Function` interface 및 그 변형들만 return type이 parameterized 되었다. 
* 표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자. 


#### 새로운 함수형 interface를 만들어야 할 때
* 동작 방식은 같더라도 `Comparator`과 같이 독자적인 인터페이스가 필요한 경우
    * API에서 굉장히 자주 사용되는데 지금의 이름이 그 용도를 훌륭히 설명해준다. 
    * 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다. 
    * 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 듬뿍 담고 있다. 
* 전용 함수형 인터페이스를 작성하기로 했다면, 인터페이스이므로 주의해서 설계해야 한다. (Item 21)
* 직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애니터에시녕르 사용하라 
    * 인터페이스가 람다용으로 설계된 것임을 알려준다.
    * 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다. 
    * 그 결과 유지보수 과정에서 실수로 메서드를 추가하지 못하게 막는다. 

#### 주의점 
* 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의해서는 안 된다.
    * 클라이언트에게 모호함을 안겨준다. 이로 인해 실제로 문제가 일어나기도 한다. 
    * e.g. `ExecutorService`의 `submit` 메서드는 `Callable<T>`, `Runnable`을 받는 것을 다중정의 했다. 그래서 올바른 메서드를 알려주기 위해 형변환해야 할 때가 왕왕 생긴다. (Item 52)
    * e.g2 `Constumer<T>`와 `Predicate<T>`를 다중정의하면, 컴파일러는 어떤 것을 사용해야 할지 모몰라 컴파일에러가 난다. 다중정의가 아니라 이름을 명확히하여 구분해주는 것이 좋다.


### 정리
1. API를 설계할 때 이제 람다도 염두에 두어야 한다. 
2. 입력값과 반환값에 함수형 인터페이스 타입을 활용하라. 보통은 `java.util.function` 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다. 
3. 흔치는 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있다. 
  