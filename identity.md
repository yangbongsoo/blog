#Identity

###SAML 기반의 web sso 원리 정리
통합인증(SSO, Single Sign On)은 한 번의 인증 과정으로 여러 컴퓨터 상의 자원을 이용 가능하게 하는 인증 기능이다. 싱글 사인온, 단일 계정 로그인, 단일 인증이라고 한다.

보안이 필요한 환경에서 통합인증을 도입하는 경우, 여러 응용 프로그램의 로그인 처리가 간소화되어 편리성을 도모할 수 있는 반면, 통합인증의 시작점이 되는, 즉 최초의 로그인 대상이 되는 응용 프로그램 혹은, 운영체제에 대한 접근 보안이 중요하게 된다. 보안위험이 적은 환경에서는 편리성만을 추구하면 되지만, 보안이 요구되는 환경에서는 1회용 비밀번호를 이용하는 등, 이중 인증 등으로 보안을 강화할 필요가 있다.

Single Sign On을 지원하기 위한 프로토콜이나 방법은 여러가지가 있다. 
그중 대표적인 방법으로 **CAS,SAML,OAuth**등이 있는데, CAS는 쿠기를 기반으로 하기 때문에 같은 도메인명 (xxx.domain.com yyy.domain.com) 사이에서만 SSO가 가능하다. (그만큼 구현도 쉽다.) OAuth는 현재 B2C쪽에 많이 사용되는 프로토콜이고, 그리고 마지막으로 SAML 있다. cross domain간 SSO 구현이 가능하며, OAuth 만큼이나 많이 사용되고 있다.

SAML은 어떤 구현체가 아니라 SSO등(꼭 SSO만은 아님)을 구현하기 위한 XML 스펙이다.

HTTP GET, POST 또는 SOAP 웹서비스 등 여러가지 방법으로 구현될 수 있으며, 여기서는 HTTP Post를 이용한 SSO 원리와 솔루션 설계시 유의 사항을 설명한다.

**site SpA로 초기 로그인**
![](/assets/SAML_sso1.png)

1. Browser에서 사이트 SpA로 접속한다.
2. 사이트 SpA에는 로그인이 되어 있지 않기 때문에 (세션이 없어서). SpA에서는 SAML request를 만들어서, Browser에게로 redirect URL을 보낸다.
3. Browser는 redirect URL에 따라 IdP에 접속하고, Idp에서 login form을 넣고 log in을 한다. 이때, IdP와 Brower 사이에 HttpSession 또는 Cookie로 Login에 대한 정보를 기록한다. 그리고 다시 사이트 SpA로의 SAML response를 포함한 redirect URL을 browser로 전송한다.
4. Browser는 SAML reponse를 가지고 SpA로 접속하면, SpA에는 인증된 정보를 가지고 로그인 처리를 한다. ※ 이 과정에서는 바로 사이트 SpA의 사용자 페이지(예를 들어 /home)등으로 가는 것이 아니라, SAML에 의해서 미리 정의한 SpA의 SAML response 처리 URL로 갔다가 SAML response를 처리가 끝나면 인증 처리를 한후, 사용자 페이지(/home)으로 다시 redirect한다.


**site SpA로 로그인된 상태에서 site SpA로 로그인**
![](/assets/SAML_sso2.png)

1. 사이트 SpA에서 로그인된 상태에서 SpB에 접속한다.
2. 사이트 SpB는 로그인이 되어 있지 않기 때문에, SAML 메시지를 만들어서 IdP의 login from으로 redirect URL을 보낸다.
3. 브라우져는 redirect URL을 따라서 IdP에 접속을 한다. IdP에 접속을 하면 앞의 과정에서 이미 Session 또는 Cookie가 만들어져 있기 때문에 별도의 로그인 폼을 띄위지 않고, SAML response message와 함께, SpB로의 redirect URL을 전송한다. 
4. Browser는 Sp B에 인증된 정보를 가지고 로그인한다.


**SAML 기반의 SSO 솔루션**
simplePHPSAML - 가장 널리 쓰이고, 사용이 쉽다.

Shibboleth - java stack으로 구현이 되어 있으며, terracotta를 이용하여 session을 저장하기 때문에 상대적으로 확장성이 높다.

WSO2 identity server - 앞에서도 언급하였듯이, 확장성에 문제가 있는 것으로 보이며, 자체 OSGi 컨테이너인 carbon 엔진 위에서 동작한다. SAML 뿐만 아니라 OAuth,STS 서비스를 추가 지원하며, Provisioning protocol인 SCIM도 함께 지원한다. 오픈 소스이지만, 제품 완성도가 매우 높고, 사용이 매우 쉽다. (모니터링,관리 기능등이 강점)

Open AM - Sun IDM을 모태로 하여, 현재 오픈소스화 되었다. 아무래도 enterprise 제품을 기반으로 하다 보니 복잡도가 상대적으로 높다.

**[원문]**
위키 : https://ko.wikipedia.org/wiki/%ED%86%B5%ED%95%A9_%EC%9D%B8%EC%A6%9D 
조대협 블로그 : http://bcho.tistory.com/755

###Facebook OAuth
추가적인 보안 강화를 위해 사용자 인증(ID, Password)뿐만 아니라, 클라이언트 인증 방식을 추가할 수 있다. 페이스북은 API 토큰을 발급받도록 사용자 ID, 비밀번호 뿐만 아니라 Client ID와 Client Secret이라는 것을 같이 입력받도록 하는데, Client ID는 특정 앱에 대한 등록 ID이고 Client Secret은 특정 앱에 대한 비밀번호로, 페이스북 개발자 포털에서 앱을 등록하면 앱 별로 발급되는 일종의 비밀번호다.

![](/developerfacebook.PNG)

API 토큰을 발급받을 때, Client ID와 Client Secret을 이용하여 클라이언트 앱을 인증하고 사용자 ID와 비밀번호를 추가로 받아서 사용자를 인증해 API 액세스 토큰을 발급한다. 

**제 3자 인증 방식(OAuth 2.0 Autorization grant type)**

제 3자 인증 방식은 페이스북이나 트위터와 같은 API 서비스 제공자들이 파트너 애플리케이션에 많이 적용하는 방법으로 자신의 서비스를 페이스북 계정을 이용하여 인증하는 경우다.

중요한 점은 자신의 서비스에서 사용자 비밀번호를 받지 않고, 페이스북이 사용자를 인증하고 알려주는 방식이다. 즉, 파트너 서비스에는 페이스북 사용자의 비밀번호가 노출되지 않는 방식이다. 전체적인 흐름을 보면 다음과 같다.

![](apitokenflow.PNG)

1.먼저 페이스북의 개발자 포털에 접속하여, 페이스북 인증을 사용하고자 하는 애플리케이션 정보를 등록한다(서비스명, 서비스 URL, 그리고 인증이 성공했을 때 인증 성공 정보를 받을 콜백 URL).

![](/oauthsetting.PNG)

2.페이스북 개발자 포털은 등록된 정보를 기준으로 해당 애플리케이션에 대한 Client ID와 Client Secret을 발급한다. 이 값은 앞에서 설명한 클라이언트 인증에 사용된다.

3.다음으로 개발하고자 하는 애플리케이션에 이 Client ID와 Client Secret을 넣고, 페이스북 인증 페이지 정보를 넣어서 애플리케이션을 개발한다(Javascript SDK 적용).

![](/fbsdk.PNG)

애플리케이션이 개발되서 실행되면 다음과 같은 절차로 사용자 인증을 수행하게 된다.

![](/스크린샷 2016-11-08 오후 6.47.36.jpg)

1. 웹 브라우저에서 사용자가 Sokit 서비스에 접근하려고 요청한다.
2. Sokit 서비스는 사용자 인증이 되지 않았기 때문에 페이스북 로그인 페이지 URL을 HTTP  리다이렉션으로 브라우저에 보낸다. 이때 이 URL로 페이스북에 이 로그인 요청이 Sokit에 대한 사용자 인증 요청임을 알려주고자, Client ID 등의 추가 정보와 함께 페이스북 정보 접근 권한(사용자 정보, 그룹 정보 등)을 scope라는 필드를 통해서 요청한다.
3. 브라우저는 페이스북 로그인 페이지로 이동하여 2단계에서 받은 추가적인 정보와 함께 로그인을 요청한다.
4. 페이스북은 사용자에게 로그인 창을 보낸다.
5. 사용자는 로그인 창에 ID/비밀번호를 입력한다.
6. 페이스북은 사용자를 인증하고 인증 관련 정보와 함께 브라우저로 전달하면서 Sokit의 로그인 완료 페이지로 리다이렉션을 요청한다.
7. Sokit은 6에서 온 인증 관련 정보를 받는다.
8. Sokit은 이 정보를 가지고 페이스북에 이 사용자가 제대로 인증을 받은 사용자인지 문의한다.
9. 페이스북은 해당 정보를 보고 제대로 인증된 사용자임을 확인해주고 Access Token을 발급한다.
10. Sokit은 9에서 받은 Access Token으로 페이스북 API 서비스에 접근한다.

![](/fblogin.PNG)

![](/fbscope.PNG)

###PingFederate(SSO)



                                                                                                                                                                                                                                                                                                                                                                                                                                                                          

