# Living Documentation

### What is living documentation
원문 : http://searchsoftwarequality.techtarget.com/definition/living-documentation

'living documentation' 은 정보들을 정확하고 이해하기 쉽게 제공하는, 시스템 documentation 의 동적인 방법이다. 자연어로 쓰여진 feature 파일은 코어로써 'living documentation’을 돕는다. 
각각의 파일은 한 단위의 코드가 어떻게 수행하도록 가정하고, 예제를 줘서 원하는 결과를 설명한다. 그 documentation은 논리적 관점에서 시스템의 원하는 동작을 설명한다. 그래서 비지니스 이해관계자들은 쉽게
확인하고 리뷰할 수 있다. 개발자들은 그들의 프로그램에 뭐가 필요한지 돕는 정보로, 가능한 최소한의 코드를 추가하면서 사용할 수 있다. 테스터들은 documentation을 통해 새로운 테스트를 만들고 테스트 결과가 요구사항들을 만족시키는지 확인할 수 있다. Operation은 진행중인 유지보수 작업을 보조하는 정보로 사용된다. 

'living documentation’ 접근은 보통 behavior-driven-development(BDD)와 specification by example(SBE)와 함께한다. 이러한 방법론들과 함께, '실행 가능한 명세' 예를 들어 ‘automated acceptance tests’ 는 종종 문서의 역할을 하도록 하는 방식으로 작성된다. ‘자동화 테스트’ 는 정기적으로 실행되기 때문에 SW의 변화는 'living documentation’ 의 변화를 필요로 한다. 따라서 이러한 방식의 테스팅과정이, 관련된 문서가 항상 최신이라는 것을 보장한다. 

cf) Specification by Example에서 ‘living documentation’
시스템이 변경될 때 문서에도 그러한 변경사항이 즉시 반영될 수 있는 문서 즉, 실행 가능한 문서를 지칭한다. 오랫동안 업데이트되지 않아서 쓸모 없게 된 문서가 아니라, 항상 최신 상태가 유지되어 신뢰할 수 있을 뿐더러 언제나 사용하고 실행해볼 수 있는 문서를 의미한다. 

### Why You should write living documentation
원문 : http://blog.mojotech.com/why-you-should-write-living-documentation/

이글을 읽는 당신이 SW 개발자라면 빨리 ‘코드 원숭이’ 라는걸 깨닫길 바란다. 만약 아니라면 당신은 자연어로 쓰여진 프로세스의 상세한 설명과, 컴퓨터가 이해 할 수 있는 프로그래밍 언어 간의 변환이 될것이다.
다음과 같은 상황을 고려해봐라. 종종 우리는 구현해야 되는 feature에 대한 막연한 명세만 갖고 시작할때가 있다. 그럴 때 feature가 정확히 무엇에 관해서인지, 왜 필요한지 그리고 어떻게 사용되는지를 이해하기 위해 몇몇 사람들과 얘기를 나눠야된다.(먼저 누구랑 얘기를 나눌지부터 결정하고) 이런 일이 발생할 때 우리는 무엇이 진짜 문제인지를 이해하기 위해 해결책으로부터 거슬러 올라가서 볼 필요가 있다. 이때 원래 의도된 해결책을 평가할 수 있는 능력을 얻게 된다. 

당신이 요구사항을 갖고 있고 그 feature를 만들어야된다면, 원래의 solution-problem을 포함해서 가능한한 각각의 solution들을 고려해야한다. 다음과 같은 질문을 하라.
- input이 무엇인가? 어떤 일이 일어나야 되는가? corner case의 경우가 있는가?
- 새로운 feature는 어떻게 기존 애플리케이션과 통합되는가?
- 기존 기능에 어떻게 영향을 끼치는가? 얼마나 복잡한가? 
- 얼마나 사이즈가 큰가?
- 성능에 문제가 있는가?
- 마지막으로 이러한 질문들이 문제에 중요한가?

많이 들어본 얘기인가? 여기 또 다른 들어본 얘기가 있다. 

“None of this involves writing code.”

지금 이 과정은 문제 해결의 필수 요소인 생각과 의사소통에 대한 것이지 코드에 관한 것이 아니다. **우리는 단순 SW 개발자가 아니라 problem solver 이다. ** 사실 우리는 코드를 전혀 쓰지 않기로 하면서 우리의 문제를 해결한다(저자는 그게 최고의 해결책이라고 주장한다). 

**TO DOCUMENT, OR NOT TO DOCUMENT**<br>
당신이 빛을 아직 보지 못한 사람이라고 해보자. 그런데 당신은 빛을 생각해내야 하는 문제에 있다. 어떻게 할 것인가? 아마 해결책에 도달할때까지 빛을 생각해보고, 스케치를 해보고, 써보는걸 반복할 것이다. 그렇게 해서 나온 output은 어떻게 될까? 그냥 무시하는가? 아니면 당신이 사용하는 프로젝트 계획 도구에 묻혀 두고 구현을 시작하는가? 아마 당신은 코드가 디자인이라고 믿을 것이다.

**코드가 디자인인가? 그렇기도 하고 아니기도 하다.**<br> 
먼저 이론적으로 코드는 디자인 될 수 있고 동작하는 애플리케이션에 필요한 모든 것이며, 애플리케이션이 어떻게 동작하는지 이해하기 위한 것이다. 하지만 코드는 어떻게 동작하는지만 알려줄 뿐이다. 코드는 ‘왜 그렇게 동작해야되는지 실제로 어떻게 동작하도록 되어 있는지’를 말해주지 않는다. 그래서 코드가 디자인이라고 해도 그것만으로는 부족하다. 

두 번째로, 새로운 애플리케이션을 만드는 사람들에 대해서 생각해보자.(혹은 오랜 시간이 지난 애플리케이션 코드를 볼 때) 단순히 코드를 보는 것만으로는 전체적으로 무슨 일을 하는건지 파악하기 쉽지 않다. 그건 마치 기계가 어떻게 동작하는지 알기 위해 부품 하나하나를 보면서 이해하려고 하는 것과 같다. 이론적으론 가능하나 시간이 오래 걸린다. 

그래서 저자는 documentation이 유용하다라는걸 동의하길 희망한다. 그러나 우리는 모두 documentation이 어떻게 될지 알고 있다. 시간이 지날수록 documentation의 가치는 떨어진다. 계속 최신화해주지 않으면 부식된다. 더 큰 문제는 개발자에게 잘못된 정보를 줄 가능성이 있어서 documentation이 없느니만 못한 결과를 초래하게 된다. 

**LIVING DOCUMENTATION**<br>
Living documentation은 테스트들로 이뤄진 documentation이다. 이러한 이유 때문에 Living documentation은 기존의 documentation처럼 시간이 지날수록 구식이 되긴 힘들다. 아래의 예제는 유명한 Living documentation 도구인 큐컴버에 의해 실행된 것이다. 
```
Feature: Photo display

  A photography site is all about photos, and the bigger 
  they are, the better. If we limit the text content, we 
  can use the entire background area to show a single photo.

  Due to time constraints, we're not building a dedicated 
  photo gallery yet, but we must give people a chance 
  to see more photos, using a very simple navigation system 
  that uses "back" and "forward" links, like a manually 
  activated slideshow.

  Scenario: photo shown on the homepage
    When a visitor visits the homepage multiple times
    Then they should see a different photo each time

  Scenario: photo navigation
    When a visitor is looking at a photo
    Then they can choose to see the photo that follows it
    And they can choose to see the photo that precedes it

  Scenario: photo navigation beyond last photo
    When a visitor tries to see the photo that comes after 
    the last photo
    Then they should be shown the first photo in the 
    slideshow instead

  ...
```
이것들은 living documentation의 일반적인 컴포넌트들이다. 
- feature 이름 
- quick overview : 왜 feature가 있는지, 해결한 문제가 뭔지, 어떤 대안들을 찾아봤는지 등등 
- 몇몇 사람들은 잘 알려진 스토리 포맷을 사용한다. “As a ... I want to ... so tht ...” 또 어떤 사람들은 자유로운 포맷을 선호한다. 
- 시나리오들은 약간의 정해진 포맷으로 작성된다.(각각의 라인은 반드시 Given, When, Then, And, But 중 하나로 시작해야 된다.)

위 documentation에 그 어떤 코드도 없다는걸 알아챘는가? 완벽하게 자연어로 작성된, 읽을수 있는 문서이기 때문에 이해 관계자(stakeholders)들 또한 볼수 있다. 하지만 무엇보다도 팀 동료들이 읽을 수 있다.
잘 정제된 이러한 documentation은 애플리케이션이 무엇을, 그리고 왜 하는지에 대한 이해를 기본으로 한다. 팀 공통 용어집을 정의하는데, 그리고 이전에 까먹었던 기능을 호출하는데 사용해라. 또 PDF로 출력해서 새로운 팀 동료가 읽게 해라.

**WHY NOT USE A LIGHWEIGHT SPEC FRAMEWORK TO WRITE THE DOCUMENTATION?**<br>
spec framework(예를 들어, RSpec, Capybara’s feature DSL)을 사용할 수 있지만 몇 가지 단점이 있다. 아래의 문서를 보자.

Gherkin 버전
```
Feature: Background picture display

  Scenario: show random background picture
    When a visitor visits the page multiple times
    Then they see a different background picture each time
```
RSpec 버전
```
describe "Background picture display" do
  context "When a visitor visits the page multiple times"
    it "They see a different background picture each time" do
    end
  end
end
```
첫 번째와 두 번째의 몇 가지 중요한 차이점들이 있다. 먼저 Gherkin 버전은 RSpec 보다 덜 지저분하다. ‘Feature:’와 ’Scenario:’ 키워드가 있고 들여쓰기가 있다. 반대로 RSpec 버전은 닫는 인용구가 필요하고 3개의 다른 키워드(describe, context, it)를 사용한다. 

둘째, 지금은 코드에 대해서 생각하는 시간이 아니다! 오롯이 비지니스 규칙에만 집중해야한다. 당신은 다시 쓰면서 생각을 명확히 하길 원하기 때문에, 몇 단계를 거쳐서 도메인에 더 잘 맞는 용어로 바꿀것이다. RSpec 예제의 딱딱한 구조는 더 어렵게 만들고 있다. 텍스트는 코드보다 조작하기도 쉽다.

셋째, 읽기 힘들다. 코드와 비지니스 규칙 설명이 혼재되어 있다. 당신이 비지니스 규칙 부분만 추출하고 싶어도 할 수 없다.

**결론**<br>
이 글에서 저자는 코드에서 한걸음 물러나서 더 상위 레벨 이슈에 대해 생각해보는것을 원했다. 이후의 글에서는 완벽한 예제를 가지고 living documentation에 대해서 더 깊이있게 살펴본 후 더 나은 workflow를 제안할 예정이다. 인간이 쓰는 언어보다 더 강력한 것은 없다. 이걸 사용해라! 