# Today I Learned
| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2023.12.31             |
| PART | 15. 클래스와 멤버의 접근 권하을 최소화하라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.


## public class의 public fields
```java
public class PublicPoint {
  public double x;
  public double y;
}
```

내부 필드에 직접 접근할 수 있게 하면 캡슐화의 이점을 제공할 수 없다. (Item 15)
물론, `package-private` 클래스 혹은 `private` nested class라면 데이터 필드를 노출한다 해도 문제가 없다. 클래스 선언이나 사용 면에서 접근자 방식보다 훨씬 깔끔하다. 패키지 밖의 코드 수정 없이 데이터 표현 방식을 바꿀 수 있기 때문이다. 

### `public` 클래스의 필드를 직접 노출한 예
`java.awt.package.Point`와 `java.awt.package.Dimension` 클래스로, 이 클래스들을 흉내내지 말자. 

### 정리
1. `public` class는 절대 가변 필드를 직접 노출해서는 안 된다. 
2. 불변 필드라면 노출해도 덜 위험하지만 여전히 안심할 수 없다. 
3. `package-private`, `private` nested class에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다. 