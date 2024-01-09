# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.09             |
| PART | 29. 이왕이면 제네릭 타입으로 만들라 |

# 5장 Generic
* 자바 5부터 지원된 generic - 지원 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다. 
* 제네릭 사용 시 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주게 된다: 컴파일 타임에서 형변환 오류를 차단하므로 더 안전하고 명확한 프로그램을 만들어준다. 
* 컬렉션이 아니더라도 관련 이점을 누릴 수 있다. 
* 이번 장에서는 제네릭의 이점을 최대로 살리고 단점을 최소화하는 방법을 이야기한다.

## Item29. 이왕이면 제네릭 타입으로 만들라

### 일반 클래스를 제네릭 클래스로 만들기
1. 클래스 선언에 타입 매개변수를 추가 (일반적으로 원소는 `E`; Item 68)
2. 코드에 쓰여있던 Object를 적절한 타입 매개변수로 바꾸고 컴파일 -> 컴파일 에러를 하나씩 수정

### 배열 -> 제네릭 클래스 예제

Item07의 아래 예제는 제네릭이 절실한 예제이다.
```java
public class Stack {
  private Object[] elements;
  ...

  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e) { ... }

  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    ...
    return result;
  }
}
```

#### 1. 우회 방법
* 제네릭 배열 생성을 금지하는 제약을 우회하는 방법
* `push` 메서드를 보면 elements는 항상 E를 추가하므로 형번환 예외가 발생하지 않는다. (컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없지만 내부적으로 비검사 형변환은 확실히 안전하다.)
* 가독성이 더 좋다. 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필하고, 코드도 더 짧다. 형변환은 배열 생성 시 단 한 번만 해주면 된다. 
* 그러나 배열의 런타임 타입(E가 Object가 아닌 한)이 컴파일타임 타입과 달라 힙 오염(heap pollution; Item 32)를 일으킨다. 
```java
public class Stack<E> {
  private E[] elements;
  ...

  @SuppressWarnings("unchecked")
  public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(E e) { ... }

  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    ...
    return result;
  }
}
```

#### 2. 비검사 형변환
배열 타입은 두고, 비검사 형변환을 수행하는 할당문 쪽만 수정한다. 
* 배열에서 원소를 읽을 때마다 형변환해줘야 하는 단점이 있다. 
```java
public class Stack<E> {
  private Object[] elements;
  ...

  public Stack() {
    elements = Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(E e) { ... }

  public E pop() {
    if (size == 0)
      throw new EmptyStackException();

	@SuppressWarnings("unchecked")
    E result = (E) elements[--size];
    ...
    return result;
  }
}
```


### 정리
* 배열보다는 리스트를 우선하라는 Item 28과 모순되어보인다. 사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다. 자바가 List를 기본 타입으로 제공하지 않으므로  ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다. HasMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다. 
* 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않지만, 기본 타입은 사용할 수 없다. 자바 제네릭 타입 시스템의 근본적인 문제이나 박싱된 기본 타입(Item 61)을 사용해 우회할 수 있다.
* Type parameter에 제약을 두는 제네릭 타입도 있다.  (한정적 타입 매개변수, bounded type parameter) 
	  e.g. `DelayQueue<E extends Delayed>`
* 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라.
* 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자. 
