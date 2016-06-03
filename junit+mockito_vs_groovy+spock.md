# JUnit+Mockito vs Groovy+Spock

발표 순서<br>
1. groovy+spock과 JUnit+Mockito의 문법 차이
2. 블로그에서 groovy를 이용한 통합테스트 방식
3. 내가 만든 Java+Mockito 단위 테스트와 groovy-spock으로 만든 단위 테스트 비교분석

cf) Spring Boot 1.4 Test방식 변경부분, mock과 stub차이
###groovy+spock과 JUnit+Mockito의 문법 차이
참고 : http://d2.naver.com/helloworld/568425


###블로그에서 groovy를 이용한 통합테스트 방식
참고 : http://groovy-coder.com/?p=111<br>

올랑(Hollandaise) 소스를 만들기 위해서는 cooking temperature를 매우 정밀하게 조절해야 한다. 
![](올랑.jpg)

그래서 올랑(Hollandaise) 소스를 위해 애플리케이션에서 temperature monitoring 하는 system을 만든다고 해보자.
```
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
다음은 Spring을 쓰지 않고 groovy+spock으로 단위 테스트를 만든 예제다. 흥미로운 점은 `Stub(Thermometer)`를 통해 spock feature Stub을 만들었고 `givenTemperature`를 리턴한다. <br>

production code HollandaiseTemperatureMonitor 클래스는 다음과 같다.
```
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

통합 테스트<br>
```
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
근데 지금 Spring Boot 1.4는 Spock과 호환되지 않는다. spock-spring 플러그인이 `@SpringBootTest`를 인식하지 못한다. 그래서 `@SpringBootTest` 이전에 `@ContextConfiguration`이나 `@ContextHierarchy`를 추가해줘야 한다.<br>
cf) `@DataJpaTest`,`@WebMvcTest`도 아직 지원안한다. 
```
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
Spring Boot 1.4에서는 persistence layer 통합 테스트에 대한 간편한 방식을 소개했다`@DataJpaTest`은 persistence layer(구체적으로는 JPA)와 상호작용이 필요한 component들만 초기화해서 빠른 통합 테스트가 가능하다.
```
@ContextConfiguration
@DataJpaTest
class HistoricTemperatureDataRepositorySpecIT extends Specification {

    @Autowired
    HistoricTemperatureDataRepository historicTemperatureDataRepository

    @Autowired
    TestEntityManager testEntityManager

    def "should load all data"() {
        given: "one temperature entry"
        int temperature = 5
        HistoricTemperatureData data = new HistoricTemperatureData(temperature, new Timestamp(System.currentTimeMillis()))
        testEntityManager.persist(data)

        when: "loading data from repository"
        def loadedData = historicTemperatureDataRepository.findAll()

        then: "persisted data is loaded"
        loadedData.first().temperature == temperature
    }
}
```

###내가 만든 Java+Mockito 단위 테스트와 groovy-spock으로 만든 단위 테스트 비교분석





###Spring Boot 1.4 Test방식 변경부분 소개
참고 : https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4<br>

Spring Framework 4.3부터 생성자를 통한 주입에서 더이상 @Autowired가 필요 없어졌다. 생성자가 하나만 있다는 전제하에 Spring이 autowire target으로 본다.
```
@Component
public class MyComponent {
    
    private final SomeService service;

    public MyComponent(SomeService service) {
        this.service = service;
    }

} 
```
그래서 MyComponent 테스트가 쉬워진다.
```
@Test
public void testSomeMethod() {
    SomeService service = mock(SomeService.class);
    MyComponent component = new MyComponent(service);
    // setup mock and class component methods
}
```

**Spring Boot 1.3에서**<br>
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=MyApp.class, loader=SpringApplicationContextLoader.class)
public class MyTest {

    // ...

}
```
`@ContextConfiguration`과 `SpringApplicationContextLoader`를 조합해서 썼다.

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(MyApp.class)
public class MyTest {

    // ...

}
```
`@SpringApplicationConfiguration`을 쓸 수도 있었다.(이게 더 직관적으로 보여서 이걸 썼다.)

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(MyApp.class)
@IntegrationTest
public class MyTest {

    // ...

}
```
`@IntegrationTest`을 쓰는 방법도 있었다. 또는 `@WebIntegrationTest(@IntegrationTest + @WebAppConfiguration)`
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(MyApp.class)
@WebIntegrationTest
public class MyTest {

    // ...

}
```

마지막으로 random port로 돌릴 수도 있고 `@WebIntegrationTest(randomPort=true)` 추가적인 프로퍼티 설정도 있다. `@IntegrationTest("myprop=myvalue")` or `@TestPropertySource(properties="myprop=myvalue")`

선택지가 너무 많아서 고통스럽다. 

**Spring Boot 1.4에서는**<br>
```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class MyTest {

    // ...
    
}
```
`SpringRunner`는 기존의 `SpringJUnit4ClassRunner`의 새로운 이름이다. 그리고
`@SpringBootTest`로 심플해졌다. `webEnvironment`속성은 테스트에서 Mock 서블릿 환경 또는 진짜 HTTP server(RANDOM_PORT or DEFINED_PORT)를 설정할 수 있다.

만약 specific configuration을 load하고 싶으면 `@SpringBootTest`의 `classes`속성을 사용하면 된다. **`classes`속성을 생략하면 inner-classes에서 @Configuration을 제일 먼저 load하려 시도하고, 없다면 @SpringBootApplication class를 찾는다.**

```
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


**Mocking and spying**<br>
```
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

###[용어정리]
**Mock Object**<br>
Mock Object 는 검사하고자 하는 코드와 맞물려 동작하는 객체들을 대신하여 동작하기 위해 만들어진 객체이다. 검사하고자 하는 코드는 Mock Object 의 메서드를 부를 수 있고, 이 때 Mock Object는 미리 정의된 결과 값을 전달한다. MockObject는 자신에게 전달된 인자를 검사할 수 있으며, 이를 테스트 코드로 전달할 수도 있다.

**stub**<br>
Stub 은 테스트 과정에서 일어나는 호출에 대해 지정된 답변을 제공하고, 그 밖의 테스트를 위해 별도로 프로그래밍 되지 않은 질의에 대해서는 대게 아무런 대응을 하지 않는다. 또한 Stub은 email gateway stub 이 '보낸' 메시지를 기억하거나, '보낸' 메일 개수를 저장하는 것과 같이, 호출된 내용에 대한 정보를 기록할 수 있다. <br>
Mock은 Mock 객체가 수신할 것으로 예상되는 호출들을 예측하여 미리 프로그래밍한 객체이다. 

**example**<br>
```
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
