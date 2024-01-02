# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.02             |
| PART | 20. 추상 클래스보다는 인터페이스를 우선하라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.

## Item20. 추상 클래스보다는 인터페이스를 우선하라

### Java의 다중 구현 메커니즘
* Interfaces (인터페이스)
* Abstract classes(추상 클래스)

#### 차이
* 상속
	* 상속한 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다. (단일상속)
	* 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어려운 게 일반적이다. 두 클래스가 같은 추상 클래스를 확장하길 원한다면? 그 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 한다. 이는 클래스 계층구조에 커다란 혼란을 일으킨다. (새로 추가된 추상 클래스의 모든 자손이 이를 강제로 상속하게 된다.)
* 구현
	* 인터페이스 구현 클래스는 어떤 클래스를 구현했든 같은 타입으로 취급한다. (다중구현) 
	* 즉, 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다. 


### Interface
#### Mixins
* 인터페이스는 mixin 정의에 안성맞춤이다.
* mixin: Type that a class implements additionally to its 'primary type'
	* 클래스가 구현할 수 있는 타입
	* 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다. 
	* 즉, 대상 타입의 주된 기능에 선택적 기능을 혼합(mixed in)한다고 해서 믹스인
	*  e.g. `Comparable`: 대상 타입의 주된 기능 + 순서를 정할 수 있음을 선언 
* 상속과 달리 다중구현이 가능하므로 mixin을 정의할 수 있다. 

#### non-hierachical type framework 설계
* 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있다. 
* 계층을 엄격히 구분하기 어려운 개념에 대해서도 구현 가능하다. 
	```java
	public in~terface SingerSonwriter extends Singer, Songwriter {
		...
	}~
	```
	같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의해야 하고, 속성 개수 만큼 조합 폭발이 일어날 것이다. 또한 매개변수 타입만 다른 메서드들을 수없이 많이 가진 거대한 클래스를 낳을 수도 있다. (거대한 계층구조에는 공통 기능을 정의한 타입이 없기 때문에)

#### 안전하고 강력한 functionality enhancements
* 타입을 추상 클래스로 정의하면? 그 타입에 기능을 추가하려면 상속뿐이다. 그러나 wrapper class 관용구와 함께 인터페이스를 활용하면 기능을 개선할 수 있는 강력한 수단이 된다. 

#### default method 
* 인터페이스 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해 편리하게 기능을 추가할 수 있다. 
* e.g. `Collection`의 `removeIf`
* 단, 디폴트 메서드로 제공할 때는 `@implSpec` javadoc 태그를 붙여 문서화해야 한다. (Item 19)
* 제약사항
	* `equals`, `hashCode`를 default method로 제공해서는 안 된다.

#### Skeletal implementation
* 인터페이스와 추상 골격 구현 클래스를 함께 제공하여 두 가지 장점을 모두 취하는 방식 
* 인터페이스: 타입 정의(+필요 시 디폴트 메서드), Skeletal Implementation 클래스(나머지 메서드 구현) - 골격 구현 클래스 확장만으로 인터페이스를 구현하는 데 필요한 일이 대부분 완성된다. `Template method pattern`이다.
* naming 관례: *Interface*가 이름이면 *AbstractInterface*로 짓는다. 
	* e.g. `AbtractCollection, AbstractSet, AbstractList`
* 구조상 골격 구현을 확장하지 못하는 처지라면? 인터페이스를 직접 구현해야 한다. 혹은 골격 구현 클래스를 우회적으로 이용할 수도 있다. 

```java
static List<Integer> intArrayAsList(int[] a) {
	Objects.requireNonNull(a);

	return new AbtractList<>() {
		@Override 
		public Integer get(int i) {
			return a[i]
		}
		...
	}
}
```
골격 구현의 힘을 보여준다. 이 예는 int 배열을 받아 Integer 인스턴스 리스트 형태로 보여주는 `Adapter`이기도 하다. 
추상 클래스처럼 구현을 도와주면서도 추상 클래스로 타입을 정의할 떄 따라오는 심각한 제약에서는 자유롭다. 

##### Simulated multiple inheritance
* 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다. wrapper class와 유사한 방식으로, 이 방식을 simulated multiple inheritance라고 부른다. 

```java
public class ConcreteClass implements MyInterface {
	private class InnerSkeleton extends AbstractMyInterface {
		@Override
		public void method() {
			// some logic
		}
	}

	private final MyInterface skeleton = new InnerSkeleton();

	@Override 
	public void method() {
		skeleton.method();
	}
	...
}
```

##### Skeletal implementation 작성
* Abstract Method: 인터페이스에서 다른 메서드들의 구현에 사용되는 기반 메서드를 선정 
* Default method: 기반 메서드들을 사용해 직접 구현 가능한 메서드
	* ***단, `equals`, `hashCode` 같은 `Object`의 메서드는 디폴트 메서드로 제공하면 안 된다.***
* 만약 인터페이스의 모든 메서드가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다. 골격 구현 클래스에는 필요하면 public이 아닌 필드와 메서드를 추가해도 된다. 
* 이는 기본적으로 상속해서 사용하는 것을 가정하므로 Item 19의 상속 설계 및 문서화 지침을 모두 따라야 한다. 

##### Simple implementation 
* 골격 구현의 작은 변종 
* e.g. `AbstractMap.SimpleEntry`


#### 정리
* 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 
* 골격 구현은 '가능한 한' 이넡페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. 인터페이스에 걸려 있는 구현상 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.

