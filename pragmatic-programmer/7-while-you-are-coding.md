# Today I Learned
| 구분  | 내용                     |
|------|-------------------------|
| DATE | 2022.05.26              |
| PART | 7. While you are coding |


## 1. 기억하고 싶은 내용 
### 1-1. 파충류의 뇌에 귀 기울이기
* 새로운 프로젝트를 시작하는 일도 두렵기는 마찬가지다. 심지어 이미 있는 프로젝트에 새로운 모듈을 추가하는 일도 그렇다. 많은 사람이 일을 시작하는 첫 발짝을 미루고 싶어 한다. 

* 첫 번째 원인은 파충류의 뇌가 여러분에게 무언가 할 말이 있어서다. 여러분은 개발자로서 여러 가지를 시도해 보면서 잘 되는 것과 안 되는 것들을 보아 왔다. 어떤 작업을 앞두고 마음 속에 의심이 계속 남아 있거나 왠지 꺼림칙하다면, 여러분의 경험이 여러분에게 말을 거는 중일지도 모른다. 그 느낌을 따라라. 정확하게 짚지는 못하더라도, 시간을 좀 주면 아마 더 실체가 있고 대응책을 생각할 수 있는 무엇으로 구체화 될 것이다.

* 두 번째는 합리적인 두려움이다. 우리 개발자들은 코드에 많은 것을 투자한다. 그래서 코드의 오류를 자신의 부족한 능력 때문이라고 받아들일 수도 있다. 아마 임포스터 증후군의 요소 또한 있을 것이다. 자신의 능력 밖이라 생각할 수도 있다. 우리는 이 길의 끝에 무엇이 기다리고 있는지 모르고, 어쩌면 너무 멀리까지 가버린 후에 사실은 길을 잃었다는 것을 인정하게 될 지도 모른다. 

* 일단 하고 있는 일을 멈추고, 여러분의 뇌가 정리를 좀 할 수 있도록 약간의 시간과 공간을 확보하라. 코드에 대해 생각하지 말고 키보드에서 떨어져서 잠깐 머리를 비운 채로 할 수 있는 일을 하라. 

* 문제를 표면으로 끄집어내 보라. 작성하는 코드에 대한 그림을 그려 보라. 동료에게 설명해 보라. 고무 오리도 괜찮다. 여러분 뇌의 다른 부위에 문제를 노출하라. 

* 이런 방법들을 시도해 보았는데도 여전히 막혀 있을 수도 있다. 행동해야 할 시간이다. 여러분의 뇌에게 여러분이 하려는 일은 별 문제가 없다고 알려줘야 한다. 바로 프로토타이핑을 하면 된다. 


### 1-2-. 우연에 맡기는 프로그래밍
* 단순히 코드가 지금 작성된 방식이 그렇기 때문에 생기는 우연한 일들이 있다. 이런 우연에 기대다 보면 결국 문서화되지 않은 에러나 예외적인 경우의 동작에 의존하게 되고 만다. 예를 들어 어떤 루틴을 잘못된 데이터를 가지고 호출했다고 해 보자. 루틴은 예상하지 못한 데이터에 특정한 방식으로 반응을 하고, 여러분은 그 반응을 기반으로 코드를 작성한다. 하지만 루틴을 만든 사람의 의도는 그 루틴이 그런 식으로 작동하는 것이 아니었다. 만든 사람이 생각조차 못했던 경우인 것이다. 이 루틴을 '고치면' 여러분의 코드는 작동을 멈출지도 모른다. 가장 극단적인 경우에는 여러분이 호출한 루틴이 실제로는 그렇게 설계된 루틴이 아닌데도 원하는 효과를 내는 것처럼 보일 수도 있다. 잘 작동하는 데 괜히 건드려서 일을 만들 필요가 있을까? 우리가 보기에는 그래야 할 이유가 몇 가지 있다. 
  * 정말 제대로 돌아가는 게 아닐지도 모른다. 
  * 여러분이 의존하는 조건이 단지 우연인 경우도 있다. 
  * 불필요한 추가 호출은 코드를 더 느리게 만든다. 
  * 추가로 호출한 루틴에 새로운 버그가 생길 수도 있다. 

* 다른 사람이 호출할 코드를 작성하고 있다면 모듈화를 잘하는 것, 그리고 잘 문서화한 적은 수의 인터페이스 아래에 구현을 숨기는 것 같은 기본 원칙들이 모두 도움이 된다. 

* 우리가 일했던 한 대형 프로젝트에서는 외부 현장에 설치된 다수의 데이터 수집 하드웨어로부터 인입되는 데이터를 가지고 보고서를 생성해야 했다. 이 수집 기기들은 미국의 여러 주와 시간대에 걸쳐 설치되어 있었는데, 다양한 프로젝트 진행 과정상의 이유와 과거로부터 이어져 온 이유로 인해 기기는 제각기 현지 시각로 설정되어 있었다. 특정한 경우 '딱' 한 시간 틀리는 것은 우연이었다. 시간을 다루는 적절한 모델이 없었기 때문에 점점 말도 안 되는 양의 +1, -1 명령들이 코드 전체에 퍼져 나갔다. 결국 제대로 맞는 값이 없었고, 프로젝트는 폐기되었다. 

* 언제나 사용자가 글을 읽을 수 있다고 생각하는가? 확실한 것이 아닌데도 의존하고 있는 것은 또 무엇이 있을까? 

* 더 경험이 적은 프로그래머에게 코드를 상세히 설명할 수 있는가? 그렇지 않다면 아마 우연에 기대고 있는 것일 터이다. 

* 자신도 잘 모르는 코드를 만들지 말라. 이것이 왜 동작하는지 잘 모른다면 왜 실패하는지도 알 리가 없다. 


### 1-3. 알고리즘의 속도 
* 매우 간단한 몇몇 알고리즘을 제외한 대부분의 알고리즘은 가변적인 입력 데이터를 다룬다. n개의 문자열 정려하기, m x n행렬의 역행렬 만들기, n비트키를 이용해서 메시지 암호화하기 등이 그렇다. 일반적으로 입력의 크기는 알고리즘에 영향을 준다. 입력의 크기가 클수록 알고리즘의 수행 시간이 길어지거나 사용하는 메모리 양이 늘어난다. 이런 관계가 늘 1차 함수처럼 선형적`linear`이라면, 다시 말해 늘어나는 시간이 n에 정비례한다면 이 항목은 그다지 중요하지 않을 것이다. 하지만 <u>중요한 알고리즘은 대부분 선형적이지 않다</u>. 좋은 소식은많은 알고리즘의 증가폭이 선형보다 작다는 것이다. 예를 들어 이진 검색은 일치하는 항목을 찾을 때 모든 후보를 다 살펴볼 필요가 없다. 나쁜 소식은 나머지 알고리즘들은 증가 폭이 선형보다 훨씬 크다는 것이다. 즉, 수행 시간이나 메모리 요구량이 n보다 훨씬 빠르게 늘어난다. 

* 우리는 반복문이나 재귀 호출을 담고 있는 코드를 작성할 때면 언제나 무의식적으로 수행 시간과 필요한 메모리 양을 계산한다. 정신 계산은 아니고, 주어진 호나경에서 우리가 하는 일이 말이 되는지 가볍게 확인해 보는 정도에 가깝다. 하지만 생각보다 훨씬 상세한 분석을 해야 하는 경우도 종종 실제로 있다. 이럴 때 대문자 O 표기법`Big-O notation`이 유용하다.

#### 대문자 O 표기법`Big-O notation`
* 대문자 O 표기법은 근사값을 다루는 수학적 방법으로 O()와 같이 표기한다. 어떤 정렬 루틴이 원소 n개를 정렬하는데 O(n2) 시간이 걸린다고 말할 때, 이는 그저 최악의 경우에 걸리는 시간이 n의 제곱에 비례하여 늘어난다고 얘기하는 것이다. 원소 수가 두 배로 늘어나면 걸리는 시간은 대략 네 배가 된다. O가 '...차수로`order of`'를 뜻한다고 생각하면 된다. 

* 예를 들어 어떤 함수가 O(n2)시간이 걸린다고 하면, 이 말은 이 함수가 실행되는 데 걸리는 시간의 최댓값이 n2보다 더 빨리 늘어나지 않는다는 뜻이다. 

* 상당히 복잡한 O() 함수가 만들어지는 경우도 있는데, n이 커질수록 가장 큰 차수에 비하면 다른 차수는 무시해도 될 정도이기 때문에, 관습적으로 최상위 차수를 제외한 다른 모든 차수는 제거하며, 상수인 계수도 표기하지 않는다. 이것이 O() 표기법의 특징이다. 어떤 O(n2) 알고리즘이 다른 O(n2) 알고리즘보다 천 배나 빠를 수도 있지만 대문자 O표기법만으로는 그 사실을 알 수 없다. 대문자 O 표기법은 수행 시간이든 메모리든, 아니면 다른 무엇을 나타내든 실제 숫자를 알려주지 않는다. 그저 <u>입력의 크기가 바뀜에 따라 이 값이 어떻게 바뀔지</u>를 알려줄 뿐이다.

* 가장 빠른 알고리즘이 언제나 가장 좋은 알고리즘은 아니다. 입력값의 규모가 작다면 단순한 삽입 정렬도 퀵 정렬과 비슷한 성능을 낸다. 그러나 삽입 정렬을 작성하고 디버깅하는 데 걸리는 시간은 퀵 정렬보다 적다. 여러분이 선택한 알고리즘이 요구하는 형식으로 입력 데이터를 준비하는 데 비용이 많이 드는 것은 아닌지 주의 깊게 보아야 한다. 그리고 성급한 최적화 `premature optimization`를 조심하라. 언제나 어떤 알고리즘을 개선하느라 여러분의 귀중한 시간을 투자하기 전에 그 알고리즘이 정말로 병목인지 먼저 확인하는 것이 좋다. 

* 알고리즘을 어떻게 설계하고 분석하는지에 대한 감각이 있어야 한다. 
  * 로버트 세지윅`Robert Sedgewick` - 알고리즘, An Introduction to the Analysis of Algorithms(알고리즘 분석 입문)
  * 도널드 카누스`Donald Knuth` - The Art Of Computer Programming(1~4A)


### 1-4. 리팩터링 
* 소프트웨어 개발은 딱딱하기보다는 유기적인 활동이다. 최초 계획과 조건에 맞추어 코드를 작성하지만, 계획한 대로 잘 되지 않는 것들은 잡초 제거하듯 뽑아내거나 가지치기를 해야 한다. 코드 고쳐쓰기, 다시 작업하기, 다시 아키텍처 만들기는 모두 아울러서 '재구성`restructuring`'이라고 부른다. 그런데 그런 활동 중 일부를 따로 떼어 '리팩터링`refactoring`'이라는 이름으로 실천하기도 한다. 

* <b>마틴 파울러</b>는 <b><리팩터링></b> 책에서 리팩터링을 밖으로 드러나는 동작은 <b>그대로 유지한 채</b> 내부 구조를 변경함으로써 이미 존재하는 코드를 재구성하는 체계적 기법이라고 정의했다. 이 정의에서 핵심적인 부분은 다음 두 가지다. 
  * 이 활동은 체계적이다. 아무렇게나 하는 것이 아니다. 
  * 밖으로 드러나는 동작은 바뀌지 않는다. 기능을 추가하는 작업이 아니다. 

* 리팩터링을 식물을 다시 심기 위해 정원 전체를 갈아엎듯이 해서는 안 된다. 특별하고 격식을 갖추어야 하는 활동, 가뭄에 콩 나듯 드물게 하는 활동이어서는 안 된다는 것이다. 리팩터링은 그런 게 아니라 잡초 제거나 갈퀴질처럼 위험하지 않은 작은 단계들을 밟은 일상 활동이다. 무질서하게 대규모로 코드를 다시 쓰는 것이 아니라, 정확한 목적을 가지고 정밀하게 접근하는 활동이다. 그래서 코드를 바꾸기 쉽게 유지하는 것이다. 밖으로 드러나는 동작이 바뀌지 않는다는 것을 보장하려면 코드의 동작을 검증하는 좋은 자동화된 단위 테스트가 필요하다. 

* 리팩터링은 여러분이 무언가를 알게 되었을 때 한다. 여러분이 작년이나 어제, 심지어 10분 전과 비교해서 더 많이 알게 되었다면, 리팩터링을 한다. 어쩌면 코드가 더는 잘 맞지 않아서 장애물에 부딪혔을 때, 두 가지가 사실은 하나로 합쳐져 있어야 한다는 것을 발견했을 때, 무엇이든 '잘못'되었다는 생각이 들 때가 있을 것이다. 주저하지 말고 변경하라. 언제나 바로 지금이 최적기다. 코드를 리팩터링할 이유는 아주 많다. 
  * DRY 원칙 위반 발견 
  * 직교적으로 바꿀 수 있는 무언가 발견 (결합도가 높은)
  * 더 이상 유효하지 않은 지식
  * 사용 사례
  * 성능 
  * 테스트 통과 

* 소스 코드를 이곳저곳 변경하는 것은 굉장히 고통스러운 작업일 수도 있다. 작동하는 코드이니 괜히 긁어 부스럼 만들지 않는 편이 나을 수도 있다. 많은 개발자들이 코드에 조금 개선할 부분이 있다는 이유만으로는 다시 돌아가서 코드 열기를 주저한다. 

* 일정의 압박은 리팩터링을 하지 않는 단골 핑계다. 하지만 이는 설득력이 떨어진다. 지금 리팩터링을 하지 않으면 일이 더 진척되었을 때, 즉 신경 써야 할 의존성이 더 많아졌을 때 문제를 고쳐야 하고, 따라서 훨씬 더 많은 시간을 투자해야 한다. 그때가 되면 일정에 더 여유가 생길까? 그럴 리가.

* 물론 코드 한 부분 때문에 리팩터링만 하는 일주일이 필요해서는 안 된다. 그건 완전 재작성이다. 만약 이 정도로 시간이 많이 필요하다면 즉시 해치울 수 없는 것도 당쳔하다. 그 대신 일정에 리팩터링할 시간을 확실히 포함시켜 두도록 하라. 그 코드를 사용하는 사람들이 코드가 조만간 재작성될 것이라는 사실과 재작성이 그들의 코드에 미칠 영향을 인지하도록 해야 한다. 

* 마틴 파울러는 오히려 손해 보는 일이 없도록 리팩터링 하는 방법에 대하여 몇 가지 간단한 조언을 해주었다. 핵심은 탄탄한 회귀 테스트를 유지하는 것이야말로 안전한 리팩터링의 비결이라는 것이다. 
  * 리팩터링과 기능 추가를 동시에 하지 말라. 
  * 리팩터링을 시작하기 전 든든한 테스트가 있는지 먼저 확인하라. 
  * 단계를 작게 나누어서 신중하게 작업하라. 

* 요즘은 많은 IDE에서 자동 리팩터링을 지원해준다. 


### 1-5. 테스트로 코딩하기 
* `여러분의 소프트웨어를 테스트하라. 그렇지 않으면 사용자가 테스트하게 된다.` 테스트는 프로그래밍의 일부다. 테스트, 설계, 코딩, 이 모든 것이 프로그래밍이다. 

* 테스트의 중요한 가치는 무엇일까? 테스트가 잘 작동하는지 확인하려는 것이 아니다. 버그를 찾기 위한 것이 아니다. 우리는 테스트의 주요한 이득이 테스트를 실행할 때가 아니라 테스트에 대해 생각하고 테스트를 작성할 때 생긴다고 믿는다. 

* 테스트에 대해 생각함으로써 우리 코드의 결합도는 낮추고 유연성은 올라간다. 코드의 작성자가 아니라 사용자인 것처럼 메서드를 외부의 시선으로 보게 되었다. `테스트가 코드의 첫 번째 사용자다.` 

* 다른 코드와 긴밀하게 결합된 함수나 메서드는 테스트하기 힘들다. 테스트를 실행하기도 전에 온갖 환경 구성을 한참 해야 하기 때문이다. 

* 게다가 무언가를 테스트하려면 그것을 이해해야만 한다. 우스꽝스럽게 들리겠지만 현실에서는 우리 모두 우리가 해야 하는 일이 무엇인지 아리송한 채로 코드를 작성해서 출시한 적이 있을 것이다. 우리는 앞으로 진행하면서 차근차근 해결할 것이라고 약속한다. 

* 테스트를 먼저 생각하는 것의 이점이 많다 보니 아예 테스트를 먼저 작성하자고 주장하는 프로그래밍 유파도 있다. `Test-driven development, TDD`라고 부르는 기법을 사용한다. `Test-first development`라고 부르기도 한다. TDD의 기본 주기는 다음과 같다. 
  * 추가하고 싶은 작은 기능 하나를 결정한다. 
  * 그 기능이 구현되었을 때 통과하게 될 테스트를 하나 작성한다. 
  * 테스트를 실행한다. 다른 테스트는 통과하고 방금 추가한 테스트 딱 하나만 실패해야 한다. 
  * 실패하는 테스트가 통과시킬 수 있는 최소한의 코드만 작성한다. 그리고 이제는 모든 테스트가 통과하는지 확인한다. 
  * 코드를 리팩터링한다. 개선 후에도 테스트가 통과하는지 확인한다. 

* TDD 발상의 핵심은 이 반복 주기가 기껏해야 몇 분 정도로 매우 짧아야 한다는 것이다. 그러나 TDD의 노예가 된 사람들도 보았다. 테스트 커버리지 100%를 달성하기 위해 과도하게 많은 시간을 투자하거나, 많은 수의 중복 테스트가 생기거나 말이다. 그리고 밑에서부터 시작하여 위로 올라가는 방식으로 설계를 한다. 어떻게든 TDD를 실천하라. 하지만 도중에 이따금 멈추어 큰 그림을 살피는 것을 잊지 말라. 초록색 "테스트 통과" 메시지에 중독된 나머지 진짜 문제 해결에는 보탬이 안 되는 코드를 한 무더기나 쓰게 되기 쉽다. 

* 소프트웨어 단위 테스트`Unit test`란 어떤 모듈에게 이것저것을 시켜보는 코드를 가리킨다. 일반적으로 단위 테스트는 일종의 인위적인 환경을 구축한 다음, 테스트할 모듈의 루틴들을 호출한다. 그런 다음 반환된 결과들을 이미 알고 있는 값과 비교해 보거나 똑같은 테스트를 이전에 돌렸을 때 나온 값과 비교하여 올바른지 검사한다. 동일한 테스트를 코드 수정 후 다시 돌려보는 것을 회귀 테스트`regression testing`라고 한다. 

* 우리는 단위 테스트를 계약을 잘 지키는지 보는 테스트라고 여긴다. 이런 테스트는 우리에게 두 가지를 알려준다. 하나는 코드가 계약을 지키는지 여부고, 다른 하나는 코드로 표현된 계약의 의미가 우리가 생각한 것과 일치하는지 여부다.

* 현실에서는 아주 사소한 모듈이 아닌 이상 여러 개의 다른 모듈에 의존하고 있을 가능성이 높다. 그렇다면 어떻게 이런 모듈들을 조합하여 테스트할 수 있을까? `A`라는 모듈이 `DataFeed`와 `LinearRegression`이라는 다른 모듈을 사용한다고 해보자. 우리는 다음과 같은 순서로 테스트를 진행할 것이다. `테스트를 할 수 있도록 설계하라.`
  * `DataFeed`의 계약을 완전히 테스트한다. 
  * `LinearRegression`의 계약을 완전히 테스트한다. 
  * `A`의 계약을 테스트한다. `A`의 계약은 나머지 모듈의 계약에 의존하지만 그 사실이 직접적으로 드러나 있지는 않다. 

* 아무리 테스트를 잘 갖추었어도 모든 버그를 발견할 수도 없다. <b>어떤 모듈의 내부 상태를 디버거 없이 다양한 형태로 볼 수 있는 방법을 제공</b>할 수도 있다. (디버거는 이미 출시된 애플리케이션에서는 사용이 불편하거나 아예 불가능할 수도 있다.) <b>로그 파일에 쌓이는 추적`trace` 메시지가 이런 메커니즘 가운데 하나다.</b> 그 메시지는 반드시 규칙적이고 일관된 형식이어야 한다. 


### 1-6. 속성 기반 테스트 
* 우리는 코드가 지켜야 하는 계약을 코드에 포함하자고 이야기했다. 선행 조건에 맞추어 입력을 넣으면, 코드가 생상하는 출력이 주어진 후행 조건에 맞음을 보장해 준다. 불변식`invariant`이라는 것도 있었는데, 함수 실행 전후로 계속 어떤 부분의 상태에 대하여 참이 되는 조건을 말한다. 이렇게 코드에 존재하는 계약과 불변식을 뭉뚱그려서 '속성`property`'이라고 부른다. 코드에서 속성을 찾아내서 테스트 자동화에 사용할 수 있는데, 이것을 '속성 기반 테스트`property-based testing`'라 한다. 
  ```Python
  from hypothesis import given
  import hypothesis.strategies as some

  @given(some.lists(some.integers()))
  def test_list_size_is_invariant_across_sorting(a_list):
    original_length = len(a_list)
    a_list.sort()
    assert len(a_list) == original_length

  @given(some.lists(some.test()))
  def test_sorted_result_is_ordered(a_list):
    a_list.sort()
    for i in range(len(a_list) - 1):
      assert a_list[i] <= a_list(i + 1)
  ```

* 속성 기반 테스트의 여러 가지 다른 수행 결과와 상관없이 문제가 발생하는 상황에 집중할 수 있게 해준다.

* 단위 테스트가 '회귀 테스트`regression test`' 역할을 한다. 속성 기반 테스트는 임의의 값을 생성하여 사용하기 때문에 다음번에 실행했을 때 똑같은 값을 테스트 함수에 넘긴다는 보장이 없다. 문제를 일으켰던 값을 사용하는 단위 테스트를 만들어 두면 버그가 완전히 해결되었음을 보장할 수 있다. 


### 1-7. 바깥에서는 안전에 주의하라
* 우리가 언제나 명심해야 하는 기본 원칙이 있다. 
  * 공격 표면을 최소화하라
  * 최소 권한 원칙`less is more`
  * 안전한 기본값
  * 민감 정보를 암호화하라 
  * 보안 업데이트를 적용하라 

* 지금까지 발생한 데이터 유출 사고 중 가장 큰 사고는 업데이트를 하지 않은 시스템 때문에 발생했다는 사실만 기억하기를 바란다. 

* 여러분이 만든 기발한 수제 암호화 알고리즘은 전문가가 몇 분 이내로 풀어버릴 것이다. 암호화는 직접 하지 않는 편이 낫다. 신뢰할 수 있는 것에만 의지하라. 많이 검토하고, 철저하게 검사하고, 잘 유지보수되며, 자주 업데이트되는 라이브러리와 프레임워크를 사용하라. 가급적 오픈 소스가 좋다. 

* 간단한 암호화 작업 되에도 여러분의 웹 사이트나 애플리케이션의 보안 관련 기능을 주의 깊게 검토하라. 예를 들어 사용자 인증`authentication`을 보자. 비밀번호나 생체 정보를 이용한 로그인 인증을 구현하려면 해시와 솔트`salt`가 어떻게 동작하는지, 해커가 레인보 테이블`rainbow table` 같은 도구를 어떻게 사용하는지, 왜 MD5나 SHA1을 더는 쓰면 안 되는지 등 다양한 문제를 이해해야 한다. 제대로 구현했다 하더라도 계속해서 데이터를 관리하고 안전하게 유지할 책임은 여전히 여러분에게 있다. 새로운 법령이나 법적 의무에 대한 대응도 여러분 몫이다. 아니면 실용주의적 접근 방법을 선택하여 다른 사람이 대신 고민하도록 할 수도 있다. 외부의 인증 서비스를 사용하는 것이다. 어떤 서비스를 사용하든 아마 여러분보다 더 나을 것이다. 


### 1-8. 이름 짓기 
* 이름이란 게 무슨 의미가 있을까? 프로그래밍에서는 이름이 "모든 것"이다. `올바른 이름으로 부르는 것이 지혜의 시작이다.`

* 여러분의 뇌는 단어를 꼭 지켜야 하는 것으로 인식한다. 그렇다면 이에 걸맞게 우리가 사용하는 이름을 잘 지어야 한다. 예제를 보자. 
  ```javascript
  let user = authenticate(credentials) 
  ```
  `user`에는 아무 의미가 없다. `customer`나 `buyer`는 어떨까? 이런 이름을 사용하면 코딩할 때마다 이 사람이 뭘 하려고 하는지, 우리에게 어떤 의미가 있는지 계속 되새기게 될 것이다. 

  주문에 할인을 적용하는 인스턴스 메서드가 있다. 
  ```java
  // deductPercent: 함수가 하는 일. 왜 이 일을 하는지 나타나있지 않다. 
  // @param amount: 헷갈린다는 비난을 면하기 힘들다. 할인 금액인가? 할인율인가? 
  public void deductPercent(double amount) { 
    // ... 
  }
  ```
  
  메서드 이름에 의도가 명확하게 드러나게 하고, 매개 변수의 타입도 변경했다. 여러분은 어떤지 모르겠지만, 우리는 백분율을 다룰 때 값이 0에서 100 사이인지, 아니면 0.0에서 1.0 사이인지 늘 헷갈린다. 전용 타입을 사용하면 함수에 어떤 값을 넘겨야 하는지 명시할 수 있다.
  ```java
  public void applyDiscount(Percentage discount) {
    // ...
  }
  ```

* 이름을 지을 때는 여러분이 표현하고 싶은 것을 더 명확하게 다듬기 위해 끊임없이 노력해야 한다. 이렇게 명확하게 다듬는 작업이 여러분이 코드를 작성할 때 코드를 더 잘 이해할 수 있도록 도울 것이다.

* 반드시 팀의 모든 사람이 각 단어의 뜻을 알고 일관성 있게 사용해야 한다. 팀에게 특별한 의미가 있는 단어를 모두 모은 프로젝트 용어 사전을 만드는 방법도 있다. 시간이 흐르다 보면 프로젝트 용여들은 자리를 잡아 나갈 것이다. 모든 사람이 어휘에 익숙해지면 짧은 단어를 말하는 것만으로도 많은 의미를 정확하고 짧게 전달할 수 있게 된다. (사실 이것이 디자인 패턴 같은 '패턴 언어'가 하는 일이다.)

* `이름을 잘 지어라. 필요하면 이름을 바꿔라.` 잘못된 이름을 바꿀 수 없는 상황이라면 더 큰 문제가 있는 것이다. 바로 ETC 위반이다. 그 문제를 고치고 나서 잘못된 이름을 바꿔라. 이름을 바꾸기 쉽게 만들고, 자주 이름을 바꿔라. 그렇지 않으면 팀에 새로운 사람이 올 때마다 시치미를 뚝 떼고 `getData`가 사실은 데이터를 파일에 쓴다는 것을 설명해야 할 것이다. 

## 2. 소감
이번 챕터의 요약 내용이 좀... 길다. 두 파일로 나눌까도 생각했었는데, 챌린지 결과를 제출해야 하기 때문에 우선은 하나의 파일로 작성한다. 책 내용을 많이 요약한다고 해서 좋은 것은 아니지만, 원래 분량 자체도 길었고 다 중요해보여서 꼽지 못했던 이유도 있다.. 

리팩터링 파트와 테스트 파트는 뼈를 때리는 것 같다. 중요성은 인지하고 있었고, 리팩터링 과정은 보는 것도 하는 것도 재밌기도(어렵기도) 하단 생각이 들지만, 때로 현실적인 벽에 부딪힐 때가 있다(같은 말로 핑계댈 때도 있다.). 잘 작동하고 있는 코드는 동작하는 귀한 코드이기 때문이다. 작업자가 정해놓은, 혹은 내가 만들었던 method spec에서 해당 input으로 값이 들어오지 않는 경우들이 있을 것 같다는 생각이 종종 들 때가 있지만, 그렇다면 일단 그런 경우는 지금까지는 없었던 코드인 것이다. 해당 method에 어떤 기능을 덧붙이려고 할 때 input을 맞춰주기 위해 어떤 변환 과정이 필요할 수도 있다. 그것도 우연에 맡기는 프로그래밍일까? 우연찮게 발견한 버그인지 모를... 그런 애매한 경우일 때 말이다. 지금 생각해보면 전에 책에서 봤던 DBC로 개발했다면, 또 테스트 코드가 있었다면 훨씬 도움이 되었을 것 같단 생각이 든다. (갑자기 내가 전에 작성했던 코드에 대해 테스트 코드를 꼭 작성해두어야겠단 생각이 든다.) 
또 그와는 별개로 어떤 기능 추가가 필요한 일이 있었는데, 테스트 코드가 있어서 그 코드로 마음껏 여러 가지를 시도해봤던 경험도 있었다. 결국 남는 건 몇 줄의 코드가 되긴 했지만 테스트 코드 작성 연습을 많이 해야겠단 생각이 다시 한 번 들었다. 바쁘다는 이유로, 일정이 빠듯하다는 이유로, 많은 것을 하나씩 빼게 된다. 

그리고 전에 테스트 관련해 TDD와 DBC의 차이에 대해서도 궁금해 했었는데, TDD로 계약조건에 대해 검증하면 되겠단 생각도 들었다. 이 부분은 추후에 조금 더 찾아봐야 할 것 같다. `given, when, then Pattern`이 그와 관련된 것이 아닐까? 

책에서 개인적으로 멋지단 생각이 들었던 코멘트가 하나 있다. 내용 요약에는 적지 않았지만, 개발자로서의 30년을 그려보게 하는 말이다. 작가는 더 이상 테스트코드를 작성하지 않는다고 한다. 그러나 거기엔 이유가 있고, 특정 경우에는 테스트를 작성한다고 한다. 그리고 작가는 이렇게 코멘트했다. `여러분이 테스트를 써야 할까? 그렇다. 하지만 테스트 작성 경험이 30년 정도 쌓였다면 테스트가 어떤 면에서 도움이 되는지 직접 실험을 해 봐도 좋을 것이다.`

## 3. 궁금한 내용 
* [[테스트] PBT(Property Based Testing)](https://jerry92k.tistory.com/m/31)
* [Big-O (빅 오) 표기법](https://ko.khanacademy.org/computing/computer-science/algorithms/asymptotic-notation/a/big-o-notation)
* [[자료구조] 빅오 표기법(Big-O notation)이란?](https://holika.tistory.com/29)