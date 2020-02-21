## 쿠키 기본 성질
쿠키는 stateless(상태 정보를 유지하지 않는) HTTP protocol 에서 **stateful 하게 사용할 수 있게** 해주는 수단이다.

![](/assets/samesite3.png)

서버에서 응답 헤더로 위와 같은 Set-Cookie 헤더를 보내주면 이후 요청마다 브라우저가 헤더에 쿠키 정보를 같이 전송해준다.

![](/assets/samesite4.png)

그리고 admin.ybs.com 도메인으로 쿠키를 만들었을 때, 브라우저는 하위 도메인 요청에도 쿠키를 같이 전달한다.

## SameSite
### SameSite 배경
Set-Cookie: ybs=1234; Path=/; Domain= admin.ybs.com; **SameSite=Strict**<br>
위와 같이 쿠키에 SameSite 속성이 새롭게 추가 되었다.<br>

사실 새롭다고 하기는 뭐한게 2016년에 draft-west-first-party-cookies-05 문서에서 처음으로 등장했다.<br>
이후 RFC 6265 draft bis 02(2017-08-07 ~ 2018-02-08) 에서 추가되었고<br>
RFC 6265 draft bis 03(2019-04-27 ~ 2019-10-29) 를 거쳐<br>
RFC 6265 draft bis 04(2020-01-20 ~ 2020-07-23) 계속 상세화 되고 있다.<br>

SameSite 가 나온 배경을 살펴보면 RFC 문서에 다음과 같은 문장이 있다. 

**"SameSite" cookies offer a robust defense against CSRF attack when deployed in strict mode, and when supported by the client.**<br>
"SameSite" 쿠키는 strict 모드 일 때, 그리고 클라이언트 지원을 받을 때 CSRF(Cross-site request forgery) 공격에 대해 강력한 방어 기능을 제공한다.<br>

CSRF 공격을 막기 위한 보안 목적으로 SameSite 속성이 생겼고, 기존에 없던 새로운 제약이 생길거라는것을 알 수 있다.<br>
먼저 SameSite 속성에 대해서 자세히 살펴보고 왜 CSRF 공격을 막을 수 있는지 설명하겠다.<br>

### SameSite 기준 및 범위
Set-Cookie: ybs=1234; Path=/; Domain= admin.ybs.com; **SameSite=Strict**<br>
admin.ybs.com 도메인과 SameSite 가 아닌 경우, 브라우저에서 쿠키 생성과 전달을 제한한다는 뜻이다.

![](/assets/samesite5.png)

아래의 2번, 3번은 딱봐도 SameSite 가 아닌거 같다. 그런데 4번은 직관적으로 바로 판단이 안선다.<br> 
결국 SameSite 판단 기준을 정확히 아는게 중요하다.<br>

A request is "same-site" if its **target's URI's origin's registered domain** is an exact match for **the request's client's "site for cookies"**,
or **if the request has no client.** The request is otherwise "cross-site".<br>

위 문장은 SameSite 판단 기준에 대한 RFC 문서 내용이다. target's URI's origin's registered domain 과 request's client's "site for cookies" 가
정확히 일치하거나 request 가 client 를 갖지 않을 때 SameSite 라고 한다. 이것만 봐서는 이해하기 힘들다. 각 워딩 개념을 정리하고 다시 보도록 하자.

먼저 TLD(top level domain) 라는게 있다.<br>
TLD + 0 레벨(public suffix)에 해당하는 도메인들은 https://publicsuffix.org/list/public_suffix_list.dat 에서 관리하는데 
예를 들어 "com", "github.io", "co.kr” 이 있다.<br>
주의) TLD + 0 레벨은 마지막 dot(.) 기준 오른쪽 문자열이 아니다.

registered domain 은 TLD + 1 레벨의 도메인이다(중요).<br> 
ex) https://www.example.com 에서 public suffix 은 "com", registered domain 은 "example.com"<br>

![](/assets/samesite1.png)

그리고 브라우저 주소창 URI's origin 의 registered domain 을 top-level site 라고 한다.<br>
위와 같이 top-level browsing context 라면 "site for cookies" 는 top-level site 이다.<br>

![](/assets/samesite2.png)

nested-browsing context 라면 각 document 의 상위 browsing context 의 origin 을 확인해야 한다.<br>
하위 document 와 상위 document 의 origin 이 같은 registered domain 일 때만 "site for cookies" 는 top-level site 이다.<br>
즉, request's client's "site for cookies＂는 브라우저 주소창 URI 의 registered domain 이다.<br> 

**결론적으로 Set-Cookie Domain 의 registered domain 과 브라우저 주소창 URI 의 registered domain 이 정확하게 일치할 경우 SameSite 이다.**

![](/assets/samesite6.png)

위 그림을 보면 브라우저 주소창 URI 의 registered domain 은 ybs.com(TLD+1) 이다.<br>
그리고 Set-Cookie Domain 의 registered domain 도 ybs.com(TLD+1) 이니 SameSite 이다.<br>

RFC 문서에서 request 가 client 를 갖지 않을 때도 SameSite 라고 했는데, 이는 브라우저 주소 창에서 직접 타이핑 쳐서 html 이 바뀌는 경우를 말한다.<br>

아래에 SameSite 인지 CrossSite 인지 판단하는 알고리즘을 보면, 브라우저 주소 창에서 직접 타이핑 쳐서 html 이 바뀔 때 request 의 client 가 null 이 되고 
SameSite 로 리턴되는것을 알 수 있다. 

```
For a given request ("request"), the following algorithm returns "same-site" or "cross-site":
1. If "request"'s client is "null", return "same-site".
Note that this is the case for navigation triggered by the user directly (e.g. by typing directly into a user agent's address bar).
2. Let "site" be "request"'s client's "site for cookies" (as defined in the following sections).
3. Let "target" be the registered domain of "request"'s current url.
4. If "site" is an exact match for "target", return "same-site".
5. Return "cross-site".
```

### SameSite 옵션
옵션은 총 3가지가 있다.

**1. SameSite=Strict**<br>
모든 cross-site subresource requests, cross-site nested navigations 상황에서 쿠키 생성 및 전달을 허용하지 않는다.
여기서 subresource requests 는 html 파일 안에서 js, css, img 등의 리소스 가져올 때를 말한다. navigation 개념도 중요한데 이는 다른 옵션에서 자세하게 설명하겠다.<br>

**2. SameSite=None**<br>
반드시 secure 속성이 함께 추가되어야 동작한다. 즉 HTTPS 만 가능하다.<br>

![](/assets/samesite7.png)

**3. SameSite=Lax**<br>
top-level navigations(which use a safe HTTP method) 일 경우에만 cross-site 상황에서 쿠키 생성 및 전달을 허용한다.

먼저 safe HTTP method 는 "GET", "HEAD", "OPTIONS", and "TRACE" 이다(RFC7231).  

navigation 은 서버에서 html 을 보내줘서 페이지 이동이 되는 케이스를 말한다.<br>
웹페이지에서 링크를 눌렀을 때 보통 2가지 flow 가 생긴다.<br>
첫째, 서버에서 html 을 보내줄 때<br> 
둘째, 서버에서 다운로드 파일을 보내줄 때

이 중 첫번째에 해당하는게 navigation 이다.

top-level navigation 은 main frame 이 이동하는 케이스를 말한다.
iframe 이 있는 페이지 경우, iframe 내부 이동은 top-level navigation 이 아니기 때문에 구분 짓기 위한 개념이다.

예를 들어 메일을 읽고 있는데 내용에 다른 링크 주소가 있다고 하자.

![](/assets/samesite8.png)

만약 projects.example 이 SameSite 이면 cross-site navigation 이기 때문에 인증에 실패할 수 있다.<br>
그럴 경우 오동작하는 사이트들이 많아서 top-level navigations(safe HTTP method) 에 한해 third-party 쿠키를 허용해주는 Lax 모드를 지원하는 것이다.<br>

**Set-Cookie 에 SameSite 속성이 없는 경우**<br>
Incrementally Better Cookies draft-west-cookie-incrementalism-00(2019-05-07 ~ 2019-11-08) 에서는 SameSite=Lax 로 한다고 되어 있다.

chromium FAQ 에서는 SameSite=Lax 로 한다고 되어 있다.

RFC 6265 draft bis 04(2020-01-20 ~ 2020-07-23) 문서는 SameSite=None 으로 되어 있다.

브라우저별 업데이트 일정<br>
chrome : 80 update(2020-02-04)부터 적용한다고 했는데 2020-02-17 부터 단계적으로 변경<br>
safari : 업데이트 할거란 글은 있지만 일정 공유 안함<br>
ie11 : 업데이트 한다는 얘기는 있지만 관련 히스토리는 없음<br>
edge : 2020-01-15 chromium 기반으로 변경. SameSite 작업은 chrome 과 동시 또는 나중에 한다고 함<br>
firefox : 69버전에서 구현완료(기본값 변경은 하지는 않아서 당장은 문제 안됨)<br>

**Set-Cookie 에  SameSite=Strict; SameSite=Lax; SameSite=None 여러개 붙은 경우**<br>
마지막 SameSite=None 을 선택한다.

### 서블릿/톰캣/스프링 SameSite 속성 지원 

**1. Servlet**<br>
java servlet api 4.0 Cookie 클래스에도 SameSite 설정을 위한 메서드는 없다. 
SameSite 속성 추가 PR 진행중 https://github.com/eclipse-ee4j/servlet-api/pull/271 (2020-02-20 기준)

**2. Spring Framework**<br>
spring-web module 5.1 버전 Spring WebFlux 에서 ResponseCookie 클래스에 SameSite 속성이 추가되었다.

**3. Tomcat**<br>
2019-06-07 release 된 Tomcat 9.0.21 버전과 Tomcat 8.5.42 버전에서 context.xml 에 sameSiteCookies 속성을 추가하여 
일괄적으로 SameSite 속성을 set-Cookie 헤더에 추가할 수 있다(Tomcat 7.x 버전은 지원하지 않음).

```xml
<Context>
	<CookieProcessor sameSiteCookies="strict" />
</Context>
```
### CSRF

![](/assets/samesite9.png)

CSRF 공격은 ybs.com 에 대한 인증을 받은 상태라는 전제가 필요하다. 만약 쿠키 SameSite 옵션이 Strict 라면, CrossSite 일 때 쿠키 전달을 차단 할 수 있다.

### 참고로 알아두면 좋은 정보

**1. RFC 문서 볼 때 동사 주의**<br>
Must, Shall, Required(반드시 지켜야 하는 스펙)<br>
Should(정말 타당한 이유가 있을 때 안지켜도 되지만 신중하고 정확히 이해되야 함)<br> 
May(필수가 아님, 구현 안해도 되지만 구현한거랑 안한거랑 상호운용이 가능해야함)<br>
https://tools.ietf.org/html/rfc2119<br>

**2. 일부 user-agent 에서는 Domain 속성이 없을때 현재 Host 로 쿠키를 저장(RFC6265)**<br>

![](/assets/samesite10.png)

만약 Host 로 쿠키를 생성하면, admin.ybs.com 도메인 한정이다.<br>
test.admin.ybs.com 같은 서브 도메인에는 쿠키가 전달 되지 않는다.<br>