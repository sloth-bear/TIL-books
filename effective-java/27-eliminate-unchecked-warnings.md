# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.08             |
| PART | 27. 비검사 경고를 제거하라 |

# 5장 Generic
* 자바 5부터 지원된 generic - 지원 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다. 
* 제네릭 사용 시 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주게 된다: 컴파일 타임에서 형변환 오류를 차단하므로 더 안전하고 명확한 프로그램을 만들어준다. 
* 컬렉션이 아니더라도 관련 이점을 누릴 수 있다. 
* 이번 장에서는 제네릭의 이점을 최대로 살리고 단점을 최소화하는 방법을 이야기한다.

## Item27. 비검사 경고를 제거하라

### 할 수 있는 한 모든 Unchecked warning을 제거하라.
* e.g. 아래 코드는 Unchecked warning이 생길 것이다. 다이아몬드 연산 추가(`new HashSet<>();`)를 통해 제거하자.
  ```java
  Set<Lark> exaltation = new HashSet(); 
  ```

* 경고를 제거할 수 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아서 경고를 숨기자. 타입 안정성을 검증하지 않은 채 경고를 숨기면 안 된다. 이는 가능한 좁은 범위에 적용하자. (절대 클래스 전체에 적용하면 안 된다.) 그리고 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다. 다른 사람이 잘못 수정하여 타입 안전성을 잃는 상황을 줄여준다. 
  ```java
  public <T> T[] toArray(T[] a) {
    if (a.length < size) {
      @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
      return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
      a[size] = null;
    return a;
  }
  ```

#### 장점
* 타입 안정성이 높아진다. (런타임에 `ClassCastException`이 발생할 일이 없고 의도대로 동작할 것이라 확신할 수 있다.)

