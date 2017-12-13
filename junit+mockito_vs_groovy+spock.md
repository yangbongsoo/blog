# JUnit+Mockito vs Groovy+Spock

**발표 순서**
1. Spock 기본적인 문법
2. 블로그에서 groovy를 이용한 통합테스트 방식 소개
3. 간단한 Spock 적용 후기

cf) 마지막 부분에 Spring Boot 1.4 Test방식 소개

## Spock
참고 : http://thejavatar.com/testing-with-spock/ 
참고 : http://farenda.com/spock-framework-tutorial/

먼저 의존성 추가 
```xml
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-core</artifactId>
    <version>1.0-groovy-2.4</version>
</dependency>
```
스프링 프로젝트
```xml
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-spring</artifactId>
    <version>1.0-groovy-2.4</version>
</dependency>
```

첫 예제
![](/assets/mockitovsgroovy1.jpg)
![](/assets/mockitovsgroovy2.jpg)

3개의 섹션으로 나눠진다(BDD에 기반해서 given when then).

cf) expect는 간단한 테스트할 때 
```groovy
class SpockNameInverterTest extends Specification{
    def "NameInverter 테스트"(){

        expect:
        invert(null) == ""
    }

    private String invert(String name){
        return null;
    }
}
```

### Stub
```groovy
def "creating example stubs"() {
   given:
      List list = Stub(List)
 
      List list2 = Stub() // preffered way
 
      def list3 = Stub(List)      
}
```

```groovy
def "Stub 사용법"() {
        given:
        List list = Stub()
        list.size() >> 3
        
        expect:
        list.size() == 3
    }
```

조금 더 난이도 있는 UserService 예제
```java
public class User {
    String name;
}
 
public interface UserService {
    void save(User user);
}
```
```groovy
def "유저 이름이 Norman이면 exception, 유저이름이 R이면 정상처리"() {
        given:
        UserService service = Stub()
        service.save({ User user -> 'Norman' == user.name }) >> {
            throw new IllegalArgumentException("We don't want you here, Norman!")
        }

        when:
        User user = new User(name: 'Norman')
        service.save(user)
        then:
        thrown(IllegalArgumentException)

        when:
        User user2 = new User(name: 'R')
        service.save(user2)
        then:
        notThrown(IllegalArgumentException)
    }
```

**wildcard**
```groovy
given:
    // 모든 인자 허용 
    list.contains(_) >> true
 
    // 모든 Integer 인자 허용
    list.add(_ as Integer) >> true
 
    // null이 아니면 허용
    list.add(!null) >> true
 
    // someObject가 아니면 허용
    list.add(!someObject) >> true
```
```groovy
def "만약 리스트에 Integer 추가하면 예외처리"() {
        given:
        List list = Stub()
        list.add(_ as Integer) >> { throw new IllegalArgumentException() }

        when:
        list.add(2)
        then:
        thrown(IllegalArgumentException)

        when:
        list.add("String")
        then:
        notThrown(IllegalArgumentException)
    }
```
cf) JDK7에서 새롭게 소개된 Invokedynamic. 자바는 static type 언어라고 불리며, 이는 컴파일 타임에서 이미 멤버 변수들이나 함수 변수들의 타입이 반드시 명시적으로 지정돼야 함을 의미한다. 그에 반해 루비나 자바스크립트는 이른바 ‘duck-typing’이라고 하는 타입 시스템을 사용함으로써 컴파일 타임에서의 타입을 강제하지 않는다. Invokedynamic은 이러한 duck-typing을 JVM레벨에서 기본적으로 지원하면서 자바 외에 다른 언어들이 JVM이라는 플랫폼 위에서 최적화된 방식으로 실행될 수 있는 토대를 제공한다.

### Mock
```groovy
def "creating example mocks"() {
   given:
      List list = Mock(List)
 
      List list2 = Mock() // preffered way
 
      def list3 = Mock(List)      
}
```
Dummy 객체 자체를 테스트하기보다 여러 인터페이스들이 연결되어 있는 특정 메서드를 체크하는게 더 관심있을 때 Mock이나 Spy를 쓴다. 

**cf) Stub vs Mock**
Stub 은 테스트 과정에서 일어나는 호출에 대해 지정된 답변을 제공하고, 그 밖의 테스트를 위해 별도로 프로그래밍 되지 않은 질의에 대해서는 대게 아무런 대응을 하지 않는다.

Mock Object 는 검사하고자 하는 코드와 맞물려 동작하는 객체들을 대신하여 동작하기 위해 만들어진 객체이다. 검사하고자 하는 코드는 Mock Object 의 메서드를 부를 수 있고, 이 때 Mock Object는 미리 정의된 결과 값을 전달한다.

cf) 토비의 스프링에서 Stub과 Mock 비교
Stub은 테스트 대상 Object의 의존객체로 존재하면서 테스트 동안에 코드가 정상적으로 수행할 수 있도록 돕는다.
때론 테스트 대상 Object가 의존 Object에게 출력한 값에 관심이 있거나, 의존 Object를 얼마나 사용했는가 하는 커뮤니케이션 행위 자체에 관심이 있을 수 있다. 문제는 이 정보를 테스트에서는 직접 알 수가 없기 때문에 목 객체를 만들어서 사용해야 한다.

### Spy
Stub이나 Mock과는 다르게 Spy는 Dummy 객체가 아니다. Spy는 실제 일반 객체를 감싼것이다. Spy를 만들 때는 interface로 만들지 않고 class로 만들어야 한다. 
```groovy
def "interface로 Spy 만들면 안된다."() {
        given:
        UserService service = Spy(UserService)
        
        expect:
        service.save(new User(name: 'Norman'))
    }
    
결과 : Cannot invoke real method on interface based mock object    
```
아래의 예제는 Transaction 객체를 생성자 인수로 받는 UserServiceImpl 클래스를 Spy한다.
```java
public interface Transaction { }

public interface UserService {
    boolean isServiceUp();
    void save(User user);
}

public class UserServiceImpl implements UserService{

    public UserServiceImpl(Transaction transaction) {  }

    @Override
    public boolean isServiceUp() {
        return false;
    }

    @Override
    public void save(User user) {
        System.out.println("UserServiceImpl");
    }
}
```
```groovy
def "class로 Spy를 만들어야 된다."() {
        given:
        Transaction transaction = Stub(Transaction)
        UserService service = Spy(UserServiceImpl, constructorArgs: [transaction])
        
        expect:
        service.save(new User(name: 'Norman'))
    }
```
cf) 참고자료에서는 이렇게 하면 Spy객체가 만들어진다고 했는데 나는 에러가 발생함. cglib 의존성 추가해주니 Spy 객체 생성됌.
```xml
org.spockframework.mock.CannotCreateMockException: Cannot create mock for class spock.basic.UserServiceImpl.
Mocking of non-interface types requires the CGLIB library. Please put cglib-nodep-2.2 or higher on the class path.

<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>2.2</version>
</dependency>
```

### where:
```groovy
def "다양한 제곱 테스트"() {
        expect:
        Math.pow(base, 2) == expectedResult
        
        where:
        base || expectedResult
        2    || 4
        3    || 9
        10   || 100
    }
```
## 블로그에서 groovy를 이용한 통합테스트 방식 소개
참고 : http://groovy-coder.com/?p=111

올랑(Hollandaise) 소스를 만들기 위해서는 cooking temperature를 매우 정밀하게 조절해야 한다. 

![](/assets/올랑.jpg)

그래서 올랑(Hollandaise) 소스를 위해 애플리케이션에서 temperature monitoring 하는 system을 만든다고 해보자.
HollandaiseTemperatureMonitor 클래스(production code)는 다음과 같다.
```java
@Service
public class HollandaiseTemperatureMonitor {

    /** Maximum hollandaise cooking temperature in degree celsius */
    private static final int HOLLANDAISE_MAX_TEMPERATURE_THRESHOLD = 80;

    /** Minimum hollandaise cooking temperature in degree celsius */
    private static final int HOLLANDAISE_MIN_TEMPERATURE_THRESHOLD = 45;

    private final Thermometer thermometer;

    @Autowired
    public HollandaiseTemperatureMonitor(Thermometer thermometer) {
        this.thermometer = thermometer;
    }

    public boolean isTemperatureOk() {
        int temperature = thermometer.currentTemperature();

        boolean belowMinimumThreshold = temperature < HOLLANDAISE_MIN_TEMPERATURE_THRESHOLD;
        boolean aboveMaximumThreshold = temperature > HOLLANDAISE_MAX_TEMPERATURE_THRESHOLD;
        boolean outOfLimits = belowMinimumThreshold || aboveMaximumThreshold;

        return !outOfLimits;
    }
}
```
Spock을 이용한 단위 테스트
```groovy
class HollandaiseTemperatureMonitorSpec extends Specification {

    @Unroll 
    def "returns #temperatureOk for temperature #givenTemperature"() {
        given: "a stub thermometer returning given givenTemperature"
        Thermometer thermometer = Stub(Thermometer)
        thermometer.currentTemperature() >> givenTemperature

        and: "a monitor with the stubbed thermometer"
        HollandaiseTemperatureMonitor watchman = new HollandaiseTemperatureMonitor(thermometer)

        expect:
        watchman.isTemperatureOk() == temperatureOk

        where:
        givenTemperature || temperatureOk
        0                || false
        100              || false
        80               || true
        45               || true
        60               || true
        -10              || false
    }

}
```
cf) @Unroll : Indicates that iterations of a data-driven feature should be made visible  as separate features to the outside world(IDEs, reports, etc.)
테스트 구현에 영향을 미치지 않음.

**통합 테스트**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ApplicationSpecWithoutAnnotation extends Specification {

    @Autowired
    WebApplicationContext context

    def "should boot up without errors"() {
        expect: "web application context exists"
        context != null
    }

}
```
@SpringApplicationConfiguration 대신에 @SpringBootTest를 사용한다. 그러나 아직 Spring Boot 1.4는 Spock과 호환되지 않는다. spock-spring 플러그인이 `@SpringBootTest`를 인식하지 못한다. 그래서 `@SpringBootTest` 이전에 `@ContextConfiguration`이나 `@ContextHierarchy`를 추가해줘야 한다.
cf) `@DataJpaTest`,`@WebMvcTest`도 아직 지원안한다. 
```groovy
@ContextConfiguration // not mentioned by docs, but had to include this for Spock to startup the Spring context
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SpringBootSpockTestingApplicationSpecIT extends Specification {

    @Autowired
    WebApplicationContext context

    def "should boot up without errors"() {
        expect: "web application context exists"
        context != null
    }
}
```

## 간단한 Spock 적용 후기
### Name Inverter
참고 : https://www.youtube.com/watch?v=czjWpmy3rkM

spock으로 진행해봤는데 에러가 났을 때 좀 더 친절한 메세지 외에는 장점을 못느꼈습니다(중요한건 리팩토링이지 명세가 아닌거 같습니다).
![](/assets/mockitovsgroovy3.jpg)

![](/assets/mockitovsgroovy4.jpg)

### 미담 프로젝트 단위 테스트
**java+mockito**
```java
@Mock
MessageRepository messageRepository;
@Mock
EmployeeRepository employeeRepository;
@Mock
EpisodeCountRepository episodeCountRepository;

@Before
public void setUp() throws Exception {
    MockitoAnnotations.initMocks(this);
    messageService = new MessageService(messageRepository,
            employeeRepository,
            episodeCountRepository);
    messageFixture();
}
```
```java
@Test(expected = NotStartException.class)
public void 시작안했는데_랜덤메세지를_호출하면_예외가_잘_발생하나_확인(){
    // Given
    messages = Arrays.asList(message1,message2,message3);

    // When
    when(episodeCountRepository.findEpisodeCount()).thenReturn(3L);
    when(messageRepository.findByEpisode(3L)).thenReturn(messages);
    when(episodeCountRepository.findEventStatus()).thenReturn(true);

    MessageService messageServiceSpy = Mockito.spy(messageService);
    when(messageServiceSpy.createRandom(3)).thenReturn(1);
    when(messageServiceSpy.getMessageByRandom()).thenReturn(message1);

    // Then
    messageServiceSpy.getMessageByRandom();
}
```
**groovy+spock**
```groovy
def "시작안했는데 랜덤메세지를 호출하면 예외가 잘 발생하나 확인"(){
        given:
        List<Message> messages = messageFixture()
        def mockMessageRepository = Mock(MessageRepository.class)
        def mockEmployeeRepository = Mock(EmployeeRepository.class)
        def mockEpisodeCountRepository = Mock(EpisodeCountRepository.class)
        def messageService = new MessageService(mockMessageRepository,
                mockEmployeeRepository, mockEpisodeCountRepository)

        when:
        mockMessageRepository.findByEpisode(3L) >> messages
        mockEpisodeCountRepository.findEpisodeCount() >> 3L;
        mockEpisodeCountRepository.findEventStatus() >> true
        messageService.getMessageByRandom()
        
        then:
        thrown(NotStartException.class)
    }
```


---

## Spring Boot 1.4 Test방식 변경부분 소개
참고 : https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4

Spring Framework 4.3부터 생성자를 통한 주입에서 더이상 @Autowired가 필요 없어졌다. 생성자가 하나만 있다는 전제하에 Spring이 autowire target으로 본다.
```java
@Component
public class MyComponent {
    
    private final SomeService service;

    public MyComponent(SomeService service) {
        this.service = service;
    }

} 
```
그래서 MyComponent 테스트가 쉬워진다.
```java
@Test
public void testSomeMethod() {
    SomeService service = mock(SomeService.class);
    MyComponent component = new MyComponent(service);
    // setup mock and class component methods
}
```

**Spring Boot 1.3에서**
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=MyApp.class, loader=SpringApplicationContextLoader.class)
public class MyTest {

    // ...

}
```
`@ContextConfiguration`과 `SpringApplicationContextLoader`를 조합해서 썼다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(MyApp.class)
public class MyTest {

    // ...

}
```
`@SpringApplicationConfiguration`을 쓸 수도 있었다.(이게 더 직관적으로 보여서 이걸 썼다.)

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(MyApp.class)
@IntegrationTest
public class MyTest {

    // ...

}
```
`@IntegrationTest`을 쓰는 방법도 있었다. 또는 `@WebIntegrationTest(@IntegrationTest + @WebAppConfiguration)`
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(MyApp.class)
@WebIntegrationTest
public class MyTest {

    // ...

}
```

마지막으로 random port로 돌릴 수도 있고 `@WebIntegrationTest(randomPort=true)` 추가적인 프로퍼티 설정도 있다. `@IntegrationTest("myprop=myvalue")` or `@TestPropertySource(properties="myprop=myvalue")`

선택지가 너무 많아서 고통스럽다. 

**Spring Boot 1.4에서는**
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class MyTest {

    // ...
    
}
```
`SpringRunner`는 기존의 `SpringJUnit4ClassRunner`의 새로운 이름이다. 그리고
`@SpringBootTest`로 심플해졌다. `webEnvironment`속성은 테스트에서 Mock 서블릿 환경 또는 진짜 HTTP server(RANDOM_PORT or DEFINED_PORT)를 설정할 수 있다.

만약 specific configuration을 load하고 싶으면 `@SpringBootTest`의 `classes`속성을 사용하면 된다. **`classes`속성을 생략하면 inner-classes에서 @Configuration을 제일 먼저 load하려 시도하고, 없다면 @SpringBootApplication class를 찾는다.**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class MyTest {
    
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void test() {
        this.restTemplate.getForEntity(
            "/{username}/vehicle", String.class, "Phil");
    }

}
```
`@SpringBootTest`가 사용되는곳에서는  `TestRestTemplate`이 빈으로 사용가능하다. 


**Mocking and spying**
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SampleTestApplicationWebIntegrationTests {

    @Autowired
    private TestRestTemplate restTemplate;

    @MockBean
    private VehicleDetailsService vehicleDetailsService;

    @Before
    public void setup() {
        given(this.vehicleDetailsService.
            getVehicleDetails("123")
        ).willReturn(
            new VehicleDetails("Honda", "Civic"));
    }

    @Test
    public void test() {
        this.restTemplate.getForEntity("/{username}/vehicle", 
            String.class, "sframework");
    }

}
```
위의 예에서 VehicleDetailsService Mockito mock을 만들고 ApplicationContext에 빈으로 주입시켰다. 그리고 setup 메서드에서 Stubbing behavior를 했다. 최종적으로 mock을 호출할 테스트를 만들었다. Mock들은 테스트마다 자동적으로 리셋되기 때문에 `@DirtiesContext`가 필요없다.
(@DirtiesContext 사용은 applicationContext의 제어를 받지 않고 직접 통제하겠다는 것)

spy도 유사하다. `@SpyBean`을 통해 ApplicationContext에 존재하는 빈을 spy로 감싼다.

**Testing the JPA slice**
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class UserRepositoryTests {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private UserRepository repository;

    @Test
    public void findByUsernameShouldReturnUser() {
        this.entityManager.persist(new User("sboot", "123"));
        User user = this.repository.findByUsername("sboot");
        
        assertThat(user.getUsername()).isEqualTo("sboot");
        assertThat(user.getVin()).isEqualTo("123");
    }
}
```
`@DataJpaTest`는 
1. Configure an in-memory database.
2. Auto-configure Hibernate, Spring Data and the DataSource.
3. Perform an `@EntityScan`
4. Turn on SQL logging

TestEntityManager는 Spring Boot에서 제공한다. standard JPA EntityManager를 대신한다. 
### [용어정리]
**Mock Object**
Mock Object 는 검사하고자 하는 코드와 맞물려 동작하는 객체들을 대신하여 동작하기 위해 만들어진 객체이다. 검사하고자 하는 코드는 Mock Object 의 메서드를 부를 수 있고, 이 때 Mock Object는 미리 정의된 결과 값을 전달한다. MockObject는 자신에게 전달된 인자를 검사할 수 있으며, 이를 테스트 코드로 전달할 수도 있다.

**stub**
Stub 은 테스트 과정에서 일어나는 호출에 대해 지정된 답변을 제공하고, 그 밖의 테스트를 위해 별도로 프로그래밍 되지 않은 질의에 대해서는 대게 아무런 대응을 하지 않는다. 또한 Stub은 email gateway stub 이 '보낸' 메시지를 기억하거나, '보낸' 메일 개수를 저장하는 것과 같이, 호출된 내용에 대한 정보를 기록할 수 있다.
Mock은 Mock 객체가 수신할 것으로 예상되는 호출들을 예측하여 미리 프로그래밍한 객체이다. 

**example**
```java
1. Behavior verify

//Let's import Mockito statically so that the code looks clearer
import static org.mockito.Mockito.*;

//mock creation
List mockedList = mock(List.class);

//using mock object
mockedList.add("one");
mockedList.clear();

//verification
verify(mockedList).add("one");
verify(mockedList).clear();

2. Stubbing

//You can mock concrete classes, not only interfaces
LinkedList mockedList = mock(LinkedList.class);

//stubbing
when(mockedList.get(0)).thenReturn("first");
when(mockedList.get(1)).thenThrow(new RuntimeException());

//following prints "first"
System.out.println(mockedList.get(0));

//following throws runtime exception
System.out.println(mockedList.get(1));

//following prints "null" because get(999) was not stubbed
System.out.println(mockedList.get(999));

//Although it is possible to verify a stubbed invocation, usually it's just redundant
//If your code cares what get(0) returns then something else breaks (often before even verify() gets executed).
//If your code doesn't care what get(0) returns then it should not be stubbed. Not convinced? See here.
verify(mockedList).get(0);
```
