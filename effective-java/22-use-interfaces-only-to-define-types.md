# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.03             |
| PART | 22. 인터페이스는 타입을 정의하는 용도로만 사용하라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.

## Item22. 인터페이스는 타입을 정의하는 용도로만 사용하라

### Interface 구현의 의미
* 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 이야기해주는 것
* ***인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할***

### Anti pattern
#### Constants interface anti pattern
* 상수 인터페이스로 사용: `static final` 필드로만 이루어진 인터페이스 
	```java
	public interface PhysicalConstants {
		static final double AVOGADROS_NUMBER = 6.022_140_857e23;
		static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
		...
	}
	```
	-> 정규화된 이름(qualified name)을 쓰는 것을 피하고자 인터페이스를 구현하곤 한다. 이는 인터페이스를 잘못 사용한 예다.
* 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. 즉, 이는 이 내부 구현을 클래스의 API로 노출하는 행위다. 클래스가 어떤 상수 interface를 사용하든 사용자에게는 아무런 의미가 없다. 오히려 혼란을 주거나 내부 구현에 해당하는 이 상수들에 종속되게 한다. 
* 다음 릴리스에서 상수들을 쓰지 않게 되더라도 바이너리 호환성을 위해 상수 인터페이스를 구현하고 있어야 하는 문제가 생긴다. 
* e.g. `java.io.ObjectStreamConstants`

##### 대안
* 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가 
	e.g. `Integer.MAX_VALUE`
* 열거 타입이 적합하다면 열거 타입으로 만들어 공개 (Item 34)
* 위의 경우가 아니라면 Instance화 할 수 없는 Utility class에 담아 공개 (Item 4)
	```java
	public class PhysicalConstants {
		private PhysicalConstants() {
			throw new AssertionError();
		}

		public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
		public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
	}
	```
	tip1) 숫자 리터럴의 밑줄: 리터럴 값에는 영향을 주지 않고 읽기 편하게 해준다. 
	tip2) 사용 시에는 클래스와 함께 사용하는 것이 좋지만, 빈번히 사용한다면 static import를 고려하자. 