# Today I Learned
| 구분  | 내용                                 |
|------|-------------------------------------|
| DATE | 2022.05.23                          |
| PART | 4. Pragmatic Paranoia - exercise 14 |


# 연습문제 14
## 문제
부엌용 믹서의 인터페이스를 설계하라. 이 믹서는 나중에 웹으로 쓸 수 있도록 사물인터넷`IoT`으로 연결될 예정이지만, 지금으로서는 믹서를 제어할 수 있는 인터페이스만 있으면 된다. 속도를 열 단계로 설정할 수 있는데 0은 중지를 의미한다. 비어 있는 상태에서는 작동할 수 없고, 한 번에 한 칸씩만 속도를 바꿀 수 있다. 즉 0에서 1, 1에서 2는 되지만, 0에서 2는 안 된다. 다음과 같은 메서드가 있을 때, 적절한 선행, 후행 조건과 불변식을 추가하라. 
```
int getSpeed()
void setSpeed(int x)
boolean isFull()
void fill()
void empty()
```


## 풀이과정
### Library
`Cofoja`라는 라이브러리를 사용했다. 
- `@Requires` 메서드를 위한 preconditions
- `@Ensures` 메서드를 위한 postconditions
- `@Invariant` 클래스와 인터페이스를 위한 invariants

### Implements 
* 일단 `Blender` 인터페이스를 작성했다. 
    ```
    public interface Blender {
        
        int getSpeed();
        
        void setSpeed(int x);

        boolean isFull();

        void fill();

        void empty();

    }
    ```

* 이 객체가 동작했다는 것은 비어있는 상태가 아니라는 것이다. 이는 불변이다. 
    ```
    @Invariant("isFull()")
    public interface Blender { ... }
    ```

* 이 객체가 동작했다는 것은 속도가 0 초과, 10 이하라는 것이다. 이는 불변이다. 
    ```
    @Invariant({ "isFull()", "getSpeed() > 0 && getSpeed() < 11" })
    public interface Blender { ... }
    ```

* 속도 설정 시 한 번에 한 칸씩만 속도를 바꿀 수 있다. 모듈을 호출하기 위한 조건이다. 
    ```
    ...
    @Requires("x == getSpeed() - 1 || x == getSpeed() + 1")
    void setSpeed(int x) { ... }
    ...
    ```

* 속도 설정 시 속도는 0보다 내려갈 수 없고 10까지 가능하다. 마찬가지로 호출 조건이다. 
    ```
    ...
    @Requires({ "x == getSpeed() - 1 || x == getSpeed() + 1", "x > 0", "x < 11 })
    void setSpeed(int x) { ... }
    ...
    ```

* 속도 설정 시 속도를 설정하고 나면 속도가 해당 값이 된다. 
    ```
    ...
    @Requires({ "x == getSpeed() - 1 || x == getSpeed() + 1", "x > 0", "x < 11 })
    @Ensures("getSpeed() == x")
    void setSpeed(int x) { ... }
    ...
    ```

* 채우고 나면 full 상태가 된다. 
    ```
    @Ensures("isFull()")
    void fill() { ... }
    ```

* 비우고 나면 full 상태가 아니다. 
    ```
    @Ensures("!isFull()")
    void empty() { ... }
    ```

* 종합해보았다. 
    ```
    @Invariant({ "isFull()", "getSpeed() > 0 && getSpeed() < 11" })
    public interface Blender {

        int getSpeed();
        
        @Requires({ "x == getSpeed() - 1 || x == getSpeed() + 1", "x > 0", "x < 11 })
        @Ensures("getSpeed() == x")
        void setSpeed(int x);

        boolean isFull();

        @Ensures("isFull()")
        void fill();

        @Ensures("!isFull()")
        void empty();

    }
    ```


## 오답풀이
* 책에서는 불변식을  `@invariant getSpeed() > 0 implies isFull()`로 표현하였다. 찾아보니까 속도가 0 이상이면 실행할 수 있는데, 그런다음 무조건 채워져있기도 해야 한다는 뜻 같다. `&&` 조건과 큰 차이가 없는 것 같은데, 대체해도 되는 것인지 궁금해졌다. 

* 나는 속도 유효 범위를 `"x == getSpeed() - 1 || x == getSpeed() + 1"`로 설정하였지만 책에서는 절대값을 이용하였다. 조금 더 효율적인 것 같다. 

* 그리고 책에서는 `fill()`, `empty()` 메소드에서 선행조건으로 각각 채워져있지 않을 것, 비워져있지 않을 것을 명시하였다. 조금 더 명확하게 호출자에게 요구사항을 전달한 것 같다. 


## Concepts
* 1986, `Object-Oriented Software Construction`, 버트란드 마이어(Eiffel 개발자)
* 프로그램 <u>모듈들의 책임</u>을 <u>문서화</u>하는데 초점
* 계약`Contract`
  * 만약 호출자가 모듈(루틴)의 모든 선행 조건을 충족한다면, 해당 모듈은 종료 시 모든 후행조건과 불변식이 참이 될 것을 보증해야 한다. 
* 선행조건`Precondition`
  * 모듈을 호출하기 위해 참이어야 하는 것 
* 후행조건`Postcondition`
  * 모듈이 자기가 할 것이라고 보장하는 것
* 불변식`Invariant`
  * 호출자의 입장에서 이 조건이 언제나 참이라고 보장하는 것
  * 모듈이 내부 처리 중에는 불변식이 참이 아닐 수도 있지만, 모듈이 종료하고 호출자로 제어권이 반환되는 때에는 참이 되는 것 


## Supports
* `C/C++`: `Nana`를 통해 지원
* `Eiffel`: 지원
* `Java`: `iContarct`, `Contract4J`, `c4j`, `Cofoja` 등을 통해 지원


## 소감
Spring으로 API를 설계할 때 java validation api를 사용하는 것처럼, 어떤 작업을 하기 전에 계약조건들을 먼저 설정하고 설계하는 것이 새롭게 다가왔다. 이렇게 하면 기계처럼 코드를 작성해나가지 않으므로 좀 더 객체의 책임이 분명해질 수 있을 것 같다. (사실 책임이 분명해야 불변식을 제대로 작성할 수 있을 것 같다. 내가 전후를 바꿔 얘기한 것인지도 모른다.) 그리고 작성하면서 느낀 건데, TDD와 비슷한 것 같으면서도 뭔가 다른 것 같다는 생각이 들었다. TDD는 case 별로 어떤 시나리오들을 정리해나가는 느낌이라면 DBC는 내 Interface는 이렇다는 것을 호출자에게 안내해주는 느낌이라고 해야 할까? 더불어서 Java와 DBC 관련 자료가 많진 않았는데, 개발자들이 별로 선호하지 않는 것인지? 혹은 필드에서 잘 쓰이지 않는 것인지 궁금하기도 했다. (Java와 JDBC로 검색결과가 더 많이 나온다.. 때문에 Designed by contract로 검색했다.) 


## References
* [계약에 의한 설계(Design by Contract; DBC](https://kevinx64.net/198)
* [DESIGN BY CONTRACT IN JAVA WITH GOOGLE](https://objectcomputing.com/resources/publications/sett/september-2011-design-by-contract-in-java-with-google)
