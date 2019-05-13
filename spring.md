소스를 다운받자마자 메소드를 인식 못한다는 에러가 발생했다. 이유는 lombok 때문(spring boot에서 지원안해줌)) !! 

lombok이 뭔고하니 
```java
@Entity
@Getter
@Setter
public class Account {

    @Id @GeneratedValue
    private Long id;

    @Column(unique = true)
    private String username;

    private String password;

    private String email;

    private String fullName;

    @Temporal(TemporalType.TIMESTAMP)
    private Date joined;

    @Temporal(TemporalType.TIMESTAMP)
    private Date updated;

}
```
위 처럼 getter/setter를 줄여주는 기능이다.

lombok을 사용하기 위해서는 

1. lombok plugin 설치

![](/assets/lombok1.PNG)

2. lombok 설정

![](/assets/lombok2.PNG)

AccountController에서 

클래스에 @Controller, @ResponseBody 두개를 붙이면 이 클래스안의 모든 public 메소드에 @ResponseBody가 다 적용이 되는것이다.
(@RestContoller = @Controller + @ResponseBody)
요즘은 서버사이드 개발이 API개발로 바뀌었기 때문에 @RestController 쓰는게 무방하다. 

메인 메소드를 실행하면 내장 임베디드 톰캣이 실행되면서 뜬다. 

Account 클래스를 만드는데 

``` java
@Id @GeneratedValue
private Long id;
```
@Id는 해당 프로퍼티가 테이블의 기본키 역할을 한다는 것을 나타낸다. @GeneratedValue는 기본키의 값을 위한 자동 생성 전략을 명시하는데 사용한다. 

AccountRepository는 jpa사용해서 만듦 두번째 인자는 pk 데이터타입
``` java
extends JpaRepository<Account, Long>
```

AccountService 클래스는 @Service 등록해서 component scan으로 빈으로 등록되게끔만 하고 @Autowired로 AccountRepository를 가지고 있게끔만 한다. 그리고 @Transactional 붙여주면 이 클래스 안에 만드는 모든 public 메소드는 다 transactional 애노테이션이 적용된다. 

AccountController에서 
``` java
public ResponseEntity createAccount(@RequestBody @Valid AccountDto.Create create,
                                          BindingResult result) {
```
@RequestBody 대신에 @RequestParam, @ModelAttribute 많이 썼었는데 요즘은 서버 사이드 개발이 주로 REST API쪽으로 많이 가다보니까 Request 본문에 들어오는걸 파싱받는게 많아져서 @RequestBody를 많이 씀
@RequestBody를 쓰면 메세지 컨버터가 작동됌 (@RequestParam, @ModelAttribute와는 다르게 바인딩해줌) 
spring-boot를 쓰게되면 여러개의 메세지 컨버터가 이미 등록이 되어 있어 

그런데 Acount 객체를 바로 받는게 아니라 AccountDto 객체를 따로 만들어서 (내가 받을것만 또 지정해주는) 그걸로 바인딩 되게 하면 Acoount객체의 모든 변수를 바인딩 받지 않을 때 헷갈려 지지 않는다. 받을것만 AccountDto를 만들어서 넣게끔하니까 

AccountDto 클래스 가보면 @NotBlank @Size가 적용되어 있는데 그걸 붙였다고 바로 검증이 되진 않는다. 
실제 검증을 하려면 createAccount메소드 파라미터로 @Valid 꼭 붙여 줘야함

**질문** 
- 테스트에 @Transactional에 붙으면 자동으로 롤백이 된다고 말씀하셨는데 제가 아는 롤백이란 개념은 
'데이터베이스에서 업데이트에 오류가 발생할 때, 이전 상태로 되돌리는 것' 인데 테스트에서 롤백이 된다라는 부분이 잘 와닿지 않습니다.


답변 : 테스트의 모든 transactional은 에러가 나든, 안나든 롤백이 되도록 설정이 되어 있다. 롤백은 데이터의 변경사항을 commit하지 않고 되돌리는것 ex) 비지니스 로직상, 트랜잭션의 타임아웃 

테스트코드를 메모리 DB쓰지않고 Mysql을 썼다고 가정하면 하나의 @Test의 DB 작업이 
다른 @Test에 영향을 끼칠 수 있다. 테스트 간의 의존성이 생기기 때문에 테스트를 만드는게 
어려워진다. 그래서 DB에 기록을 남기지 않으려고 so @Transactional을 붙임으로써 기본적으로 롤백이 되게끔 한다. 


- 테스트에 붙은 @Transactional 과 서비스에 있는 @Transactional과 정확히 어떤 차이가 있는건가요?
 

답변 : 테스트의 @Transactional이 기본적으로 롤백이 디폴트로 적용되는거 밖에 없다. 


TransactionConfiguration는 class-level
rollback은 method 레벨 
근데 4.2 버전부터는 Rollback이 다 한다. 
TransactionConfiguration는 desperated 될것. 

![](/assets/sdfsdf2.PNG)

ErrorResponse 클래스에 @Data붙이면 lombok에 의해 getter, setter 만들어짐 

**Maven에서 dependency 의 scope 설정**
compile(default) : 컴파일 시 라이브러리를 사용한다
runtime : 실행 시 라이브러리를 사용한다
provided : 외부에서 라이브러리가 제공된다. 컴파일 시 사용하지만 빌드에 포함하지 않는다. 보통 JSP, Servlet 라이브러리들에 사용한다.
test : 테스트 코드에만 사용한다.

```xml
<mvc:annotation-driven/> 
<context:component-scan base-package=""/>
```

base-package포함, 하위의 클래스들 중 @Controller, @Repository, @Service, @Component가 붙어 있는 클래스들을 자동으로 스프링 빈으로 등록한다. 

```xml
<tx:annotation-driven/>
<context:component-scan base-package=""/>
```
bace-package포함, 하위의 클래스들 중 @Transcational이 붙은 곳에 트랜잭션을 적용한다.

`<context:component-scan> / <mvc:annotation-driven> / <context:annotation-config> 차이점`
1. context:component-scan
    1. 특정 패키지안의 클래스들을 스캔하고, 빈 인스턴스를 생성한다.
    2. @Component @Controller @Service @Repository 애노테이션이 존재해야 빈을 생성할 수 있다. 
    3. 이것의 장점 중 하나는 @Autowired 와 @Qualifier 애노테이션을 이해한다는 것인데 component-scan을 선언했다면 context:annotation-config를 선언할 필요가 없다.
2. mvc:annotation-driven
    1. 스프링 MVC 컴포넌트들을 그것의 디폴트 설정을 가지고 활성화 하기위해 사용된다. 만약 context:component-scan을 XML 파일에서 빈을 생성하기 위해 사용하면서 mvc:annotation-driven을 포함시키지 않아도 MVC 애플리케이션은 작동할 것이다. 그러나 mvc:annotation-driven은 특별한 일들을 하는데 이 태그는 당신의 @Controllers에게 요청을 전파하기위해 요구되는 HandlerMapping과 HandlerAdapter를 등록한다. 게다가, 클래스패스상에 존재하는 디폴트 작업을 수행한다. 
3. context:annotation-config
    1. context:annotation-config은 애플리케이션 컨텍스트안에 이미 등록된 빈들의 애노테이션을 활성화하기 위해 사용된다.(그것들이 XML로 설정됐는지 혹은 패키지스캐닝을 통한건지는 중요하지 않다.) 그 의미는 이미 스프링 컨텍스트에 의해 생성되어 저장된 빈들에 대해서 @Autowired와 @Qualifier 애노테이션을 해석할거란 얘기다. component-scan 또한 같은일을 할 수 있는데, 추가적으로 애플리케이션 컨텍스트에 빈을 등록하기위한 패키지들을 스캔한다. context:annotation-config는 빈을 등록하기 위해 검색할 수 없다. 
    2. context:annotation-config 태그를 설정하면 @Required @Autowired @Resource @PostConstruct @PreDestroy @Configuration 기능을 각각 설정하는 수고를 덜게 해준다. 


**mvc:resources**

```xml
<mvc:default-servlet-handler default-servlet-name="default"/>
```
DispatcherServlet이 처리하지 못한 요청을 서블릿 컨테이너의 DefaultServlet에게 넘겨주는 역할을 하는 핸들러이다. 

/js/jquery.js 처럼 컨트롤러에 매핑안되는 URL같은 경우는 DefaultServletHttpRequestHandler가 담당한다. 이 핸들러는 매핑 우선순위가 가장 낮아서 애노테이션 매핑 등등을 거쳐서 다 실패한 URL만 넘어온다. 그리고 요청을 자신이 직접 읽어서 처리하는 것이 아니라, 원래 서버가 제공하는 디폴트 서블릿으로 넘겨버린다. 그러면 서버의 기본 디폴트 서블릿이 동작해서 스태틱리소스를 처리하는 것이다. 다시말해 일단 스프링이 다 받고 스프링이 처리 못하는 건 다시 서버의 디폴트 서블릿으로 넘긴다는 아이디어이다. 

`*.ico`파일 처리 못하는 현상은 `web.xml`에 ico의 MIME타입을 지정해주면 된다. 
```xml
<mime-mapping>
    <extension>ico</extension>
    <mime-type>image/vnd.microsoft.icon</mime-type>
</mime-mapping>
```

**정적자원 설정하기**
CSS,JS,이미지 등의 자원은 거의 변하지 않기 때문에, 웹 브러우저에 캐시를 하면 네트워크 사용량, 서버 사용량, 웹 브라우저의 반응 속도 등을 개선할 수 있다. 스프링 MVC를 이용하는 웹 애플리케이션에 정적 자원 파일이 함께 포함되어 있다면 웹 서버 설정을 사용하지 않고 캐시를 사용하도록 지정할 수 있다. 

```xml
<mvc:resources mapping="/resources/**" location="/resources/" cache-period="60"/>
```

mapping : 요청 경로 패턴을 설정한다. (컨텍스트 경로를 제외한 나머지 부분의 경로)
location : 웹 애플리케이션 내에서 요청 경로 패턴에 해당하는 자원의 위치를 지정한다. 위치가 여러곳일 경우 각 위치를 콤마로 구분한다.
cache-period: 웹 브라우저에 캐시 시간 관련 응답 헤더를 전송한다. 초 단위로 캐시 시간을 지정하며 이 값이 0이면 웹 브라우저가 캐시하지 않도록 한다. 

위 설정의 경우 요청 경로가 /resources/로 시작하면, 그에 해당하는 자원을 /resources/나 /WEB-INF/resources/ 디렉토리에서 검색한다. 

**빈 설정방식의 변화**

![](/assets/bean설정방식의변화.PNG)

1.x : 모든걸 `<bean> </bean>`으로
2.0.x : `<tx:annotation-driven`
2.5x : @Service, @Repository 같이 설정 `<context:component-scan base-package="~"`
3.0.x : @Configuration, @Bean
3.1.x : @Enable~ ex) @EnableTransactionManagement

![](/assets/bean설정방식의변화2.PNG)

![](assets/ooo.PNG)

@EnableWebMvc는 `<mvc:annotation-driven>`과 똑같다. 따라서 애노테이션 드리븐을 설정하면 위의 주석에 해당하는 것들이 자동으로 등록이 된다. 기본적으로 Http메세지 컨버터도 등록이 되지만, JSON을 위한 MappingJackson2HttpMessageConverter는 직접 등록해줘야 사용할 수 있다. 

---
**웹 환경에서 스프링 애플리케이션이 기동하는 방식**

![](/assets/webapplicationcontext.PNG)

서블릿 컨테이너는 브라우저와 같은 클라이언트로부터 들어오는 요청을 받아서 서블릿을 동작시켜주는 일을 맡는다. 서블릿은 웹 애플리케이션이 시작될 때 미리 만들어둔 웹 애플리케이션 컨텍스트에게 빈 오브젝트로 구성된 애플리케이션의 기동 역할을 해줄 빈을 요청해서 받아둔다. 그리고 미리 지정된 메서드를 호출함으로써 스프링 컨테이너가 DI 방식으로 구성해둔 애플리케이션의 기능이 시작되는 것이다. 

스프링은 이런 웹 환경에서 애플리케이션 컨텍스트를 생성하고 설정 메타정보로 초기화해주고, 클라이언트로부터 들어오는 요청마다 적절한 빈을 찾아서 이를 실행해주는 기능을 가진 DispatcherServlet이라는 이름의 서블릿을 제공한다. DispatcherServlet은 서블릿이 초기화 될때 자신만의 컨텍스트를 생성하고 초기화한다.