# Today I Learned

| 구분 | 내용                        |
| ---- | --------------------------|
| DATE | 2024.02.02                |
| PART | 58. 전통적인 for 문보다는 for-each 문을 사용하라 |

# 10장 일반적인 프로그래밍 원칙
* 자바 언어의 핵심 요소에 집중
* 지역변수, 제어구조, 라이브러리, 데이터 타입
* 리플렉션과 네이티브 메서드
* 최적화와 명명 규칙

## Item58. 전통적인 for문 보다는 for-each 문을 사용하라

### Traditional `for` loop
#### Iterate
```java
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
  Element e = i.next();
  ... // do something with e
}
```

#### `for` loop
```java
for (int i = 0; i< a.length; i++) {
 ... // do something with a[i]
}
```

#### 단점
* 반복자, 인덱스 변수: 모두 코드를 지저분하게 하고 사용되지 않을 수 있다. 
* 쓰이는 요소 종류가 늘어나면 오류 가능성이 높아진다. 
* 잘못된 변수를 사용했을 때 컴파일러가 잡아주리라는 보장도 없다. 
* 컬렉션이냐 배열이냐에 따라 코드 형태가 상당히 달라지므로 주의해야 한다. 


#### for-each loop: Enhanced for statement
```java
for (Element e : elements) {
  ... // do something with e
}
```
* 콜론(:)은 안의(in)라고 읽으면 된다. 
* "`elements` 안의 각 원소 `e`에 대해" 라고 읽는다. 

#### 장점
* 반복자, 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다. 
* 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지는 신경 쓰지 않아도 된다. 

### for-each 문 사용 불가한 경우 
* 아래 3가지 케이스의 경우 일반적인 `for` 문을 사용하되 이번 아이템에서 언급한 문제들을 경계하기 바란다. 
* for-each 문은 컬렉션과 배열은 물론 `Iterable` 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다. 
* `Iterable`을 처음부터 직접 구현하기는 까다롭지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면 `Iterable`을 구현하는 쪽으로 고민해보기 바란다. 해당 타입에서 `Collection` 인터페이스는 구현하지 않기로 했더라도 말이다. 

#### 파괴적인 필터링 (destructive filtering)
* 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다. 
* 자바 8부터는 `Collection`의 `removeIf` 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다. 

#### 변형 (transforming)
* 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

#### 병렬 반복 (parallel iteration)
* 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다. 