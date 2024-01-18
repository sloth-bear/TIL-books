# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.18             |
| PART | 39. 명명 패턴보다 애너테이션을 사용하라 |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item39. 명명 패턴보다 애너테이션을 사용하라 

### 명명 패턴
1. 전통적으로 tool, framework가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다. e.g. JUnit3: 테스트 메서드명을 test로 시작하게끔 하였다. -> Junit4부터 애너테이션 도입
2. 여기에는 단점이 있다. 
  1. 오타가 나면 안 된다. 
  2. 올바른 프로그램 요소에만 사용되리라 보증할 방법이 없다. e.g. `TestSafetyMechanisms`라는 클래스를 만들어 JUnit에 던져주어도, JUnit은 클래스 이름에는 관심이 없다. 테스트는 수행되지 않는다. 
  3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.


### 애너테이션
#### 종류 
* meta annotation: 애너테이션 선언에 쓰이는 애너테이션
  * e.g. `Retention(RetentionPolicy.RUNTIME)`
* marker annotation: 아무 매개변수 없이 단순히 대상에 마킹(marking)한다.


#### Example
```java
public class Sample {
  @Test public static void m1() {}
}
```
* `@Test` 애너테이션이 `Sample` 클래스의 의미에 직접적인 영향을 주진 않지만, 이 애너테이션에 관심 있는 프로그램에게 추가 정보를 제공한다. 즉, 코드의 의미는 그대로 두지만, 이 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 준다. 