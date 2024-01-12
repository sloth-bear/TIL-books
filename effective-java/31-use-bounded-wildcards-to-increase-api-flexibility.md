# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.12             |
| PART | 31. 한정적 와일드카드를 사용해 API 유연성을 높이라 |

# 5장 Generic
* 자바 5부터 지원된 generic - 지원 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다. 
* 제네릭 사용 시 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주게 된다: 컴파일 타임에서 형변환 오류를 차단하므로 더 안전하고 명확한 프로그램을 만들어준다. 
* 컬렉션이 아니더라도 관련 이점을 누릴 수 있다. 
* 이번 장에서는 제네릭의 이점을 최대로 살리고 단점을 최소화하는 방법을 이야기한다.

## Item31. 한정적 와일드카드를 사용해 API 유연성을 높이라

### Invariant (불공변)
* Parameterize type(매개변수화 타입)은 불공변이다. 
* 즉, 서로 다른 타입 `List<String>`은 `List<Object>`의 하위 타입이 아니다. `List<Object>`에는 어떤 객체든 넣을 수 있지만, 전자는 불가능하다. 즉, `List<String>`은 `List<Object>`가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다. (리스코트 치환 원칙 위배;Item10)
* 하지만 때로 불공변 방식보다 유연한 무언가가 필요하다. 

### Bounded wildcard type (한정적 와일드카드 타입)
```java
public Stack<E> {
  private E[] elements;

  public void pushAll(Iterable<E> src) {  
    for (E e : src)  
      push(e);  
  }
  ...
}

Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```

* `Integer`는 `Number`의 하위타입이니 잘 동작할 것 같지만 실제로는 매개변수화 타입이 불공변이기 때문에 컴파일 에러가 발생한다. (incompatible types)
* 이 경우 한정적 와일드 타입이라는 특별한 매개변수화 타입을 사용할 수 있다. 

* 아래와 같이 `pushAll`의 input parameter 타입은 `E`의 `Iterable`이 아니라, `E의 하위 타입의 Iterable`이어야 한다. (extends라는 키워드는 이 상황에 딱 어울리진 않는다. 하위 타입이란 자기 자신도 포함하지만, 그렇다고 자신을 확장한 것은 아니기 때문이다.)
```java
public void pushAll(Iterable<? extends E> src) {
  for (E e : src) 
    push(e);
}
```

원소를 옮기려고 한다면 어떨까?
```java
public void popAll(Collection<E> dst) {
  while (!isEmpty())
    dst.add(pop());
}

Stack<Number> numberStack = new Stack();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

* 위의 경우와 비슷하게 `Collection<Object>`는 `Collection<Number>`의 하위 타입이 아니라는 에러가 발생한다. 
* `popAll`의 Input parameter 타입이 `E`의 `Collection`이 아니라, E의 상위 타입의 `Collection`이어야 한다. (모든 타입은 자기 자신의 상위 타입이다.)
```java
public void popAll(Collection<? super E> dst) {  
  while (!isEmpty())  
    dst.add(pop());  
}
```

* 유연성을 극대화하려면 원소의 생산자나 소비자용 input parameter에 Wildcard 타입을 사용하라. 
* 단, input parameter가 생산자/소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 것이 없다. 타입을 정확하게 지정하고, 와일드 카드 타입을 쓰지 말아야 한다. 


#### PECS: producer-extends, consumer-super
* 매개변수화 타입 `T`가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용하라.

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {
  ...
}
```
* return 타입은 여전히 `Set<E>`이다. 왜 그럴까? 반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다. 유연성을 높이기는 커녕, 클라이언트 코드에서도 와일드카드 타입을 써야 하기 때문이다.
* 클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다. 

##### 예시
```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```
위 예시는 PECS 공식을 두 번 사용했다.
1. `Comparable<E>`: `E` instance를 소비한다. 따라서 `Comparable<? super E>`로 대체한다. (일반적으로 Comparable과 Comparator는 이처럼 사용하는 편이 낫다.)
2. 이렇게 작성해야 하는 이유는, `Comparable`을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드 카드가 필요하다. 


### Type parameter vs wildcard
메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라. (두 가지 케이스 모두 가능한데, 신경 써야 할 타입 매개변수가 없다.)
```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

#### 예외 케이스
```java
public static void swap(List<?> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```
위 코드는 컴파일되지 않는다. `List<?>`에는 `null` 외에 어떤 값도 넣을 수 없기 때문이다. 이때는 와일드 카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용할 수 있다. 
```java
private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```
