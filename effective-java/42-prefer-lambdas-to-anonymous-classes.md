# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.19             |
| PART | 42. 익명 클래스보다는 람다를 사용하라  |

# 7장 Lambdas and Streams
* Java 8 - Functional interface, lambda, method reference 개념이 추가되면서 함수 개체를 더 쉽게 만들 수 있게 되었다. 
* 그리고 Stream API까지 추가되어 Data element의 시퀀스 처리를 라이브러리 차원에서 지원하기 시작했다. 
* 이 기능들을 효과적으로 사용하는 방법을 알아보자. 


## Item42. 익명 클래스보다는 람다를 사용하라

### Anonymous class instance as a function object (낡은 ver)
```java
final List<String> words = Arrays.asList("mango", "fineApple", "apple", "banana", "egg");  
Collections.sort(words, new Comparator<String>() {  
    @Override  
    public int compare(String s1, String s2) {  
        return Integer.compare(s1.length(), s2.length());  
    }  
});
```
* 예전에는 자바에서 함수 타입을 사용할 때 single abstract method를 가진 인터페이스(드물게는 추상 클래스)를 사용했다: `function object`
* 이는 특정 함수나 동작을 나타내는 데 썼으며 97년 JDK 1.1이 등장하면서 주로 익명 클래스(Item 24)를 수단으로 사용하였다. 
* 위 예제에서는 `Comparator` interface가 정렬을 담당하는 추상 전략이다.

#### 단점
* 익명 클래스 방식은 코드가 너무 길어서 자바는 함수형 프로그래밍에 적합하지 않았다. 


### Lambda Expression as function object 
* 이제 single abstract method interface는 functional interface라 불린다.
* 이를 lambda expression을 사용해 만들 수 있게 되었다. 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

#### Example 1. - Replaces annonymous class
```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));

// Comparator construction method
Collections.sort(words, Comparator.comparingInt(String::length));

// java 8 - List default method 
words.sort(Comparator.comparingInt(String::length));
```
* `s1, s2`: compiler가 문맥을 살펴 타입을 추론하였다. 상황에 따라 타입을 결정하지 못할 때는 명시해주어야 한다. (타입 추론 규칙은 매우 복잡하며, 잘 알지 못해도 상관없다.) 
* 타입을 명시해야 더 명확할 때를 제외하고는 람다의 모든 매개변수 타입은 생략하자.
* tip: 컴파일러가 타입을 추론하는 데 필요한 타입 정보 대부분을 제네릭에서 얻는다. 앞의 아이템과 이어지는 맥락이다. (Item 29, 30)


#### Example 2. Functional object 활용 사례
```java
public enum Operation {  
    PLUS("+", (x, y) -> x + y),  
    MINUS("-", (x, y) -> x - y),  
    TIMES("*", (x, y) -> x * y),  
    DIVIDE("/", (x, y) -> x / y);  
  
    private final String symbol;  
    // predefined functional interface in java.util.function 
    private final DoubleBinaryOperator op;  
  
    Operation(String symbol, DoubleBinaryOperator op) {  
        this.symbol = symbol;  
        this.op = op;  
    }  
  
    public double apply(double left, double right) {  
        return op.applyAsDouble(left, right);  
    }  
  
    @Override public String toString() { return symbol; }  
}
```
* 상수별 class body를 가지는 것보단 열거 타입에 인스턴스 필드를 두는 것이 낫다. (Item 34) 람다를 이용하면 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다. 
* **그러나 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야 하는 상황이라면 상수별 class body를 사용하자.** 
	* 열거 타입 생성자에 넘겨지는 인수들의 타입은 컴파일 타임에 추론된다. 인스턴스는 런타임에 만들어지므로 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다.

#### 단점
* 메서드나 클래스와 달리 람다는 이름이 없고 문서화도 못한다. 코드 자체로 동작히 명확히 설명되지 않거나 코드 라인 수가 많아지면 람다를 쓰지 말아야 한다. - 한 줄 일 떄 가장 좋고 길어야 세 줄 안에 끝내는 게 좋다.
* 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링하자.


### Few things with anonymous class
* 람다는 함수형 인터페이스에서만 쓰인다. 따라서 아래 케이스의 경우에는 익명 클래스를 써야 한다. (즉, 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때)
	* 추상 클래스의 인스턴스를 만들 때 
	* 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때
* 람다는 자신을 참조할 수 없다. `this`는 바깥 인스턴스를 가리킨다. 따라서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다. 


### 유의사항
* 람다도 익명 클래스처럼 직렬화 형태가 구현별로(가령 가상머신별로) 다를 수 있다. 따라서 람다를 직렬화하는 일은 극히 삼가야 한다. (익명 클래스도 마찬가지다.)
* 직렬화해야만 하는 함수 객체가 있다면(e.g. `Comparator`) private static nested class(Item 24)의 인스턴스를 사용하자. 