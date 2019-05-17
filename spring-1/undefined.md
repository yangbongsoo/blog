# 테스트

**웹을 통한 DAO 테스트 방법의 문제점** 보통 웹 프로그램에서 사용하는 DAO 테스트 방법은 다음과 같다. 서비스 계층, MVC 프레젠테이션 계층까지 포함한 모든 입출력 기능을 대충이라도 코드로 다 만든다. 이렇게 웹 화면을 통해 값을 입력하고, 기능을 수행하고, 결과를 확인하는 방법은 가장 흔히 쓰이는 방법이지만 DAO에 대한 테스트로서는 단점이 너무 많다. DAO뿐만 아니라 서비스 클래스. 컨트롤러, 뷰 등 모든 레이어의 기능을 다 만들고 나서야 테스트가 가능하다는 점이 가장 큰 문제점이다.

JUnit 프레임워크가 요구하는 조건 두가지, 첫째는 메서드가 public으로 선언되어야 하고 두번째는 @Test 애노테이션 붙여줘야한다.\(하나의 클래스에 안에 여러 개의 테스트 메서드가 들어가는 것도 허용하는데 리턴 값이 void형이고 파라미터가 없다는 조건을 지키면 된다.\)

테스트의 결과를 검증하는 if/else대신 JUnit이 제공하는 assertThat을 사용한다.

cf\) 인텔리제이 IDE에서 assertThat import 자동으로 안해준다.

```java
import static org.hamcrest.CoreMatchers.is; 
import static org.junit.Assert.assertThat; 

assertThat(user2.getName(), is(user.getName()));
```

JUnit은 특정한 테스트 메서드의 실행 순서를 보장해주지 않는다. 데스트의 결과가 테스트 실행 순서에 영향을 받는다면 테스트를 잘못 만든 것이다.

**모든 테스트는 실행 순서와 상관없이 독립적으로 항상 동일한 결과를 낼 수 있도록 해야 한다.**

예외가 반드시 발생해야 하는 경우를 테스트하고 싶을 때는 @Test 애노테이션의 expected 엘리먼트를 사용하면 된다. expected는 테스트 메서드 실행 중에 발생하리라 기대하는 예외 클래스를 넣어주면 된다.

```java
@Test(expected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLException{
    ...

    dao.get("unknown_id");
}
```

`@Before @After`테스트 코드들의 공통적인 작업을 처리하는 애노테이션

픽스처\(fixture\) : 테스트를 수행하는 데 필요한 정보나 오브젝트

```java
public class UserDaoTest{
    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;

    @Before
    public void setUp(){
        ...
        this.user1 = new User("gyumee", "박성철", "springno1");
        this.user2 = new User("leegw700", "이길원", "springno2");
        this.user3 = new User("bumjin", "박범진", "springno3");
    }
    ...
}
```

JUnit은 테스트 클래스 전체에 걸쳐 딱 한 번만 실행되는 @BeforeClass 스태틱 메서드를 지원한다. 하지만 스프링은 JUnit을 이용하는 테스트 컨텍스트 프레임워크를 제공한다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest{
    @Autowired
    private ApplicationContext context;

    ...

    @Before
    public void setUp(){
        System.out.println(context);
        System.out.println(this); 
    }

    ... //테스트 3개 
}
```

SpringJUnit4ClassRunner라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스를 지정해주면 JUnit이 테스트를 진행하는 중에 테스트가 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업을 진행해준다.

위의 코드를 실행 시켜보면 테스트 3개마다 각각 setUp 메서드가 실행되고, context값은 항상 동일 한것을 확인할 수 있다. 하지만 this, UserDaoTest 오브젝트는 테스트 마다 새롭게 생성된다. JUnit은 테스트 메서드를 실행할 때마다 새로운 테스트 오브젝트를 만들기 때문이다.

정말 실행할때마다 새로운 테스트 오브젝트를 만드는지 테스트해보자

```java
public class JUnitTest{
    static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();

    @Test
    public void test1(){
        assertThat(testObjects, not(hasItem(this)));
        testObject.add(this);
    }

    @Test
    public void test2(){
        assertThat(testObjects, not(hasItem(this)));
        testObject.add(this);
    }

    @Test
    public void test3(){
        assertThat(testObjects, not(hasItem(this)));
        testObject.add(this);
    }
}
```

