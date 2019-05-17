# 클린코드
## 함수
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

