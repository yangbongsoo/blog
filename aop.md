스프링에 적용된 가장 인기 있는 AOP 적용 대상은 바로 선언적 트랜잭션 기능이다. 
##1. 데코레이터 패턴을 이용한 트랜잭션 코드 분리
![](스크린샷 2016-09-14 오후 4.09.11.jpg)
UserService 인터페이스를 도입해서 순수 비지니스로직을 담당하는 UserServiceImpl과 트랜잭션 처리를 담당하는 UserServiceTx로 나눈다.
```
public interface UserService {
  void add(User user);
  void upgradeLevels();
}
```
UserServiceTx에서는 트랜잭션 경계설정을 통해 트랜잭션 작업을 수행하고 실질적인 비지니스로직은 주입받은 userServiceImpl에게 위임하는 구조다.
```
public class UserServiceTx implements UserService {
  UserService userService;
  PlatformTransactionManager transactionManager;
  
  public void setTransactionManager(PlatformTransactionManager transactionManager) {
    this.transactionManager = transactionManager;
  }
  
  public void setUserService(UserService userService) {
    this.userService = userService;
  }
  
  public void add(User user) {
    this.userService.add(user);
  }
  
  public void upgradeLevels() {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
      userService.upgradeLevels();
      this.transactionManager.commit(status);
    } catch(RuntimeException e) {
      this.transactionManager.rollback(status);
      throw e;
    }
  }
}
```
```
public class UserServiceImpl implements UserService {
  UserDao userDao;
  MailSender mailSender;
  
  public void upgradeLevels() {
    List<User> users = userDao.getAll();
    for (User user : users) {
      if (canUpgradeLevel(user)) {
        upgradeLevel(user);
      }
    }
  }
  
  ...
}
```
그리고 아래와 같이 클라이언트가 UserServiceTx 빈을 호출해서 사용하도록 만든다. 따라서 userService라는 대표적인 빈 아이디는 UserServiceTx 클래스로 정의된 빈에게 부여해준다.  
```
<bean id="userService" class="springbook.user.service.UserServiceTx">
  <property name="transactionManager" ref="transactionManager" />
  <property name="userService" ref="userServiceImpl" />
</bean>

<bean id="userServiceImpl" class="springbook.user.service.UserServiceImpl">
  <property name="userDao" ref="userDao" />
  <property name="mailSender" ref="mailSender" />
</bean>

...
```
**cf) 프록시 패턴 vs 데코레이터 패턴**<br>
먼저 프록시란 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 말한다. 그리고 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타깃이라고 부른다.<br>

프록시의 특징은 타깃과 같은 인터페이스를 구현했다는 것과 프록시가 타깃을 제어할 수 있는 위치에 있다는 것이다. 프록시는 사용 목적에 따라 두 가지로 구분할 수 있다. 첫째는 클라이언트가 타깃에 접근하는 방법을 제어(프록시 패턴)하기 위해서다. 두 번째는 타깃에 부가적인 기능을 부여(데코레이터 패턴)해주기 위해서다.<br>

이 책에서는 타깃과 동일한 인터페이스를 구현하고 클라이언트와 타깃 사이에 존재하면서 기능의 부가 또는 접근 제어를 담당하는 오브젝트를 모두 프록시라 부른다.<br>

##2. 다이내믹 프록시를 이용한 트랜잭션 부가기능
프록시를 이용하는 방법은 유용하지만 매번 새로운 클래스를 정의해야 하고, 인터페이스의 구현해야 할 메서드가 많으면 모든 메서드를 일일히 구현해서 위임하는 코드를 넣어야하는 번거로움이 있다. 하지만 자바의 리플렉션 기능을 이용하면 손쉽게 구현할 수 있다.<br>
![](스크린샷 2016-09-14 오후 4.49.29.jpg)

다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트다. 다이내믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로 만들어진다. 클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용할 수 있다. 이 덕분에 프록시를 만들 때 인터페이스를 모두 구현해가면서 클래스를 정의하는 수고를 덜 수 있다. 프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어주기 때문이다.<br>

다이내믹 프록시가 인터페이스 구현 클래스의 오브젝트는 만들어주지만, 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야 한다. 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다. InvocationHandler 인터페이스는 다음과 같은 메서드 한 개만 가진 간단한 인터페이스다.
```
public Object invoke(Object proxy, Method method, Object[] args)
```
invoke() 메서드는 리플렉션의 Method 인터페이스를 파라미터로 받는다. 메서드를 호출할 때 전달되는 파라미터도 args로 받는다. 다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke() 메서드로 넘기는 것이다. 타깃 인터페이스의 모든 메서드 요청이 하나의 메서드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.<br>
![](스크린샷 2016-09-14 오후 4.57.53.jpg)
간단한 예를 통해 다시 살펴보자. 
```
interface Hello {
  String sayHello(String name);
  String sayHi(String name);
  String sayThankYou(String name);
}
```
위와 같은 Hello 인터페이스가 있고 이를 구현한 타깃 클래스는 다음과 같다.
```
public class HelloTarget implements Hello {
  public String sayHello(String name) {
    return "Hello" + name;
  }
  
  public String sayHi(String name) {
    return "Hi" + name;
  }
  
  public String sayThankYou(String name) {
    return "Thank You" + name;
  }
}
```
이제 다이내믹 프록시를 위해 InvocationHandler 인터페이스를 구현한 부가기능 코드를 만들어보자(부가기능은 모든 글자를 대문자로 만드는 것이다).
```
public class UppercaseHandler implements InvocationHandler {
  Hello target;
  
  public UppercaseHandler(Hello target) {
    this.target = target;
  }
  
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String ret = (String)method.invoke(target, args);
    return ret.toUpperCase();
  }
}
```
Hello 인터페이스의 모든 메서드는 결과가 String 타입이므로 메서드 호출의 결과를 String 타입으로 변환해도 안전하다. 타깃 오브젝트의 메서드 호출이 끝났으면 프록시가 제공하려는 부가기능인 리턴 값을 대문자로 바꾸는 작업을 수행하고 결과를 리턴한다. 리턴된 값은 다이내믹 프록시가 받아서 최종적으로 클라이언트에게 전달될 것이다.<br>

이제 이 InvocationHandler를 사용하고 Hello 인터페이스를 구현하는 프록시를 만들어보자. 다이내믹 프록시의 생성은 Proxy 클래스의 newProxyInstance() 정적 팩토리 메서드를 이용하면 된다.
```
Hello proxiedHello = (Hello)Proxy.newProxyInstance(
  // 동적으로 생성되는 다이내믹 프록시 클래스의 로딩에 사용할 클래스 로더
  getClass().getClassLoader(), 
  // 구현할 인터페이스
  new Class[] { Hello.class },
  // 부가기능과 위임 코드를 담은 InvocationHandler
  new UppercaseHandler(new HelloTarget())
);
```
사용 방법을 자세히 살펴보자. 첫 번째 파라미터는 클래스 로더를 제공해야 한다. 다이내믹 프록시가 정의되는 클래스 로더를 지정하는 것이다. 두 번째 파라미터는 다이내믹 프록시가 구현해야 할 인터페이스다. 다이내믹 프록시는 한 번에 하나 이상의 인터페이스를 구현할 수도 있다. 따라서 인터페이스의 배열을 사용한다. 마지막 파라미터로는 부가기능과 위임 관련 코드를 담고 있는 InvocationHandler 구현 오브젝트를 제공해야 한다. Hello 타입의 타깃 오브젝트를 생성자로 받고, 모든 메서드 호출의 리턴 값을 대문자로 바꿔주는 UppercaseHandler 오브젝트를 전달했다.<br>

이제 이 방법을 이용해서 트랜잭션에 적용해보자.
```
public class TransactionHandler implements InvocationHandler {
  private Object target;
  private PlatformTransactionManager transactionManager;
  private String pattern;
  
  public void setTarget(Object target) {
    this.target = target;
  }
  
  public void setTransactionManager(PlatformTransactionManager transactionManager) {
    this.transactionManager = transactionManager;
  }
  
  public void setPattern(String pattern) {
    this.pattern = pattern;
  }
  
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (method.getName().startsWith(pattern)) {
      return invokeInTransaction(method, args);
    } else {
      return method.invoke(target, args);
    }
  }
  
  private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTranscationDefinition());
    try {
      Object ret = method.invoke(target, args);
      this.transactionManager.commit(status);
      return ret;
    } catch (InvocationTargetException e) {
      this.transactionManager.rollback(status);
      throw e.getTargetException();
    }
  }
}
```
```
@Test
public void upgradeAllOrNothing() throws Exception {
  ...
  TransactionHandler txHandler = new TransactionHandler();
  txHandler.setTarget(testUserService);
  txHandler.setTransactionManager(transactionManager);
  txHandler.setPattern("upgradeLevels");
  
  UserService txUserService = (UserService)Proxy.newProxyInstance(
    getClass().getClassLoader(),
    new Class[] { UserService.class },
    txHandler
  );
  ...
}
```
위와 같이 TransactionHandler 오브젝트를 이용해 UserService 타입의 다이내믹 프록시를 생성하면 트랜잭션 프록시가 적용된다. 
**그런데 문제는 DI의 대상이 되는 다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로는 등록할 방법이 없다.** 스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다. 스프링은 지정된 클래스 이름을 가지고 리플렉션을 이용해서 해당 클래스의 오브젝트를 만든다. 문제는 다이내믹 프록시 오브젝트는 이런 식으로 프록시 오브젝트가 생성되지 않는다는 점이다. 사실 다이내믹 프록시 오브젝트의 클래스가 어떤 것인지 알 수도 없다. 클래스 자체도 내부적으로 다이내믹하게 새로 정의해서 사용하기 때문이다. 따라서 사전에 프록시 오브젝트의 클래스 정보를 미리 알아내서 스프링의 빈에 정의할 방법이 없다. 다이내믹 프록시는 Proxy 클래스의 newProxyInstance() 라는 정적 팩토리 메서드를 통해서만 만들 수 있다.<br>

사실 스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에도 빈을 만들 수 있는 여러 가지 방법을 제공한다. 대표적으로 팩토리 빈을 이용한 빈 생성 방법을 들 수 있다. 팩토리 빈이란 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈을 말한다.
```
package org.springframework.beans.factory;

public interface FactoryBean<T> {
  T getObject() throws Exception;
  Class<? extends T> getObjectType();
  boolean isSingleton();
}
```
FactoryBean 인터페이스를 구현한 클래스를 스프링 빈으로 만들어두면 getObject()라는 메서드가 생성해주는 오브젝트가 실제 빈의 오브젝트로 대치된다. 따라서 팩토리 빈의 getObject() 메서드에 다이내믹 프록시 오브젝트를 만들어주는 코드를 넣으면 된다. **하지만 이 방식은 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 일이 불가능하다.** 하나의 타깃 오브젝트에만 부여되는 부가기능이라면 상관없겠지만, 트랜잭션과 같이 비지니스 로직을 담은 많은 클래스의 메서드에 적용할 필요가 있다면 거의 비슷한 프록시 팩토리 빈의 설정이 중복되는 것을 막을 수 없다. 그래서 이 방식에 대해서는 더 이상 살펴보지 않고 스프링의 프록시 팩토리 빈에 대해서 자세히 알아보자.
##3. 스프링의 프록시 팩토리 빈
스프링은 트랜잭션 기술과 메일 발송 기술에 적용했던 서비스 추상화를 프록시 기술에도 동일하게 적용하고 있다. 자바에는 JDK에서 제공하는 다이내믹 프록시 외에도 편리하게 프록시를 만들 수 있도록 지원해주는 다양한 기술이 존재한다. 따라서 스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다. 생성된 프록시는 스프링의 빈으로 등록돼야 한다. **스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공해준다.** 스프링의 ProxyFactoryBean은 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈이다. ProxyFactoryBean은 순수하게 프록시를 생성하는 작업만을 담당하고, 프록시를 통해 제공해줄 부가기능은 별도의 빈에 둘 수 있다.<br>

ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다. MethodInterceptor는 InvocationHandler와 비슷하지만 한 가지 다른 점이 있다. InvocationHandler의 invoke() 메서드는 타깃 오브젝트에 대한 정보를 제공하지 않는다. 따라서 타깃은 InvocationHandler를 구현한 클래스가 직접 알고 있어야 한다. 반면에 MethodInterceptor의 invoke() 메서드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다. 그 차이 덕분에 MethodInterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다. 따라서 MethodInterceptor 오브젝트는 타깃이 다른 여러 프록시에서 함께 사용할 수 있고, 싱글톤 빈으로 등록 가능하다.
```
@Test
public void proxyFactoryBean() {
  ProxyFactoryBean pfBean = new ProxyFactoryBean();
  // 타깃 설정
  pfBean.setTarget(new HelloTarget());
  // 부가기능을 담은 어드바이스를 추가한다. 여러 개를 추가할 수도 있다.
  pfBean.addAdvice(new UppercaseAdvice());
  // FactoryBean 이므로 getObject()로 생성된 프록시를 가져온다.
  Hello proxiedHello = (Hello) pfBean.getObject();
  assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
}

static class UppercaseAdvice implements MethodInterceptor {
  public Object invoke(MethodInvocation invocation) throws Throwable {
    // 리플렉션의 Method와 달리 메서드 실행 시 타깃 오브젝트를 전달할 필요가 없다.
    // MethodInvocation은 메서드 정보와 함께 타깃 오브젝트를 알고 있기 때문이다.
    String ret = (String)invocation.proceed();
    return ret.toUpperCase();
  }
}
```
**어드바이스 : 타깃이 필요 없는 순수한 부가기능을 담은 오브젝트**<br>
InvocationHandler를 구현했을 때와 달리 MethodInterceptor를 구현한 UppercaseAdvice에는 타깃 오브젝트가 등장하지 않는다. MethodInterceptor로는 메서드 정보와 함께 타깃 오브젝트가 담긴 MethodInvocation 오브젝트가 전달된다. MethodInvocation은 일종의 콜백 오브젝트로, proceed() 메서드를 실행하면 타깃 오브젝트의 메서드를 내부적으로 실행해주는 기능이 있다(MethodInvocation 구현 클래스는 일종의 공유 가능한 템플릿처럼 동작).<br>

바로 이 점이 JDK의 다이내믹 프록시를 직접 사용하는 코드와 스프링이 제공해주는 프록시 추상화 기능인 ProxyFactoryBean을 사용하는 코드의 가장 큰 차이점이자 ProxyFactoryBean의 장점이다. ProxyFactoryBean은 작은 단위의 템플릿/콜백 구조를 응용해서 적용했기 때문에 템플릿 역할을 하는 MethodInvocation을 싱글톤으로 두고 공유할 수 있다. 마치 SQL 파라미터 정보에 종속되지 않는 JdbcTemplate이기 때문에 수많은 DAO 메서드가 하나의 JdbcTemplate 오브젝트를 공유할 수 있는 것과 마찬가지다.<br>

**포인트컷 : 부가기능 적용 대상 메서드 선정 방법**<br>
![](스크린샷 2016-09-18 오후 2.08.27.jpg)
스프링은 부가기능을 제공하는 오브젝트를 어드바이스라고 부르고, 메서드 선정 알고리즘을 담은 오브젝트를 포인트컷이라고 부른다. 어드바이스와 포인트컷은 모두 프록시에 DI로 주입돼서 사용된다. 두 가지 모두 여러 프록시에서 공유가 가능하도록 만들어지기 때문에 스프링의 싱글톤 빈으로 등록이 가능하다.<br>

프록시는 클라이언트로부터 요청을 받으면 먼저 포인트컷에게 부가기능을 부여할 메서드인지를 확인해달라고 요청한다. 포인트컷은 Pointcut 인터페이스를 구현해서 만들면 된다. 프록시는 포인트컷으로부터 부가기능을 적용할 대상 메서드인지 확인받으면, MethodInterceptor 타입의 어드바이스를 호출한다.
```
@Test
public void pointcutAdvisor() {
  ProxyFactoryBean pfBean = new ProxyFactoryBean();
  pfBean.setTarget(new HelloTarget());
  
  NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
  pointcut.setMappedName("sayH*");
  
  //포인트컷과 어드바이스를 Advisor로 묶어서 한번에 추가
  pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercaseAdvice()));
  Hello proxiedHello = (Hello) pfBean.getObject();
}
```
포인트컷이 필요 없을 때는 ProxyFactoryBean의 addAdvice() 메서드를 호출해서 어드바이스만 등록하면 됐다. 그런데 포인트컷을 함께 등록할 때는 어드바이스와 포인트컷을 Advisor 타입으로 묶어서 addAdvisor() 메서드를 호출해야 한다. 왜 굳이 별개의 오브젝트로 묶어서 등록해야 할까? 그 이유는 ProxyFactoryBean에는 여러 개의 어드바이스와 포인트컷이 추가될 수 있기 때문이다. 포인트컷과 어드바이스를 따로 등록하면 어떤 어드바이스(부가기능)에 대해 어떤 포인트컷(메서드 선정)을 적용할지 애매해지기 때문이다. 그래서 이 둘을 Advisor 타입의 오브젝트에 담아서 조합을 만들어 등록하는 것이다.<br>
`어드바이저 = 포인트컷(메서드 선정 알고리즘) + 어드바이스(부가기능)`<br>

**ProxyFactoryBean을 이용해 트랜잭션 기능 적용**<br>
```
public class TransactionAdvice implements MethodInterceptor {
  PlatformTransactionManager transactionManager;
  
  public void setTransactionManager(PlatformTransactionManager transactionManager) {
    this.transactionManager = transactionManager;
  }
  
  public Object invoke(MethodInvocation invocation) throws Throwable {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
      Object ret = invocation.proceed();
      this.transactionManager.commit(status);
      return ret;
    } catch (RuntimeException e) {
      this.transactionManager.rollback(status);
      throw e;
    }
  }
}
```
트랜잭션 어드바이스 빈 설정
```
<bean id="transactionAdvice" class="xxx.xxx.service.TransactionAdvice">
  <property name="transactionManager" ref="transactionManager" />
</bean>
```
포인트컷 빈 설정
```
<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
  <property name="mappedName" ref="upgrade*" />
</bean>
```
어드바이저 빈 설정
```
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
  <property name="advice" ref="transactionAdvice" />
  <property name="pointcut" ref="transactionPointcut" />
</bean>
```
이제 ProxyFactoryBean을 등록할 차례다. 아래와 같이 프로퍼티에 타깃 빈과 어드바이저 빈을 지정해주면 된다.
```
<bean id="userService class="org.springframework.aop.framework.ProxyFactoryBean">
  <property name="target" ref="userServiceImpl" />
  <property name="interceptorNames">
    <list>
      <value>transactionAdvisor</value>
    </list>
  </property>
</bean>
```
어드바이저는 interceptorNames라는 프로퍼티를 통해 넣는다. 프로퍼티 이름이 advisor가 아닌 이유는 어드바이스와 어드바이저를 혼합해서 설정할 수 있도록 하기 위해서다. 그래서 property 태그의 ref 애트리뷰트를 통한 설정 대신 list와 value 태크를 통해 여러 개의 값을 넣을 수 있도록 하고 있다. value 태그에는 어드바이스 또는 어드바이저로 설정한 빈의 아이디를 넣으면 된다. 한 개 이상을 넣을 수 있다.<br>

**어드바이스와 포인트컷의 재사용**<br>
![](스크린샷 2016-09-18 오후 3.01.20.jpg)
위의 그림은 ProxyFactoryBean을 이용해서 많은 수의 서비스 빈에게 트랜잭션 부가기능을 적용했을 때의 구조다. 트랜잭션 부가기능을 담은 TransactionAdvice는 하나만 만들어서 싱글톤 빈으로 등록해주면, DI 설정을 통해 모든 서비스에 적용이 가능하다. 메서드 선정 방식이 달라지는 경우만 포인트컷의 설정을 따로 등록하고 어드바이저로 조합해서 적용해주면 된다.
##4. 스프링 AOP
아직 한 가지 해결할 과제가 남아 있다. 부가기능의 적용이 필요한 타깃 오브젝트마다 거의 비슷한 내용의 ProxyFactoryBean 빈 설정정보를 추가해주는 부분이다. 새로운 타깃이 등장했다고 해서 코드를 손댈 필요는 없어졌지만, 설정은 매번 복사하고 붙이고 target 프로퍼티의 내용을 수정해줘야 한다. 이런 류의 중복은 어떻게 제거할까?<br>


##5. 트랜잭션 속성




