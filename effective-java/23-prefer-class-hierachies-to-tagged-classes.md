# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.03             |
| PART | 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.

## Item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### Tag class
두 가지 이상의 의미를 표현할 수 있고, 그 중 현재 표현하는 의미를 태그 값으로 알려주는 클래스 
```java
class Figure {
	enum Shape { RECTANGLE, CIRCLE };

	// 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다. 
	double length; 
	double width; 

	// 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다. 
	double radius;

	// 원용 생성자 
	Figure(double radius) { 
		shape = Shape.CIRCLE; 
		this.radius = radius; 
	} 
	
	// 사각형용 생성자 
	Figure(double length, double width) { 
		shape = Shape.RECTANGLE; 
		this.length = length; 
		this.width = width; 
	} 
	
	double area() { 
		switch(shape) { 
			case RECTANGLE: 
				return length * width; 
			case CIRCLE: 
				return Math.PI * (radius * radius); 
			default:throw new AssertionError(shape); 
		}
	} 
}
```
* 예제에도 주석을 되도록 뺐지만, 이 클래스는 주석을 사용할 수밖에 없었다. 주석이 없었다면 위 필드가 각 Shape 별로 사용된단 것을 알 수 있었을까? 내부 구현을 분석해야 했을 것이다. 
* 위 예제의 단점
	* 열거 타입 선언, 태그 필드, switch 문(예제에서 생략했음) 등 쓸데 없는 코드가 많아짐 
	* 여러 구현이 한 클래스에 혼합되어있어 가독성이 떨어짐 
	* 다른 의미를 위한 코드도 언제나 함께 하니 추가적인 메모리 사용 
	* 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화 해야 함 (불필요한 코드)
	* 각 타입의 필드들 초기화 시에 컴파일 에러로 잡기 어렵다 
	* 새로운 의미를 추가할 때마다 모든 switch 문을 찾아 새 의미를 처리하는 코드를 추가해야 하며, 하나라도 누락되면 런타임 에러가 발생한다 
	* 즉, 태그 달린 클래스는 장황하고 오류를 내기 쉽고 비효율적이다 

태그 달린 클래스는 클래스 계층 구조를 어설프게 흉내낸 아류일 뿐이다.


### 클래스 계층 구조 
#### Tag class -> 클래스 계층 구조 전환 방법
1. 계층 구조의 root가 될 추상 클래스 정의
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언
3. 태그 값에 상관 없이 동작이 일정한 메서드를 루트 클래스에 일반 메서드로 추가 
4. 공통 데이터 필드 - 루트 클래스에 추가 
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의 

#### 예시 
참고) 코드 단순화를 위해 접근자 메서드 없이 필드를 직접 노출 것 
```java
abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;

	Circle(double radius) {
		this.radius = radius;
	}

	@Override
	double area() {
		return Math.PI * (radius * radius);
	}
}

class Rectangle extends Figure {
	final double length;
	final double width;

	Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}

	@Override
	double area() {
		return length * width; 
	}
}
```
* 간결하고 명확해졌으며 쓸데 없는 코드가 모두 사라졌다.
* 살아남은 필드는 모두 final이다. 
* 모든 필드를 남김없이 초기화했는지, 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다. 
* 루트를 건드리지 않고도 독립적으로 계층구조를 확장하고 함께 사용할 수 있다. 


### References
* [hierarchical-inheritance-in-java](https://technogeekscs.com/hierarchical-inheritance-in-java/
