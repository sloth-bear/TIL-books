# Today I Learned

| 구분 | 내용                            |
| ---- | ------------------------------|
| DATE | 2023.12.29                    |
| PART | 13. clone 재정의는 주의해서 진행해라 |

# 4장 Methods common to All Objects (모든 객체의 공통 메서드)
* `Object`: 객체를 만들 수 있는 구체 클래스지만, 기본적으로는 상속해서 사용하도록 설계됨 
* `final`이 아닌 메서드(`equals, hashCode, toString, clnoe, finalize`)는 모두 overriding을 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있음. 
* 그래서 `Object`를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의해야 함 -> 만약 메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(`HashMap`과 `HashSet` 등)를 오작동하게 만들 수 있다. 
* ***`final`이 아닌 `Object` 메서드들을 언제 어떻게 재정의해야 하는가에 초점***

## Item13. clone 재정의는 주의해서 진행해라
* `clone` 메서드를 잘 동작하게끔 해주는 구현 방법
* 언제 구현 방법을 써야 되는지 
* 가능한 다른 선택지 

### `Cloneable` 
* 복제해도 되는 클래스임을 명시하는 용도의 mixin interface(Item 20). 
* 메서드가 없다. 

### 언제 구현 방법을 써야 될까? 
* 결론부터 이야기하면 `Cloneable`은 많은 문제점을 가지고 있다. 새로운 인터페이스를 만들 땐 절대 `Cloneable`을 확장해서는 안 되고, 새로운 클래스도 이를 구현해선 안 된다. 
* `final` 클래스라면 `Cloneable`을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별 다른 문제가 없을 때만 드물게 허용해야 한다.(Item 67)
* 기본 원칙은 복제 기능은 생성자와 팩터리를 이용하는 것이 가장 좋다. 단, 배열만은 `clone` 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라고 볼 수 있다. 

### 문제점 
1. `Cloneable` 인터페이스 자체에는 `clone` 메서드가 정의되어 있지 않다. 즉, 메서드가 전혀 없는 마커 인터페이스다. 그렇다면 `clone` 메서드는 어디에 정의되어있을까? `Object` 클래스에서 `protected`로 정의되어있다. 즉, 클래스가 `Cloneable` interface를 구현하더라도 `clone` 메서드를 오버라이드 하고, `public`으로 만들지 않으면 외부에서 사용할 수 없음을 의미한다. 
	* ***즉, `Cloneable`을 구현하는 것만으로는 외부 객체에서 `clone` 메서드를 호출할 수 없다.*** 
		-> `Cloneable` interface를 구현하고 (그렇지 않으면 `clone` 호출 시`CloneNotSupportedException`이 발생한다. 컴파일 시점에선 알 수 없다.)
		-> `clone` method를 override 하고 
		-> `public`으로 변경해야 한다.
2. 의도한 목적을 제대로 이루지 못한 것: 객체의 복사본을 생성할 수 있는 클래스를 표시하기 위함이지만, 실제 구현은 구현이 복잡하고 계약이 불명확하다. 
3. deep copy & shallow copy: `Object`의 기본 `clone` 메서드는 얕은 복사를 수행한다. 객체가 복잡한 구조를 가진 경우, deep copy 수행을 위해서는 신중하게 오버라이딩해야 한다. 

그러나 이를 포함한 여러 문제점에도 불구하고 `Cloneable` 방식은 널리 쓰이고 있어 잘 알아두는 것이 좋다. 

### 동작
* `Object`의 protected 메서드인 `clone`의 동작 방식을 결정한다. 
* `Cloneable`을 구현한 클래스의 인스턴스에서 `clone`을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다. 
* 인터페이스를 상당히 이례적으로 사용한 예이니 따라 하지는 말자. (이 인터페이스에서 정의한 기능을 제공한다고 선언하는 것이 보통의 경우인데, 이 경우는 상위 클래스에 정의된 `protected` 메서드 동작 방식을 변경한 것.)
* 명세에서 이야기하진 않지만 실무에서 `Cloneable`을 구현한 클래스는 `clone` 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. -> 생성자를 호출하지 않아도 객체를 생성할 수 있게 된다.

### 구현 방법
####  `super.clone()` 호출 
* 클래스에 정의된 모든 필드는 원본 필드와 같은 값을 같는다. 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽한 상태이므로 더 손 볼 것이 없다. 
* `super.clone()` 대신 생성자를 호출해도 결과가 같을 수 있지만, 하위 클래스에서 호출하면 문제가 생기고 `Cloneable`을 구현할 이유도 없다.
* 그런데, 불변 객체는 쓸데 없는 복사를 지양한다. 굳이 `clone` 메서드를 제공하지 않는 것이 좋다. 
* 그럼에도 불구하고 `clone` 메서드를 제공하자면, 아래와 같이 구현할 수 있다. 
  ```java
  @Override
  public PhoneNumber clone() {
	  try {
		  return (PhoneNumber) super.clone();
	  } catch (CloneNotSupportedException e) { 
		  // checked excpetion catching - unchecked exception이어야 했다.
		  throw new AssertionError();
	  }
  }
  ```

#### 내부 필드가 가변 객체일 경우: `clone` 재귀적 호출
* 단순 `super.clone()`만으로 복사한다면, 참조 필드는 원본 인스턴스와 똑같은 필드를 참조하게 된다. 원본, 복제본 중 하나만 수정해도 다른 하나가 수정되어 불변식을 해친다. 
* 즉, `clone`은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.
* 가장 쉬운 방법은 배열의 `clone`을 재귀적으로 호출하는 것이다.
```java
public class Stack {
	private Object[] elements;
	private int size = 0;

	...

	@Override
	public Stack clone() {
		try {
			Stack result = (Stack) super.clone();
			result.elements = elements.clone();
			return result;
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}
}
```

* 배열의 `clone`은 런타임 타입, 컴파일 타임의 타입 모두 원본 배열과 똑같은 배열을 반환한다. 따라서 배열을 복제할 때는 배열의 `clone` 메서드를 사용하라고 권장한다. `clone` 기능을 제대로 사용하는 유일한 예라고 할 수 있다. 
* 문제점
	* 가변 객체를 참조하는 필드는 final로 선언하라는 용법과 충돌된다. elements는 가변 객체인데, final로 선언하면 새로운 값을 할당할 수가 없다. 그렇다면 복사는 어떻게 해야 되는가? 복제 가능한 클래스를 만들기 위해 일부 필드에서 final 키워드를 제거해야 될까? 

#### 내부 필드가 가변 객체일 경우: `clone` 반복문에서 호출 
* HashTable의 clone 메서드는 적절한 크기의 새로운 버킷 배열을 할당한 다음, 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은 복사를 한다. (entry.deepCopy()) 이 Entry의 deepCopy 메서드는 자신이 가리키는 연결 리스트를 복사하기 위해서 자신을 재귀적으로 호출하는데, 리스트의 원소 수만큼 스택 프레임을 소비하여 리스트가 길면 스택 오버플로를 일으킬 위험이 있다. 
```java
public class HashTable implements Cloneable {
	private Entry[] buckets = ...;
	
	private static class Entry {
		final Object key;
		Object value;
		Entry next;

		Entry(Object key, Object value, Entry next) {
			this.key = key;
			this.value = value;
			this.next = next;
		}

		Entry deepCopy() {
			return new Entry(key, value, next == null ? null : next.deepCopy());
		}
	}
	...

	@Override 
	public HashTable clone() {
		try {
			HashTable result = (HashTable) super.clone();
			result.buckets = buckets.clone();
			return result;
		}
	}
}
```

* Entry의 deepCopy()는 리스트의 원소 수 만큼 스택 프레임을 소비하므로, 리스트가 길면 스택 오버플로우를 일으키기 쉽다. 아래와 같이 반복문으로 대체하자. 
```java
Entry deepCopy() {
	Entry result = new Entry(key, value, next);
	for (Entry p = result; p.next != null; p = p.next) {
		p.next = new Entry(p.next.key, p.next.value, p.next.next);
	}
	return result;
}
```


#### 내부 필드가 가변 객체일 경우: `super.clone()` 호출 및 원본 객체의 상태를 다시 생성하는 고수준 메서드 호출 
* `super.clone()`을 통해 얻은 객체의 모든 필드를 초기 상태로 설정하고, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다. 
* 생성자에서는 재정의될 수 있는 메서드를 호출하지 않아야 하는데 `clone` 메서드도 마찬가지다. 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다. ***즉, `clone` 메서드에서 호출할 메서드는 final이거나 private이어야 한다.*** 


### `Cloneable` 구현을 해서는 안 되는 경우
상속용 클래스는 `Cloneable`을 구현해서는 안 된다. 
1. 동작하는 protected `clone` 메서드와 CloneNotSupportedException throws: 하위 클래스에서 결정
2. 미동작하는 public `clone` 메서드 구현 
	```java
	@Ovveride
	protected final Object clone() throws CloneNotSupportedException {
		throw new CloneNotSupportedException();
	}
	```


### 주의사항
* `Cloneable` 구현한 스레드 안전 클래스를 작성할 때는 `clone` 메서드 역시 적절히 동기화해줘야 한다. 


### Recap
구현 방법을 요약하면 아래와 같다. 
1. `Cloneable` 구현 클래스는 `clone`을 재정의해야 한다. 접근 제한자는 public, 반환 타입은 클래스 자신으로 변경한다. 
2. 가장 먼저 `super.clone()`을 호출한다. 
3. 필요한 필드를 전부 적절히 수정한다: 객체의 내부 "깊은 구조"에 숨어 있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야 된다. (주로 clone을 재귀적으로 호출하지만, 항상 최선인 것은 아니다.) - 불변 객체 참조(혹은 기본 타입)만 가진다면 아무 필드도 수정할 필요가 없다. 단, 일련번호나 고유 ID는 비록 기본 타입이나 불변일지라도 수정해줘야 한다. 


### 정리 
* 꼭 필요할까? 이처럼 복잡한 경우는 사실 드물다. 
* `Cloneable`을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 `clone`이 잘 동작하도록 구현해야 된다. (애초에 안 하는 것이 좋을 것 같다.)
* 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다. (자신과 같은 클래스의 인스턴스를 인수로 받는 생성자)