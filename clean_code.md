# 클린코드

## 2장 의미 있는 이름
### 검색 하기 쉬운 이름을 사용하라
MAX_CLASSES_PER_STUDENT는 grep으로 찾기가 쉽지만, 숫자 7은 은근히 까다롭다. 7이 들어가는 파일 이름이나 수식이 모두 검색되기 때문이다.
개인 사례) unAccess 라는 변수가 있을 때 관련 api url을 `/un/access` 로 짓는거보다는 `/unaccess` 로 짓는게 검색하기 쉽다. 검색이 쉬우니 네이밍 변경하기도 쉽다.

## 3장 함수
### 한 가지만 해라!
```java
// 목록 3-2 HtmlUtil (리팩토링한 버전)
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
    boolean isTestPage = pageData.hasAttribute("Test");
    if (isTestPage) {
        WikiPage testPage = pageData.getWikiPage();
        StringBuffer newPageContent = new StringBuffer();

        includeSetupPages(testPage, newPageContent, isSuite);
        newPageContent.append(pageData.getContent());
        includeTeardownPages(testPage, newPageContent, isSuite);

        pageData.setContent(newPageContent.toString());
    }
    return pageData.getHtml();
}
```

설정(setup)페이지와 해제(teardown)페이지를 testPage에 넣은 후 해당 testPage를 HTML로 렌더링 하는 코드다.

함수를 만드는 첫째 규칙은 '작게'다. 함수를 만드는 둘째 규칙은 '더 작게'다. 증거자료나 근거를 대기 곤란하지만 저자는 오랜 시행착오를 바탕으로 작은 함수가 좋다고 확신한다.
다시 말해, if 문 / else 문 / while 문 등에 들어가는 블록은 한 줄이어야 한다는 의미다.

```java
// 목록 3-3
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData))
        includeSetupAndTeardownPages(pageData, isSuit);
    return pageData.getHtml();
}
```

그리고 함수는 한 가지만을 해야 한다. 이 충고의 문제점은 그 '한 가지'가 무엇인지 알기가 어렵다는 점이다. 위의 코드는 한 가지만 하는가? 세 가지를 한다고 주장할 수도 있다.

1. 페이지가 테스트 페이지인지 판단한다.
2. 그렇다면 설정 페이지와 해제 페이지를 넣는다.
3. 페이지를 HMTL로 렌더링한다.

하지만 위에서 언급하는 세 단계는 **지정된 함수 이름 아래에서** 추상화 수준이 하나다. 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다.
어쨌거나 우리가 함수를 만드는 이유는 큰 개념을(다시 말해, 함수 이름을) 다음 추상화 수준에서 여러 단계로 나눠 수행하기 위해서가 아니던가.

목록 3-2 코드는 추상화 수준이 둘이다(한가지 일을 하는 함수로 더 분리시킬 수 있다). 그래서 목록 3-3으로 축소가 가능했다.
**함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일 해야 한다.** 예를 들어 `getHtml()`은 추상화 수준이 아주 높다.
반면, `String pagePathName = PathParser.render(pagepath);`는 추상화 수준이 중간이다. 그리고 `.append("\n")`와 같은 코드는 추상화 수준이 아주 낮다.

한 함수 내에 추상화 수준을 섞으면 코드를 읽는 사람이 헷갈린다. 특정 표현이 근본 개념인지 아니면 세부사항인지 구분하기 어려운 탓이다. 더 큰 문제는 깨진 창문(실용주의 프로그래머)처럼
사람들이 함수에 세부사항을 점점 더 추가한다.

### Switch 문
switch 문은 본질적으로 N가지를 처리하기 때문에 작게 만들기 어렵다. 그리고 switch 문을 완전히 피할 방법은 없다. 하지만 각 switch 문을 저차원 클래스에 숨기고 절대로 반복하지 않는 방법은 있다.
물론 다형성을 이용한다.

```java
// 목록 3-4
	public Money calculatePay(Employee e) throws InvalidEmployeeType {
		switch (e.type) {
			case COMMISSIONED :
				return calculateCommissionedPay(e);
			case HOURLY :
				return calculateHourlyPay(e);
			case  SALARIED :
				return calculateSalariedPay(e);
			default: throw new InvalidEmployeeType(e.type);
		}
	}
```

위 코드는 직원 유형에 따라 다른 값을 계산해 반환하는 함수다. 위 함수에는 몇 가지 문제가 있다. 첫째, 함수가 길다. 새 직원 유형을 추가하면 더 길어진다. 둘째, '한 가지' 작업만 수행하지 않는다.
세째, SRP를 위반한다. 코드를 변경할 이유가 여럿이기 때문이다. 네째, OCP를 위반한다. 새 직원 유형을 추가할 때마다 코드를 변경하기 때문이다. 하지만 아마 가장 심각한 문제라면 위 함수와 구조가 동일한 함수가
무한정 존재한다는 사실이다. 예를 들어, 다음과 같은 함수가 가능하다.

```java
isPayday(Employee e, Date date);

deliverPay(Employee e, Money pay);
```

가능성은 무한하다. 그리고 모두가 똑같이 유해한 구조다.

이 문제를 해결한 코드가 목록 3-5다.

```java
// 목록 3-5
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}

-----------------

public interface EmployeeFactory {
	Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}

-----------------

public class EmployeeFactoryImpl implements EmployeeFactory {
	@Override
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED :
				return new CommissionedEmployee(r);
			case HOURLY :
				return new HourlyEmployee(r);
			case  SALARIED :
				return new SalariedEmployee(r);
			default: throw new InvalidEmployeeType(r.type);
		}
	}
}
```

위 코드는 switch 문을 추상 팩토리에 꽁꽁 숨긴다. 아무에게도 보여주지 않는다. 팩토리는 switch 문을 사용해 적절한 Employee 파생 클래스의 인스턴스를 생성한다.
calculatePay, isPayday, deliverPay 등과 같은 함수는 Employee 인터페이스를 거쳐 호출된다. 그러면 다형성으로 인해 실제 파생 클래스의 함수가 실행된다.

### 서술적인 이름을 사용하라!
길고 서술적인 이름이 짧고 어려운 이름보다 좋다.

### 함수 인수
함수에서 이상적인 인수 개수는 0개(무항)다. 다음은 1개(단항)고, 다음은 2개(이항)다. 3개(삼항)는 가능한 피하는 편이 좋다. 4개 이상(다항)은 특별한 이유가 필요하다.
특별한 이유가 있어도 사용하면 안 된다. 인수는 개념을 이해하기 어렵게 만든다.

**많이 쓰는 단항 형식** <br>
함수에 인수 1개를 넘기는 이유로 가장 흔한 경우는 두 가지다. 하나는 인수에 질문을 던지는 경우다. `boolean fileExists("MyFile")`이 좋은 예다. 다른 하나는 인수를 뭔가로 변환해
결과를 반환하는 경우다. `InputStream fileOpen("MyFile")`은 String 형의 파일 이름을 InputStream으로 변환한다. 이들 두 경우는 독자가 당연하게 받아들인다.

다소 드물게 사용하지만 그래도 아주 유용한 단항 함수 형식이 이벤트다. 이벤트 함수는 입력 인수만 있다. 출력 인수는 없다. 프로그램은 함수 호출을 이벤트로 해석해 입력 인수로 시스템 상태를 바꾼다.
`passwordAttemptFailedNtimes(int attempts)`가 좋은 예다. 이벤트 함수는 조심해서 사용한다. 이벤트라는 사실이 코드에 명확히 드러나야 한다. 그러므로 이름과 문맥을 주의해서 선택한다.

지금까지 설명한 경우가 아니라면 단항 함수는 가급적 피한다. 예를 들어, `void includeSetupPageInto(StringBuffer pageText)`는 피한다. 변환 함수에서 출력 인수를 사용하면 혼란을 일으킨다.

입력 인수를 변환하는 함수라면 변환 결과는 반환값을 돌려준다. `StringBuffer transform(StringBuffer in)`이 `void transform(StringBuffer out)`보다 좋다.

`StringBuffer transform(StringBuffer in)`이 입력 인수를 그대로 돌려주는 함수라 할지라도 변환 함수 형식을 따라는 편이 좋다. 적어도 변환 형태는 유지하기 때문이다.

**플래그 인수** <br>
함수로 부울 값을 넘기는 관례는 정말로 끔찍하다. 함수가 한꺼번에 여러 가지를 처리한다고 대놓고 공표하는 셈이다(플래그가 참이면 이걸 하고 거짓이면 저걸하고).

**이항 함수** <br>
인수가 2개인 함수는 1개인 함수보다 이해하기 어렵다. 예를 들어, `writeField(name)`는 `writeField(outputStream, name)`보다 이해하기 쉽다. 둘 다 의미는 명백하지만
전자가 더 쉽게 읽히고 더 빨리 이해된다. 후자는 잠시 주춤하며 첫 인수를 무시해야 한다는 사실을 깨닫는 시간이 필요하다. 그리고 바로 그 사실이 결국은 문제를 일으킨다. 왜냐고? 어떤 코드든
절대로 무시하면 안되니까. 무시한 코드에 오류가 숨어드니까.

물론 이항 함수가 적절한 경우도 있다. `Point p = new Point(0,0)`가 좋은 예다. 직교 좌표계 점은 일반적으로 인수 2개를 취한다. 여기서 인수 2개는 한 값을 표현하는 두 요소다.
이 두 요소에는 자연적인 순서도 있다. 반면 위의 writeField 예에서 outputStream과 name은 한 값을 표현하지도, 자연적인 순서가 있지도 않다.

심지어 아주 당연하게 여겨지는 이항 함수 `assertEquals(expected, actual)`에도 문제가 있다. expected 인수에 actual 값을 집어넣는 실수가 얼마나 많던가? 두 인수는 자연적인 순서가 없다.
인위적으로 기억해야 한다.

이항 함수가 무조건 나쁘다는 소리는 아니다. 불가피한 경우도 생긴다. 하지만 그만큼 위험이 따른다는 사실을 이해하고 가느하면 단항 함수로 바꾸도록 애써야 한다.


**인수 객체** <br>
인수가 2~3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어본다. 예를 들어, 다음 두 함수를 살펴보자.

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

객체를 생성해 인수를 줄이는 방법이 눈속임이라 여겨질지 모르지만 그렇지 않다. 위 예제에서 x와 y를 묶었듯이 변수를 묶어 넘기려면 이름을 붙여야 하므로 결국은 개념을 표현하게 된다.

### 부수 효과를 일으키지 마라!
함수에서 한 가지를 하겠다고 약속하고선 남몰래 다른 짓도 하니까 부수 효과는 거짓말이다. 많은 경우 시간적인 결합이나 순서 종속성을 초래한다.

```java
// 목록 3-6
public boolean checkPassword(String userName, String password) {
    User user = UserGateWay.findByName(userName);
    if (user != User.NULL) {
        String codedPhrase = user.getPhraseEncodedByPassword();
        String phrase = cryptographer.decrypt(codedPhrase, password);
        if ("Valid Password".equals(phrase)) {
            Session.initialize();
            return true;
        }
    }
    return false;
}
```

여기서 함수가 일으키는 부수 효과는 Session.initialize() 호출이다. checkPassword 함수는 이름 그대로 암호를 확인한다. 이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다.
그래서 함수 이름만 보고 함수를 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처한다.

이런 부수 효과가 시간적인 결합을 초래한다. 즉, checkPassword 함수는 특정 상황에서만 호출이 가능하다. 다시 말해, 세션을 초기화해도 괜찮은 경우에만 호출이 가능하다. 자칫 잘못 호출하면
의도하지 않게 세션 정보가 날아간다. 목록 3-6은 checkPasswordAndInitializeSession 이라는 이름이 훨씬 좋다. 물론 함수가 '한 가지'만 한다는 규칙을 위반하지만.

**출력 인수** <br>
일반적으로 우리는 인수를 함수 입력으로 해석한다. 인수를 출력으로 사용하는 함수에 어색함을 느끼리라. 예를 들어 다음 함수를 보자.

```java
appendFooter(s);
```

이 함수는 무언가에 s를 바닥글로 첨부할까? 아니면 s에 바닥글을 첨부할까? 인수 s는 입력일까 출력일까? 함수 선언부를 찾아보면 분명해진다.

```java
public void appendFooter(StringBuffer report)
```

인수 s가 출력 인수라는 사실은 함수 선언부를 찾아보고 나서야 알았다. 인지적으로 거슬린다는 뜻이므로 피해야 한다. 즉, appendFooter(s)는 무언가에 s를 바닥글로 첨부하는 흐름이 맞다.

출력 인수로 사용하라고 설계한 변수가 this이기 때문에 appendFooter는 report.appendFooter()로 호출하는 방식이 좋다. 일반적으로 출력 인수는 피해야 한다.
함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다.

### 명령과 조회를 분리하라!
함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다(객체 상태를 변경하거나 객체 정보를 반환하거나). 예를 들어, 다음 함수를 살펴보자.

```java
public boolean set(String attribute, String value);
```

이 함수는 이름이 attribute인 속성을 찾아 값을 value로 설정한 후 성공하면 true 실패하면 false를 반환한다. 그래서 다음과 같이 괴상한 코드가 나온다.

```java
if (set("username", "unclebob")) ...
```
독자 입장에서 코드를 읽어보자.
set을 형용사로 받아들이면, "만약 username이 unclebob으로 설정되어 있다면 ... " 으로 읽힌다.
set을 동사로 받아들이면, "만약 username을 unclebob으로 설정하는데 성공하면 ..." 으로 읽힌다.
함수를 호출하는 코드만 봐서는 의미가 모호하다. 해결책은 명령과 조회를 분리해 혼란을 애초에 뿌리뽑는 방법이다.

```java
if (attritubteExists("username")) {
    setAttribute("username", "unclebob");
}
```

### 오류 코드보다 예외를 사용하라!
명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다. 자칫하면 if문에서 명령을 표현식으로 사용하기 쉬운 탓이다.

```java
if (deletePage(page) == E_OK)
```

위 코드는 동상/형용사 혼란을 일으키지 않는 대신 여러 단계로 중첩되는 코드를 야기한다. 오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야 한다는 문제에 부딪힌다.

```java
if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
```

반면 오류 코드 대신 예외를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해진다.

```java
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}
catch (Exception e) {
    logger.log(e.getMessage());
}
```

try/catch 블록은 원래 추하다. 코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 뒤섞는다. 그러므로 try/catch 블록을 별도 함수로 뽑아내는 편이 좋다.

```java
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    }
    catch (Exception e) {
        logError(e);
    }
}

private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
    logger.log(e.getMessage());
}
```

## 4장 주석
코드로 의도를 표현하지 못해 주석을 사용한다. 표현력을 강화해서 애초에 주석이 필요 없는 방향으로 개발해야 한다.

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))
```

위 코드보다는 아래의 코드가 더 낫다.

```java
if (employee.isEligibleForFullBenefits())
```

## 5장 형식 맞추기
코드 형식(컨벤션)은 중요하다. 너무나도 중요하므로 융통성 없이 맹목적으로 따르면 안 된다. 코드 형식은 의사소통의 일환이다. 의사소통은 전문 개발자의 일차적인 의무다.
어쩌면 '돌아가는 코드'가 전문 개발자의 일차적인 의무라 여길지도 모르겠다. 하지만 이 책을 읽으면서 생각이 바뀌었기 바란다. 오늘 구현한 기능이 다음 버전에서 바뀔 확률은 아주 높다.
그런데 오늘 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다. 오랜 시간이 지나 원래 코드의 흔적을 더 이상 찾아보기 어려울 정도로 코드가 바뀌어도 맨 처음 잡아놓은 구현 스타일과
가독성 수준은 유지보수 용이성과 확장성에 계속 영향을 미친다. 원래 코드는 사라질지라도 개발자의 스타일과 규율은 사라지지 않는다.

## 6장 객체와 자료 구조
변수를 private으로 정의하는 이유가 있다. 남들이 변수에 의존하지 않게 만들고 싶어서다. 그렇다면 어째서 수많은 프로그래머가 getter/setter 함수를 당연하게 public해 private 변수를 외부에 노출할까?

### 자료 추상화
변수를 private으로 선언하더라도 각 값마다 getter/setter 함수를 제공한다면 구현을 외부로 노출하는 셈이다. 자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋다.
그러나 인터페이스나 getter/setter 함수만으로는 추상화가 이뤄지지 않는다. 개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민해야 한다.

```java
// 목록 6-3 구체적인 Vehicle 클래스
public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline();
}
```

```java
// 목록 6-4 추상적인 Vehicle 클래스
public interface Vehicle {
    double getPercentFuelRemaining();
}
```

목록 6-3은 자동차 연료 상태를 구체적인 숫자 값으로 알려준다. 목록 6-4는 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려준다.
목록 6-3은 두 함수가 변수값을 읽어 반환할 뿐이라는 사실이 거의 확실하다. 목록 6-4는 정보가 어디서 오는지 전혀 드러나지 않는다.

### 자료/객체 비대칭
객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다. 자료 구조는 자료를 그대로 공개하며 별다른 함수를 제공하지 않는다. 두 정의는 본질적으로 상반되고 두 개념은 사실상 정반대다.

```java
// 목록 6-5 절차적인 도형
public class Square {
	public Point topLeft;
	public double side;
}

public class Rectangle {
	public Point topLeft;
	public double height;
	public double width;
}

public class Circle {
	public Point center;
	public double radius;
}

public class Geometry {
	public final double PI = 3.141592;

	public double area(Object shape) throws Exception {
		if (shape instanceof Square) {
			Square s = (Square)shape;
			return s.side * s.side;
		} else if (shape instanceof Rectangle) {
			Rectangle r = (Rectangle)shape;
			return r.height * r.width;
		} else if (shape instanceof Circle) {
			Circle c = (Circle)shape;
			return PI * c.radius * c.radius;
		}
		throw new Exception();
	}
}
```

목록 6-5는 절차적인 도형 클래스다. Geometry 클래스는 세 가지 도형 클래스를 다룬다. 각 도형 클래스는 간단한 자료 구조다. 아무 메서드도 제공하지 않는다. 도형이 동작하는 방식은
Geometry 클래스에서 구현한다.

만약 Geometry 클래스에 둘레 길이를 구하는 perimeter() 함수를 추가하고 싶다면? 도형 클래스들은 아무 영향도 받지 않는다. 도형 클래스에 의존하는 다른 클래스도 마찬가지다. 반대로 새 도형을 추가하고 싶다면?
Geometry 클래스에 속한 함수를 모두 고쳐야 한다.

```java
/// 목록 6-6 다형적인 도형
public interface Shape {
	double area();
}

public class Square implements Shape {
	private Point topLeft;
	private double side;

	@Override
	public double area() {
		return side * side;
	}
}

public class Rectangle implements Shape {
	private Point topLeft;
	private double height;
	private double width;

	@Override
	public double area() {
		return height * width;
	}
}

public class Circle implements Shape {
	private Point center;
	private double radius;
	public final double PI = 3.141592;

	@Override
	public double area() {
		return PI * radius * radius;
	}
}

public class Geometry {
	public double area(Shape shape) {
		return shape.area();
	}
}
```

목록 6-6은 객체 지향적인 도형 클래스다. 여기서 area()는 다형 메서드다. 그러므로 새로운 도형을 추가해도 기존 함수에 영향을 미치지 않는다. 반면 새로운 함수를 추가하고 싶다면 도형 클래스 전부를 고쳐야 한다.
둘은 상호 보완적인 특질이 있다. 분별 있는 프로그래머는 모든 것이 객체라는 생각이 미신임을 잘 안다. 때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.

### 디미터 법칙
디미터 법칙은 잘 알려진 휴리스틱으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다. 좀 더 정확히 표현하자면, "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다.
1. 클래스 C
2. f가 생성한 객체
3. f 인수로 넘어온 객체
4. C 인스턴스 변수에 저장된 객체

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안된다.

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

흔히 위와 같은 코드를 기차 충돌(train wreck)이라 부른다. 일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다. 위 코드는 다음과 같이 나누는 편이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

위 예제가 디미터 법칙을 위반하는지 여부는 ctxt, Options, ScratchDir이 객체인지 아니면 자료 구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다. 반면, 자료 구조라면
당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```

위와 같이 구현했다면 디미터 법칙을 거론할 필요가 없어진다. 자료 구조는 무조건 함수 없이 공개 변수만 포함하고 객체는 비공개 변수와 공개 함수를 포함한다면 문제는 훨씬 간단해진다.

하지만 단순한 자료 구조에도 조회 함수와 설정 함수를 정의하라 요구하는 프레임워크와 표준(ex 빈)이 존재한다. 이런 혼란으로 말미암아 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다.
잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 getter/setter 함수도 있다. getter/setter 함수는 비공개 변수를 그대로 노출한다. 덕택에 다른 함수가 절차적인 프로그래밍의 자료 구조 접근 방식처럼
비공개 변수를 사용하픈 유혹에 빠지기 십상이다.

이런 잡종 구조는 되도록 피하는 편이 좋다. 프로그래머가 함수나 타입을 보호할지 공개할 지 확신하지 못해(더 나쁘게는 무지해) 어중간하게 내놓은 설계에 불과하다.

### 자료 전달 객체(DTO)
자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다. DTO라고 하는데 굉장히 유용한 구조체다. 특히 데이터베이스와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용하다.
좀 더 일반적인 형태는 빈(bean) 구조다.

```java
public class Address {
    private String street;
    private String city;

    // getter setter
}
```

일종의 사이비 캡슐화로 일부 OO 순수주의자나 만족시킬 뿐 별다른 이익을 제공하지 않는다.

DTO 클래스에 save나 find 같은 탐색 함수도 제공하는 활성 레코드가 있는데 그러면 자료 구조도 아니고 객체도 아닌 잡종 구조가 나온다.

## 7장 오류 처리
### null을 반환하지 마라
우리가 흔히 저지르는 바람에 오류를 유발하는 행위가 있다. 그 중 첫째가 null을 반환하는 습관이다.

```java
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = peristentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item);
            }
        }
    }
}
```

위 코드는 나쁜 코드다. null을 반환하는 코드는 일거리를 늘릴 뿐만 아니라 호출자에게 문제를 떠넘긴다. 누구 하나라도 null 확인을 빼먹는다면 애플리케이션이 통제 불능에 빠질지도 모른다.
peristentStore가 null이라면 NPE가 발생한다.

메서드에서 null을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환한다. 사용하려는 외부 API가 null을 반환한다면 감싸기 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려한다.

### null을 전달하지 마라
메서드로 null을 전달하는 방식은 더 나쁘다. 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다.
메서드는 전달받은 파라미터를 null 체크 해서 다른 예외로 처리하는 방법(throw new InvalidArgumentException)이 있고, assert 문을 이용하는 방법이 있다. 하지만 호출자가 실수로 넘기는 null 자체를
막을 방법은 없기 때문에 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.