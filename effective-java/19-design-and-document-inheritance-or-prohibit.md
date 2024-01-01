# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.01             |
| PART | 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.

## Item19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
### 상속을 고려한 문서화
메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서화 하는 것이다. 
  * 재정의 가능한 메서드(public, protected 메서드 중 final이 아닌 것)들을 내부적으로 어떻게 이용하는지 (자기사용)
  * 재정의 가능한 메서드를 호출하는 메서드에도 API 설명에 그 사실을 적시 
  * 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지 
  * `@impleSpec` 참조 (java 8에서 도입, Java 9부터 본격 사용)

#### 예시: `AbstractCollection`
다음은 `public boolean remove(Object o)` 메서드의 doc 일부이다. 
```java
/*
* @implSpec  
* This implementation iterates over the collection looking for the  
* specified element.  If it finds the element, it removes the element  
* from the collection using the iterator's remove method.
* ...
*/
public boolean remove(Object o) {
  ...
}
```

#### 좋은 API 문서 작성 격언 위배?
좋은 API 문서란 '어떻게'가 아닌 '무엇'을 하는지 설명해야 한다는 격언과는 대치되는 것이 맞다. 그러나 상속은 캡슐화를 해치기 때문에 클래스를 안전하게 상속할 수 있게 하려면 (상속이 아니었다면 기술하지 않았어야 할) 내부 구현 방식을 설명해야 한다. 

### 상속을 고려한 설계
1. 클래스 내부 동작 중 필요하다면? `protected` 메서드 형태로 공개 고려
    * 클래스 내부 동작 과정 중 끼어들 수 있는 hook을 잘 선별하여 protected 메서드 형태로 공개해야 할 수 있다. 
    * ***심사숙고해서 잘 예측해보고, 실제 하위 클래스를 만들어 시험해보는 것이 최선이며 유일하다.***  즉, 거꾸로 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 `protected` 멤버는 사실 `private`이어야 할 가능성이 크다. (3개 정도를 추천한다. 하나 이상은 제 3자가 작성해본다.) 
    * `protected`로 어쩔 수 없이 공개된 메서드 하나하나가 내부 구현에 해당하므로 가능한 적어야 한다. 
    * 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의해야 한다. 
  2. 상속용 클래스 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드 호출 금지 
    * 이 규칙을 어기면 프로그램이 오작동할 것이다. 생성자가 하위 클래스 생성자보다 먼저 호출되므로 하위에서 재정의한 메서드가 실행된다. 


#### 예시: `AbstractList`
`clar` 메서드를 고성능으로 만들기 쉽게 하기 위해서인데, 이 메서드가 없다면 제거할 원소 수의 제곱에 비례해 성능이 느려지거나 부분리스트의 메커니즘을 밑바닥부터 새로 구현해야 했을 것이다. 때문에 `protected`형태로 고려된 것이다.
```java
/*
* Implementation Requirements:
* This implementation gets a list iterator positioned before fromIndex, and repeatedly calls ListIterator.next followed by ListIterator.remove until the entire range has been removed. Note: if ListIterator.remove requires linear time, this implementation requires quadratic time.
*/
protected void removeRange(int fromIndex, int toIndex) {
  ...
}
```


### `Cloneable`과 `Serializable`
1. 가능한 상속을 피한다. 
    이 두 인터페이스는 상속용 설계의 어려움을 한층 더해준다. 둘 중 하나라도 구현한 클래스를 상속할 수 있게 설계하는 것은 일반적으로 좋지 않은 생각이다. 확장 시에 엄청난 부담이 된다. (상속 가능하게 하는 특별한 방법도 있다: Item 13, Item 86)
  2. `clone`과 `readObject`에서 직/간접적으로 재정의 가능 메서드를 호출해서는 안 된다. 
    * `clone()`과 `readObject` 메서드는 생성자와 비슷한 효과를 낸다. (새로운 객체를 만든다.) 따라서 이들을 상속할 때 따르는 제약도 생성자와 비슷하다. ***즉, 이 두 메서드는 직/간접적으로 재정의 가능 메서드를 호출해서는 안 된다.***
    * `clone`: 하위 클래스의 `clone` 메서드가 복제본의 상태를 수정하기 전에 재정의한 메서드를 호출한다. 
    * `readObject`: 하위 클래스의 상태가 미처 다 역직렬화 되기 전에 재정의한 메서드부터 호출하게 된다. 
  3. `Serializable` 구현 클래스를 상속해 `readResolve`나 `writeReplace`를 가진다면 이 메서드들은 `protected`로 선언해야 된다. `private`으로 선언하면 하위에서 무시된다. 
  

### 상속용으로 설계되지 않은 클래스 관리
final도 아니고, 상속용으로 설계되거나 문서화되지도 않았다면? 상속을 금지한다. 
1. 클래스 final 선언
2. 모든 생성자를 `private` 혹은 `package-private`로 선언하고, `public` 정적 팩터리 생성 

구체 클래스가 표준 인터페이스를 구현하지 않았는데 상속을 금지하면 사용하기에 상당히 불편해진다. 이런 클래스에 대해 상속을 허용하려면 클래스 내부에서 재정의 가능한 메서드를 사용하지 않게 만들고 이 사실을 문서로 남기자: 각각의 재정의 가능 메서드를 모두 `private`으로 만들고, 이 도우미 메서드를 호출하도록 수정한다.