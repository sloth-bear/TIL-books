# Today I Learned

| 구분 | 내용                        |
| ---- | --------------------------|
| DATE | 2024.01.26                |
| PART | 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라 |

# 8장 Method
* 이번 장에서는 메서드를 설계할 때 주의할 점들을 살펴본다.
* 구체적으로는 매개변수와 반환값을 어떻게 처리해야 하는지, 메서드 시그니처는 어떻게 설계해야 하는지, 문서화는 어떻게 해야 하는지를 다룬다. 
* 상당 부분은 메서드뿐 아니라 생성자에도 적용된다. 
* 사용성, 견고성, 유연성에 집중할 것이다.

## Item54. `null`이 아닌, 빈 컬렉션이나 배열을 반환하라

### Don't do this!
```java
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```
* 재고가 없다고 해서 특별히 취급할 이유가 없다. 
* 거기다 클라이언트는 이 null 상황을 처리하는 코드를 추가적으로 생성해야 한다. 


### 빈 컨테이너 할당 비용
* 때로는 빈 컨테이너 할당 비용으로 `null`을 반환하는 쪽이 낫다는 주장도 있다. 그러나 이것은 두 가지 면에서 틀린 주장이다. 
* 첫 번째, 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못 된다.
* 두 번째, 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. 
    * e.g.1 `new ArrayList<>(cheeseInStock);` - 대부분의 경우
    * e.g.2 `Collection.emptyList()` - 매번 같은 빈 '불변' 컬렉션을 반환한다. 단, 이 역시 최적화에 해당하니 꼭 필요할 때만 사용하자. 최적화가 필요하다고 판단되면 수정 전과 후의 성능을 측정하여 실제로 성능이 개선되는지 꼭 확인하자. 

#### 배열의 경우 
* 절대 null을 반환하지 말고 길이가 0인 배열을 반환하라. 
    * 보통 단순히 정확한 길이의 배열을 반환하기만 하면 된다. 길이는 0일 수도 있다. 
        ```java
        public Cheese[] getCheeses() {
          return cheesesInStock.toArray(new Cheese[0]);
        }
        ```
      * 성능을 떨어뜨릴 것 같다면? - cheesesInStock이 비어있을 때면 언제나 EMPTY_CHEESE_ARRAY를 반환한다.
        ```java
        private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

        public Cheese[] getCheeses() {
          return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
        }
        ```

##### Anti pattern
* 단순 성능 개선 목적이라면 toArray에 넘기는 배열을 미리 할당하는 것은 추천하지 않는다. 오히려 성능이 떨어질 수 있다. 
```java
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```