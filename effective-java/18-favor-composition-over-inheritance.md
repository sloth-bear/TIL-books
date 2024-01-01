# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.01             |
| PART | 18. 상속보다는 컴포지션을 사용하라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.

## Item18. 상속보다는 컴포지션을 사용하라
상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다. 

### Safe to use inheritance
1. 같은 프로그래머가 통제하는 패키지 내
2. 확장할 목적으로 설계되었고 문서화도 잘 된 케이스 (Item 19)

### Unsafe to use inheritance
1. 일반적으로 다른 패키지의 구체 클래스를 상속하는 일은 위험하다. (`interface` 구현을 이야기하는 것은 아니다.)

### 상속의 위험성 - 캡슐화 위반
1. 상위 클래스가 어떻게 구현되냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다. (캡슐화 위반)
2. 상위 클래스 설계자는 확장을 충분히 고려하고 문서화도 제대로 해두어야 한다. 그렇지 않으면, 하위 클래스는 상위 클래스의 변화에 발맞춰 수정되어야 한다.

#### 캡슐화 위반 예시 
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
  private int addCount = 0;

  public InstrumentedHashSet() {}
  public InstrumentedHashSet(int initCap, float loadFactor) {
    super(initCap, loadFactor);
  }

  @Override 
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override 
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    // HashSet의 addAll은 각 원소에 대해 add 메서드를 호출해 추가한다.
    // 이떄 불리는 add는 overriding 된 add가 호출될 것이다. 
    return super.addAll(e);
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```

`addAll` 메서드는 `HashSet`의 `addAll`을 호출하는데 `HashSet` `addAll`은 내부적으로 `add()`를 호출해 추가한다. `addCount`는 두 번씩 추가되어 올바르지 않은 값을 반환한다.

1. `addAll()`을 재정의하지 않는다면? 문제는 당장 고칠 수 있지만 `HashSet`의 내부 로직이 어떻게 변할지 모르는 상황이므로 이 `InstrumentHashSet`은 깨지기 쉽다. 
2. `addAll()` 재정의 시 컬렉션을 순회하며 하나당 `add` 메서드를 한 번만 호출한다면? 상위 클래스의 `addAll`에 기대지 않으므로 조금 더 나은 해법이다. 그러나 상위 클래스의 메서드 동작을 다시 구현하는 이 방식은 어렵다. 성능을 떨어뜨리거나 오류를 낼 수도 있다. 또한 접근 불가한 `private` 필드를 써야 된다면? 구현도 불가능하다. 
 3. 상위 클래스에 새로운 메서드가 추가된다면? 예를 들어 이 클래스에서 추가될 원소는 특정 조건을 만족해야 되는데, 상위 클래스에서 새로운 메서드가 추가되면 허용되지 않은 원소를 추가할 수 있게 된다. (보안 구멍이 생긴다.) 
 4. 상위 클래스의 메서드를 재정의하는 대신 새로운 메서드를 추가한다면?
      -> 다음 릴리스에서 우연찮게 시그니처가 같은 메서드가 추가된다면? 컴파일도 되지 않을 확률이 있다. 
      -> 이 클래스의 메서드는 상위 클래스의 메서드가 요구하는 규악을 만족하지 못할 가능성이 크다. 

### 해결책: Composition(구성) - 상속 대신 composition
기존의 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자. 그리고 기존 클래스의 메서드를 호출하면서 결과를 반환한다. `forwarding` 방식이다. 새 클래스의 메서드는 `forwarding method`라고 부른다. 

#### 예시
`InstrumentedSet<E>`
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
  private int addCount = 0;

  public InstrumentedSet(Set<E> s) {
    super(s);
  }

  @Override 
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }
  
  @Override 
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(e);
  }
  
  public int getAddCount() {
    return addCount;
  }
}
```

`ForwardingSet<E>`
```java
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;

  public ForwardingSet(Set<E> s) { this.s = s; }

  public boolean add(E e) {
    return s.add(e); 
  }

  public boolean addAll(Collection<? extends E> c) {
    return s.addAll(c);
  }
}
```

이 ForwardingSet은 HashSet의 구현과는 무관해졌다. 즉, HashSet의 API를 호출하기만 하면 된다. 다른 Set instane를 감싸고 있다는 뜻에서 InstrumentedSet 같은 클래스를 wrapper 클래스라고 한다. 다른 `Set`에 계측 기능을 덧씌운다는 뜻에서 `Decorator pattern`이라고 한다. (정확하게는 ForwardingSet이 wrapper class 아닐까? 그리고 의도에 따라 `proxy pattern` 이라고도 할 수 있겠다.)
그리고 컴포지션과 Forwarding의 조합은 넓은 의미로 위임(delegation)이라고도 한다. 엄밀히 따지면, 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다. 

### wrapper class의 단점
단점이 거의 없다. 다만, 콜백(callback) framework와는 어울리지 않는다. 성능이나 메모리 사용량에 주는 문제를 걱정하는 사람들도 있지만, 실전에서는 둘 다 별다른 영향이 없다고 밝혀졌다. 

#### SELF 문제
콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다. 이를 [SELF 문제](https://softwareengineering.stackexchange.com/questions/117628/why-are-wrapper-classes-not-suited-for-use-in-callback-frameworks)라고 한다. 


### 상속이 쓰일 때는?
#### is-a
상속은 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다. 즉, class B가 class A와 is-a 관계일 때만 class A를 상속해야 한다. 
B가 정말 A인가? 자문했을 때 확신할 수 없다면 상속해서는 안 된다. A를 instance로 두고, A와는 다른 API를 제공해야 하는 상황이 대다수다. 즉, 필수 구성 요소가 아니라 구현 방법 중 하나일 뿐이다. 
또한, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다. 

그리고 추가적으로 아래 내용을 자문해 보아야 한다. 
1. 확장하고자 하는 클래스 API에 아무런 결함이 없는가?
2. 결함이 있다면, 이 결함이 여러분 클래스의 API까지 전파되어도 괜찮은가? 


### JDK에서의 위반 사례
1. `Stack` extends `Vector`
2. `Properties` extends `HashTable`
