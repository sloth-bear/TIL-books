# Today I Learned
| 구분  | 내용               |
|------|-------------------|
| DATE | 2022.05.21        |
| PART | 5. Bend, or Break |


## 1. 기억하고 싶은 내용
현대의 미친 듯이 빠른 변화 속도를 따라가려면 모든 수단을 동원하여 가능한 한 느슨하고 유연한 코드를 작성해야 한다. 그렇지 않으면 코드는 금세 낡고 수정하기 어려워지고, 결국 기억 저편으로 사라질 것이다. 유연함을 유지하는 한 가지 좋은 방법은 물론 가능한 한 코드를 적게 작성하는 것이다. 코드 수정은 새로운 버그가 생기는 계기이기도 하다. <1.5. 설정>에서는 세부 사항을 완전히 코드 밖으로 옮기는 방법에 대해 설명한다. 그렇게 하면 세부 사항을 안전하고도 쉽게 변경할 수 있다.

### 1.1. 결합도 줄이기 
* 좋은 설계 원칙을 따르면 바꾸기 쉬운 코드를 만들 수 있다고 주장했다. 높은 결합도는 변경의 적이다. 다리를 설계할 때는 그 형태가 바뀌지 않기를 바랄 것이다. 따라서 구조가 단단해야 한다. 하지만 소프트웨어를 설계할 때는 언젠가 형태를 바꾸려 할 것이다. 바라는 것이 정확히 반대다. 소프트웨어의 구조는 유연해야 한다. 그리고 유연하려면 각각의 부품이 다른 부품에 가능한 한 조금만 연결되어야 한다. 

* 난감한 부분은 결합이 추이적`transitive`이라는 것이다. 즉 <strong>A가 B, C와 결합</strong>되어 있고, <strong>B는 N, M과</strong>, <strong>C는 X, Y와 결합</strong>되어 있다면, 결국 A는 B, C, M, N, X, Y 모두와 결합되어 있는 것이다. 그러니 다음 간단한 원칙을 따라야 한다. `결합도가 낮은 코드가 바꾸기 쉽다.` 

* 또한 여러분의 코드에서 나타나는 다음과 같은 결합의 증상을 놓치지 않도록 주의해야 한다. 
  * 관계없는 모듈이나 라이브러리 간의 희한한 의존 관계
  * 한 모듈의 '간단한' 수정이 이와 관계 없는 모듈을 통해 시스템 전역으로 퍼져 나가거나 시스템의 다른 곳에서 무언가를 깨뜨리는 경우 
  * 개발자가 수정하는 부분이 시스템에 어떤 영향을 미칠지 몰라 코드의 수정을 두려워하는 경우 
  * 변경 사항에 누가 영향을 받는지 파악하고 있는 사람이 없어서 결국 모든 사람이 참석해야 하는 회의 

#### 열차 사고`train wreck`: 연쇄 메서드 호출 
* 다음과 같은 식의 코드를 본 적이 있을 것이다. 
    ```java
    // totals 객체의 내부 상태에 따라 판단을 내리고 해당 객체를 갱신함. 
    public void applyDiscount(customer, order_id, discount) {
        totals = customer
                    .orders // customer 객체에 주문 컬렉션을 묻기 
                    .find(order_id)
                    .getTotals();

        // 주문 객체 구현할 때 합계를 별도의 객체에 저장하는 것을 노출, 내부 상태 갱신
        totals.grandTotal = totals.grandTotal = discount; 
        totals.discount = discount;
    }
    ```java
    이 코드는 customer에서 total까지 다섯 단계의 추상화를 오간다. 결국 최상위 코드가 모든 것을 알아야 한다. 설상가상으로 이 코드를 계속 지원하기 위해서 앞으로 바꾸면 안 되는 것도 너무 많다. 기차의 모든 객차가 서로 연결되어 있듯이 메서드나 속성들이 모두 연결되어 있다. 이런 코드를 '열차 사고'라고 부른다. 이를 고치려면 `묻지 말고 말하라(Tell, Don't Ask. TDA.)` 원칙을 적용해야 한다. (묻는다 = 객체의 내부 상태에 따라 '내가' 판단하고 그 객체를 갱신한다.)

    * 할인 처리 -> total 객체에 위임, 고객 객체에서 바로 주문 객체 얻기, 합계를 별도의 객체에 저장했단 사실을 굳이 보여주지 않기
        ```
        public void applyDiscount(customer, order_id, discount) {
            customer
                .findOrder(order_id) // 고객 객체 > 주문 객체 바로 얻기
                .applyDiscount(discount); // 할인 처리 위임, 합계 캡슐화
        }
        ```

* 데메테르 법칙`Law of Demeter, LoD`: 메서드 호출을 엮지 말라. `.`을 딱 하나만 쓰려고 노력해 보라.

* 함수를 조합하여 파이프라인을 만드는 것은 메서드 호출로 이루어진 열차 사고와는 다르다. 숨겨진 구현 세부 사항에 의존하지 않기 때문이다. 그렇다고 파이프라인이 결합을 하나도 만들지 않는 것은 아니다. 파이프라인의 함수에서 반환하는 데이터는 반드시 다음 함수가 처리할 수 있는 형식이어야 한다. 이런 파이프라인 형태의 결합은 열차 사고로 인한 결합에 비해 코드를 바꿀 때 문제가 생기는 경우가 적었다. 

#### 글로벌화: 정적인`static` 것의 위험함
* 전역`global` 데이터는 교묘하게 애플리케이션 컴포넌트 간의 결합을 만들어 낸다. 이를 변경할 때 시스템 코드 전체에 영향을 줄 수 있음은 분명하지만 실제 상황에서 그 파급 효과는 제한적이다. 문제는 바꿔야 하는 곳을 모두 바꿧는지 확인하기 힘들다는 데 있다.

* 코드 재사용의 장점은 많이 알려져 있다. 코드를 처음 작성하는 시점의 제 1 관심사가 코드 재사용이어서는 안 될 것이다. 하지만 코드를 재사용할 수 있도록 해야 한다는 생각이 코딩 습관의 일부가 되어야 한다. 

* `singleton`도 전역 데이터다. 

* 외부 리소스도 전역 데이터다. Database, 저장소, 파일 시스템, 서비스 API 등을 사용한다면 전역 데이터의 함정에 빠질 위험이 있는 것이다. 여기서도 해법은 반드시 이 리소스들을 여러분이 작성하는 코드로 모두 감싸는 것이다. 

### 상속: 왜 클래스 상속이 위험한가? 
* `1.4. 상속세`에서 설명한다.

### 1.2. 실세계를 갖고 저글링하기 
#### 이벤트에 반응하는 네 가지 서로 다른 전략 
* 이벤트는 무언가 정보가 있다는 것을 의미한다. 애플리케이션을 이런 이벤트에 반응하도록, 그리고 그에 기반해서 하는 일을 조절하도록 만들면, 진짜 세상에서 더 잘 작동하는 애플리케이션이 탄생할 것이다. 사용자들은 애플리케이션의 상호 작용이 더 원활하다고 느낄 것이고 애플리케이션 자체는 리소스를 더 효율적으로 사용할 것이다. 

* 어떻게 이벤트에 잘 반응하는 애플리케이션을 만들 수 있을까? 
  * 유한 상태 기계
  * 감시자`observer` 패턴
  * publish-subscribe
  * 반응형 프로그래밍과 stream (대부분의 클라우드 서비스가 제공한다.)

### 1.3. 변환 프로그래밍 
모든 프로그램은 데이터를 변환한다. 받은 입력을 출력으로 바꾼다. 하지만 우리는 설계를 고민할 때 변환을 만드는 것에 대해서는 거의 생각하지 않는다. 오직 클래스와 모듈, 자료 구조, 알고리즘, 언어, 프레임워크에 대해서만 걱정할 뿐이다. 이렇게 코드에만 집중하면 핵심을 놓칠 수 있다고 본다. 프로그램이란 입력을 출력으로 바꾸는 것이라는 사고방식으로 돌아갈 필요가 있다. 

#### Function pipeline
* 입력을 출력으로 바꿔 가는 단계들을 찾는다. 일종의 '하향식`top-down`' 접근 방식이다. 

* 요구 사항을 달성하기 위해 필요한 것은 하나로 연결된 변환들 뿐이다. 각각은 앞의 변환에서 입력을 받아 처리한 결과를 다음 변환으로 넘겨준다. 이보다 글처럼 읽기 쉬운 코드는 만들기 어려울 것이다. 더 깊은 의미도 있다. 객체 지향 프로그래밍 경험이 많다면 반사적으로 데이터를 숨기고, 객체 안에 캡슐화해야 한다고 느낄 것이다. 이런 객체들은 서로 이리저리 이야기하며 서로의 상태를 변경한다. 이런 방식은 결합을 많이 만들어 내고, 이는 결국 객체 지향 시스템이 바꾸기 어려워지는 큰 요인이 된다. `상태를 쌓아 놓지 말고 전달하라.`

* 오류 처리는 어떻게 할까? 연쇄 변환이 일직선으로만 이어진다면 어떻게 오류 검사에 필요한 조건부 논리를 추가할 수 있을까? 여러 가지 방법이 있지만 공통으로 사용되는 기본적인 관례가 하나 있다. 바로 변환 사이에 값을 절대 날것으로 넘기지 않는 것이다. 대신 wrapper 역할을 하는 자료 구조나 타입으로 값을 싸서 넘긴다. 이런 자료 구조나 타입은 안에 들어 있는 값이 유효한지를 추가로 알려준다. 예를 들어 하스켈에서는 이런 래퍼를 `Maybe`라고 부르고 F#과 스칼라에서는 `Option`이다. 

* 오류가 발생했을 때 파이프라인을 더 이상 실행하고 싶지 않고, 변환 코드들은 무슨 일이 일어나는지도 모르게 하고 싶다는 것이다. 그 말은 파이프라인에 있는 함수들의 실행을 앞 단계가 성공적으로 끝난 것을 확인할 때까지 미루어야 한다는 뜻이다. 따라서 우리는 함수 호출을 나중에 호출할 수 있는 값 형태로 바꿔야 한다.

### 1.4. 상속세
* 객체 지향 언어로 프로그래밍하는가? 상속을 사용하는가? 그렇다면 멈춰라! 아마 여러분에게 필요한 것은 상속이 아닐 것이다. 

* 상속도 일종의 결합이다. 자식 클래스가 부모 클래스(부모의 부모...)에게 연결되는 것은 물론이요, 자식 클래스를 사용하는 코드도 이 클래스의 모든 조상과 얽히게 된다. 

* 더 나쁜 것은 다중 상속 문제다. `Car`는 `Vehicle`의 일종일 수 있는데, 동시에 `Asset(자산)` 등의 일종일 수도 있다. 제대로 모델링하려면 다중 상속이 필요할 것이다. C++는 1990년대에 다중 상속에 오명을 씌웠다. C++의 모호성 해소 방식에 의심스러운 부분들이 있었기 때문이다. 그 결과 이제는 많은 객체 지향 언어에서 다중 상속을 지원하지 않는다. 따라서 아무리 복잡한 클래스 계층도가 마음에 들더라도 어차피 여러분의 도메인을 정확하게 모델링할 수는 없다. `상속세를 내지 말라.`

* 더는 상속을 쓸 필요가 없게 해주는 세 가지 기법은 `인터페이스와 프로토콜`, `위임`, `믹스인과 트레이드`다. 

* 인터페이스와 프로토콜: 대부분의 객체 지향 언어는 클래스가 특정한 동작을 구현한다고 지정할 수 있다. 여러 동작을 지정할 수도 있다.
    ```java
    public class Car implements Drivable, Locatable {
        // Drivable, Locatable이 요구하는 기능 모두 구현
    }
    ```
    `Drivable`, `Locatable` 같은 것을 자바에서 `interface`라고 부른다. `protocal`이라고 부르는 언어도 있고, `trait`라고 부르는 언어도 있다. 

* 인터페이스나 프로토콜이 강력한 까닭은 이들을 타입으로 사용할 수 있고, 해당 인터페이스를 구현하는 클래스라면 무엇이든 그 타입과 호환되기 때문이다. 상속 없이도 다향성을 가져다준다.

* 상속은 개발자들이 점점 더 메서드가 많은 클래스를 만들도록 유도한다. 부모 클래스에 메서드가 20개 있으면 하위 클래스는 그 중 딱 두 개만 사용하고 싶더라도 필요 없는 18개의 메서드까지 함께 따라와서 자리르 차지하고 호출되기만을 기다린다. 대신 위임`delegation`을 사용하면 어떨지 생각해 보라. 서비스에 위임하라. Has-A가 Is-A보다 낫다. (디자인 패턴에서도 객체 지향 설계의 주요 원칙으로 상속보다 객체 합성`composition`을 꼽고, 이펙티브 자바에서도 `상속보다는 컴포지션을 사용하라`고도 한다.)

* 믹스인`mixin`, 트레이드`trait`, 카테고리`category`, 프로토콜 확장`extension` 등은 이 기능을 구현했을 때 기존의 것과 새로운 것의 기능 집합을 합칠 수 있게 된다. `믹스인으로 기능을 공유하라.` 

* 상속이 답인 경우는 드물다. 상황에 따라 더 나은 방법들이 있을 것이다. 타입 정보를 공유하고 싶은 건지, 기능을 더하고 싶은 건지, 메서드를 공유하고 싶은 건지에 따라 다르다. 정글 전체를 끌어들이지 않도록 조심하라.

### 1.5. 설정
* 애플리케이션이 출시된 이후 바뀔 수도 있는 값에 코드가 의존하고 있다면 그 값을 애플리케이션 외부에서 관리하라. 

* 일반적으로 설정 데이터 안에 넣는 것은 다음과 같다. 
  * 데이터베이스나 외부 API 같은 외부 서비스의 인증 정보 
  * 로그 레벨과 로그 저장 위치 
  * 애플리케이션이 사용하는 포트 번호, IP 주소, 기계나 클러스터 이름
  * 특정 실행 환경에만 적용되는 검증 매개 변수 
  * 외부에서 지정하는 매개 변수, 예를 들어 배송비 
  * 지역에 따른 세부 서식 
  * 라이선스 키 

* 정적`static` 설정: 대부분의 프레임워크 그리고 상당수의 애플리케이션이 설정을 일반 파일이나 데이터베이스 테이블로 관리한다. 정보를 일판 파일로 관리할 때는 널리 쓰이는 텍스트 형식을 사용하는 추세다. 2021년 기준으로는 YAML과 JSON이 가장 많이 쓰인다. 

* 서비스형 설정`Configuration-As-A-Service`: 일반 파일이나 데이터베이스가 아니라 서비스 API 뒤에서 관리하는 것이다. 여러 애플리케이션이 설정 정보를 공유할 수 있고, 전체 설정을 한 번에 바꿀 수 있으며, 전용 UI로도 관리할 수 있다. 동적으로도 계속 바꿀 수 있다. 

## 2. 소감
결합도와 상속 관련해서는 평소 보던 코드들이 많다보니 조금 더 와닿게 느껴졌다. 최근에 어떤 API를 구현하기 위해 Template pattern으로 상속을 구현한 적이 있었는데, 요구사항이 바뀌면서 부모가 조금씩 덩치가 커짐을 느꼈고, 분기문이 자꾸 생기게 되면서 뭔가 구조가 잘못된 것 같다는 생각이 들어서 리팩토링의 필요성을 느꼈다. 더 수정이 어려워지기 전에 말이다. 최근 추가된 요구사항이 부모 클래스를 더 변경해야 되는 부분이 있기 때문에 리팩토링의 시점이 온 것 같다. 미리 테스트 코드를 만들어두지 않은 것이 후회로 좀 남는다.

상속 대신 위임하라는 것은 알고 있었지만 생각보다 상속 대신 활용할 수 있는 방법과 그 케이스들이 다르다는 점도 알게 되었다. 관련 케이스들을 이해하기 위해서는 그 기법을에 대한 이해도가 조금 더 높아져야 할 것 같다. 

또한 oop와 fp는 각자의 이점들이 있기 때문에, 꼭 oop가 나쁜가? 혹은 fp가 oop를 대체해야 하는가?에 대해서는 생각해볼 여지가 있는 것으로 알고 있다. 적절히 섞어서, 잘 사용해야 한다는 이야기도 들었다. oop는 확실히 어려운 것 같다. 이 책에서 말하듯 결합도가 많이 높아진다. 숙련도의 문제라고 생각했는데, 과연 oop의 난이도는 얼마나 되는 것일까? 코드를 직접 구현하는 연습을 더 많이 해야 하고, 또 피드백도 많이 받아야겠다는 생각이 다시 한 번 자리잡는다. 배움에는 정말.. 끝이 없다. 이벤트 관련 부분은 사실 이해가 어렵기도 했고 피상적으로 와닿는 부분들이 많았다. 그 외로도 이번 챕터는 궁금한 점들이 많았는데, 단번에 이해하기엔 양이 좀 방대하다 생각된다. 한 번 더 읽으면서 이해도가 높아지면 관련 부분들을 조금씩 더 찾아봐야 할 것 같다. 

## 3. 궁금한 내용
* 유한 상태 기계(FSM)
* Observer pattern 
* 하스켈의 `Maybe`, F#과 스칼라에서는 `Option`. Java에서는 `Optional`일까? 
* C++의 다중 상속과 모호성 해소 방식의 의심스러운 부분은 무엇인가? 
* 인터페이스를 구현하더라도 점점 더 메서드가 많은 클래스를 만들도록 유지하게 되지 않는가? 
* 상속보다는 위임이 무조건 좋은가? 상속이 좋을 때는 없는가?