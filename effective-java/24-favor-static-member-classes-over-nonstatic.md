# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.04             |
| PART | 24. 멤버 클래스는 되도록 static으로 만들라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.

## Item24. 멤버 클래스는 되도록 static으로 만들라

### Nested Class
다른 클래스 안에 정의된 클래스다. 
* 자신을 감싼 바깥 클래스에서만 쓰여야 한다.
* 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

#### 종류
1. static member class
2. inner class - non-static member class
3. inner class - anonymous class
4. inner class - local class


### 언제, 왜 사용해야 하는가?
#### static member class
1. 다른 클래스 내 선언된다.
2. 바깥 클래스의 private 멤버에도 접근할 수 있다. 

static member class는 다른 static member와 동일한 접근 규칙을 적용 받는다. 예를 들어 private으로 선언하면 바깥 클래스에서만 접근할 수 있다. 

* 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미로 쓰인다. `Calculator.Operation.MINUS`

개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있는 경우에 만들어야 한다. 



#### non-static member class
non-static member class의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 그래서 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다. `ClsasName.this` 형태로 바깥 클래스 이름을 명시하는 용법이 정규화된 this다. 

개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있는 경우라면 static member class로 만들어야 한다. non-static member class는 바깥 인스턴스 없이는 생성할 수 없다. 
이 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다. 이 관계는 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어지는 게 보통이지만, 드물게는 직접 `바깥클래스.new MemberClass(args)`를 호출해 수동으로 만들기도 한다. non-static member class의 instance 내에 만들어져 메모리 공간을 차지하고, 생성 시간도 더 걸린다. 

##### 사용처
`Adapter` 정의 시에 자주 쓰인다. 즉, 어떤 클래스의 인스턴스를 감싸서 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다. 
e.g. `Map`인터페이스의 구현체 - `keySet, entrySet, valeus`가 반환하는 자신의 컬렉션 뷰를 구현할 때 
e.g. `Set`, `List` 같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때 non-static member class 자주 사용

```java
public class MySet<E> extends AbstractSet<E> {
	@Override 
	public Iterator<E> iterator() {
		return new MyIterator();
	}

	private class MyIterator implements Iterator<E> {
		...
	}
}
```

member class에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 static member class로 만들자. static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다. 이 참조를 저장하기 위해서는 시간과 공간이 소비된다. 그리고 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다. (Item 7)

e.g. `Map.Entry`: entry의 메서드들은 맵을 직접 사용하지 않는다. private static member class가 알맞다.


#### static / non-static class 주의사항
public이나 protected 멤버라면 훨씬 중요하므로 하위 호환성을 생각하자.


#### Anonymous class
member와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다. 어디서든 만들 수 있다. 그리고 오직 non-static 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다. static 문맥에서라도 상수 변수 외의 static member는 가질 수 있다. 


#### Local class
* 4가지 중첩 클래스 중 가장 드물게 사용된다.
* 지역변수를 선언할 수 있는 곳이면 어디서든 선언할 수 있고, 유효 범위도 지역변수와 같다. 
* 멤버 클래스처럼 이름이 있고, 반복해서 사용 가능하다. 
* 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다. 
* 정적 멤버는 가질 수 없고 가독성을 위해 짧게 작성해야 한다. 


### 정리

#### member class
메서드 밖에서도 사용해야 하거나, 메서드 안에 정의하기에 너무 길다면 멤버 클래스로 만든다. 
1. static: non-static member를 만들어야 될 때가 아닐 때
2. non-static: instance 각각이 바깥 instance를 참조하는 경우

#### Anonymous class & Local Class
1. anonymous: 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고, 해당 타입으로 쓰기에 적합한 클래스가 있을 때
2. local: 1.의 경우가 아닐 때