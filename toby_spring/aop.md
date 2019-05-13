스프링에 적용된 가장 인기 있는 AOP 적용 대상은 바로 선언적 트랜잭션 기능이다. 
## 1. 데코레이터 패턴을 이용한 트랜잭션 코드 분리

![](/assets/tobyaop1.jpg)

UserService 인터페이스를 도입해서 순수 비지니스로직을 담당하는 UserServiceImpl과 트랜잭션 처리를 담당하는 UserServiceTx로 나눈다.
```java
public interface UserService {
  void add(User user);
  void upgradeLevels();
}
```
UserServiceTx에서는 트랜잭션 경계설정을 통해 트랜잭션 작업을 수행하고 실질적인 비지니스로직은 주입받은 userServiceImpl에게 위임하는 구조다.
```java
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
```java
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
```xml
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
**cf) 프록시 패턴 vs 데코레이터 패턴**

먼저 프록시란 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 말한다. 그리고 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타깃이라고 부른다.

프록시의 특징은 타깃과 같은 인터페이스를 구현했다는 것과 프록시가 타깃을 제어할 수 있는 위치에 있다는 것이다. 프록시는 사용 목적에 따라 두 가지로 구분할 수 있다. 첫째는 클라이언트가 타깃에 접근하는 방법을 제어(프록시 패턴)하기 위해서다. 두 번째는 타깃에 부가적인 기능을 부여(데코레이터 패턴)해주기 위해서다.

이 책에서는 타깃과 동일한 인터페이스를 구현하고 클라이언트와 타깃 사이에 존재하면서 기능의 부가 또는 접근 제어를 담당하는 오브젝트를 모두 프록시라 부른다.

## 2. 다이내믹 프록시를 이용한 트랜잭션 부가기능
프록시를 이용하는 방법은 유용하지만 매번 새로운 클래스를 정의해야 하고, 만약 인터페이스의 구현해야 할 메서드가 많으면 모든 메서드를 일일히 구현해서 위임하는 코드를 넣어야하는 번거로움이 있다. 하지만 자바의 리플렉션 기능을 이용하면 손쉽게 구현할 수 있다.

![](/assets/tobyaop2.jpg)

다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트다. 다이내믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로 만들어진다. 클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용할 수 있다. 이 덕분에 프록시를 만들 때 인터페이스를 모두 구현해가면서 클래스를 정의하는 수고를 덜 수 있다. 프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어주기 때문이다.

다이내믹 프록시가 인터페이스 구현 클래스의 오브젝트는 만들어주지만, 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야 한다. 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다. InvocationHandler 인터페이스는 다음과 같은 메서드 한 개만 가진 간단한 인터페이스다.
```java
public Object invoke(Object proxy, Method method, Object[] args)
```
invoke() 메서드는 리플렉션의 Method 인터페이스를 파라미터로 받는다. 메서드를 호출할 때 전달되는 파라미터도 args로 받는다. 다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke() 메서드로 넘기는 것이다. 타깃 인터페이스의 모든 메서드 요청이 하나의 메서드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.

![](/assets/tobyaop3.jpg)

간단한 예를 통해 다시 살펴보자. 
```java
interface Hello {
  String sayHello(String name);
  String sayHi(String name);
  String sayThankYou(String name);
}
```
위와 같은 Hello 인터페이스가 있고 이를 구현한 타깃 클래스는 다음과 같다.
```java
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
```java
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
Hello 인터페이스의 모든 메서드는 결과가 String 타입이므로 메서드 호출의 결과를 String 타입으로 변환해도 안전하다. 타깃 오브젝트의 메서드 호출이 끝났으면 프록시가 제공하려는 부가기능인 리턴 값을 대문자로 바꾸는 작업을 수행하고 결과를 리턴한다. 리턴된 값은 다이내믹 프록시가 받아서 최종적으로 클라이언트에게 전달될 것이다.

이제 이 InvocationHandler를 사용하고 Hello 인터페이스를 구현하는 프록시를 만들어보자. 다이내믹 프록시의 생성은 Proxy 클래스의 newProxyInstance() 정적 팩토리 메서드를 이용하면 된다.
```java
Hello proxiedHello = (Hello)Proxy.newProxyInstance(
  // 동적으로 생성되는 다이내믹 프록시 클래스의 로딩에 사용할 클래스 로더
  getClass().getClassLoader(), 
  // 구현할 인터페이스
  new Class[] { Hello.class },
  // 부가기능과 위임 코드를 담은 InvocationHandler
  new UppercaseHandler(new HelloTarget())
);
```
사용 방법을 자세히 살펴보자. 첫 번째 파라미터는 클래스 로더를 제공해야 한다. 다이내믹 프록시가 정의되는 클래스 로더를 지정하는 것이다. 두 번째 파라미터는 다이내믹 프록시가 구현해야 할 인터페이스다. 다이내믹 프록시는 한 번에 하나 이상의 인터페이스를 구현할 수도 있다. 따라서 인터페이스의 배열을 사용한다. 마지막 파라미터로는 부가기능과 위임 관련 코드를 담고 있는 InvocationHandler 구현 오브젝트를 제공해야 한다. Hello 타입의 타깃 오브젝트를 생성자로 받고, 모든 메서드 호출의 리턴 값을 대문자로 바꿔주는 UppercaseHandler 오브젝트를 전달했다.

이제 이 방법을 이용해서 트랜잭션에 적용해보자.
```java
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
```java
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

**그런데 문제는 DI의 대상이 되는 다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로는 등록할 방법이 없다.** 스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다. 스프링은 지정된 클래스 이름을 가지고 리플렉션을 이용해서 해당 클래스의 오브젝트를 만든다. 문제는 다이내믹 프록시 오브젝트는 이런 식으로 프록시 오브젝트가 생성되지 않는다는 점이다. 사실 다이내믹 프록시 오브젝트의 클래스가 어떤 것인지 알 수도 없다. 클래스 자체도 내부적으로 다이내믹하게 새로 정의해서 사용하기 때문이다. 따라서 사전에 프록시 오브젝트의 클래스 정보를 미리 알아내서 스프링의 빈에 정의할 방법이 없다. 다이내믹 프록시는 Proxy 클래스의 newProxyInstance() 라는 정적 팩토리 메서드를 통해서만 만들 수 있다.

사실 스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에도 빈을 만들 수 있는 여러 가지 방법을 제공한다. 대표적으로 팩토리 빈을 이용한 빈 생성 방법을 들 수 있다. 팩토리 빈이란 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈을 말한다.
```java
package org.springframework.beans.factory;

public interface FactoryBean<T> {
  T getObject() throws Exception;
  Class<? extends T> getObjectType();
  boolean isSingleton();
}
```
FactoryBean 인터페이스를 구현한 클래스를 스프링 빈으로 만들어두면 getObject()라는 메서드가 생성해주는 오브젝트가 실제 빈의 오브젝트로 대치된다. 따라서 팩토리 빈의 getObject() 메서드에 다이내믹 프록시 오브젝트를 만들어주는 코드를 넣으면 된다. **하지만 이 방식은 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 일이 불가능하다.** 하나의 타깃 오브젝트에만 부여되는 부가기능이라면 상관없겠지만, 트랜잭션과 같이 비지니스 로직을 담은 많은 클래스의 메서드에 적용할 필요가 있다면 거의 비슷한 프록시 팩토리 빈의 설정이 중복되는 것을 막을 수 없다. 그래서 이 방식에 대해서는 더 이상 살펴보지 않고 스프링의 프록시 팩토리 빈에 대해서 자세히 알아보자.
## 3. 스프링의 프록시 팩토리 빈
스프링은 트랜잭션 기술과 메일 발송 기술에 적용했던 서비스 추상화를 프록시 기술에도 동일하게 적용하고 있다. 자바에는 JDK에서 제공하는 다이내믹 프록시 외에도 편리하게 프록시를 만들 수 있도록 지원해주는 다양한 기술이 존재한다. 따라서 스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다. 생성된 프록시는 스프링의 빈으로 등록돼야 한다. **스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공해준다.** 스프링의 ProxyFactoryBean은 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈이다. ProxyFactoryBean은 순수하게 프록시를 생성하는 작업만을 담당하고, 프록시를 통해 제공해줄 부가기능은 별도의 빈에 둘 수 있다.

ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다. MethodInterceptor는 InvocationHandler와 비슷하지만 한 가지 다른 점이 있다. InvocationHandler의 invoke() 메서드는 타깃 오브젝트에 대한 정보를 제공하지 않는다. 따라서 타깃은 InvocationHandler를 구현한 클래스가 직접 알고 있어야 한다. 반면에 MethodInterceptor의 invoke() 메서드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다. 그 차이 덕분에 MethodInterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다. 따라서 MethodInterceptor 오브젝트는 타깃이 다른 여러 프록시에서 함께 사용할 수 있고, 싱글톤 빈으로 등록 가능하다.
```java
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
**어드바이스 : 타깃이 필요 없는 순수한 부가기능을 담은 오브젝트**

InvocationHandler를 구현했을 때와 달리 MethodInterceptor를 구현한 UppercaseAdvice에는 타깃 오브젝트가 등장하지 않는다. MethodInterceptor로는 메서드 정보와 함께 타깃 오브젝트가 담긴 MethodInvocation 오브젝트가 전달된다. MethodInvocation은 일종의 콜백 오브젝트로, proceed() 메서드를 실행하면 타깃 오브젝트의 메서드를 내부적으로 실행해주는 기능이 있다(MethodInvocation 구현 클래스는 일종의 공유 가능한 템플릿처럼 동작).

바로 이 점이 JDK의 다이내믹 프록시를 직접 사용하는 코드와 스프링이 제공해주는 프록시 추상화 기능인 ProxyFactoryBean을 사용하는 코드의 가장 큰 차이점이자 ProxyFactoryBean의 장점이다. ProxyFactoryBean은 작은 단위의 템플릿/콜백 구조를 응용해서 적용했기 때문에 템플릿 역할을 하는 MethodInvocation을 싱글톤으로 두고 공유할 수 있다. 마치 SQL 파라미터 정보에 종속되지 않는 JdbcTemplate이기 때문에 수많은 DAO 메서드가 하나의 JdbcTemplate 오브젝트를 공유할 수 있는 것과 마찬가지다.

**포인트컷 : 부가기능 적용 대상 메서드 선정 방법**

![](/assets/tobyaop4.jpg)

스프링은 부가기능을 제공하는 오브젝트를 어드바이스라고 부르고, 메서드 선정 알고리즘을 담은 오브젝트를 포인트컷이라고 부른다. 어드바이스와 포인트컷은 모두 프록시에 DI로 주입돼서 사용된다. 두 가지 모두 여러 프록시에서 공유가 가능하도록 만들어지기 때문에 스프링의 싱글톤 빈으로 등록이 가능하다.

프록시는 클라이언트로부터 요청을 받으면 먼저 포인트컷에게 부가기능을 부여할 메서드인지를 확인해달라고 요청한다. 포인트컷은 Pointcut 인터페이스를 구현해서 만들면 된다. 프록시는 포인트컷으로부터 부가기능을 적용할 대상 메서드인지 확인받으면, MethodInterceptor 타입의 어드바이스를 호출한다.
```java
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
포인트컷이 필요 없을 때는 ProxyFactoryBean의 addAdvice() 메서드를 호출해서 어드바이스만 등록하면 됐다. 그런데 포인트컷을 함께 등록할 때는 어드바이스와 포인트컷을 Advisor 타입으로 묶어서 addAdvisor() 메서드를 호출해야 한다. 왜 굳이 별개의 오브젝트로 묶어서 등록해야 할까? 그 이유는 ProxyFactoryBean에는 여러 개의 어드바이스와 포인트컷이 추가될 수 있기 때문이다. 포인트컷과 어드바이스를 따로 등록하면 어떤 어드바이스(부가기능)에 대해 어떤 포인트컷(메서드 선정)을 적용할지 애매해지기 때문이다. 그래서 이 둘을 Advisor 타입의 오브젝트에 담아서 조합을 만들어 등록하는 것이다.
`어드바이저 = 포인트컷(메서드 선정 알고리즘) + 어드바이스(부가기능)`

**ProxyFactoryBean을 이용해 트랜잭션 기능 적용**
```java
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
```xml
<bean id="transactionAdvice" class="xxx.xxx.service.TransactionAdvice">
  <property name="transactionManager" ref="transactionManager" />
</bean>
```
포인트컷 빈 설정
```xml
<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
  <property name="mappedName" ref="upgrade*" />
</bean>
```
어드바이저 빈 설정
```xml
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
  <property name="advice" ref="transactionAdvice" />
  <property name="pointcut" ref="transactionPointcut" />
</bean>
```
이제 ProxyFactoryBean을 등록할 차례다. 아래와 같이 프로퍼티에 타깃 빈과 어드바이저 빈을 지정해주면 된다.
```xml
<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
  <property name="target" ref="userServiceImpl" />
  <property name="interceptorNames">
    <list>
      <value>transactionAdvisor</value>
    </list>
  </property>
</bean>
```
어드바이저는 interceptorNames라는 프로퍼티를 통해 넣는다. 프로퍼티 이름이 advisor가 아닌 이유는 어드바이스와 어드바이저를 혼합해서 설정할 수 있도록 하기 위해서다. 그래서 property 태그의 ref 애트리뷰트를 통한 설정 대신 list와 value 태크를 통해 여러 개의 값을 넣을 수 있도록 하고 있다. value 태그에는 어드바이스 또는 어드바이저로 설정한 빈의 아이디를 넣으면 된다. 한 개 이상을 넣을 수 있다.

**어드바이스와 포인트컷의 재사용**

![](/assets/tobyaop5.jpg)

위의 그림은 ProxyFactoryBean을 이용해서 많은 수의 서비스 빈에게 트랜잭션 부가기능을 적용했을 때의 구조다. 트랜잭션 부가기능을 담은 TransactionAdvice는 하나만 만들어서 싱글톤 빈으로 등록해주면, DI 설정을 통해 모든 서비스에 적용이 가능하다. 메서드 선정 방식이 달라지는 경우만 포인트컷의 설정을 따로 등록하고 어드바이저로 조합해서 적용해주면 된다.
## 4. 스프링 AOP
아직 한 가지 해결할 과제가 남아 있다. 부가기능의 적용이 필요한 타깃 오브젝트마다 거의 비슷한 내용의 ProxyFactoryBean 빈 설정정보를 추가해주는 부분이다. 새로운 타깃이 등장했다고 해서 코드를 손댈 필요는 없어졌지만, 설정은 매번 복사하고 붙이고 target 프로퍼티의 내용을 수정해줘야 한다. 이런 류의 중복은 어떻게 제거할까?

### 빈 후처리기를 이용한 자동 프록시 생성기
스프링은 OCP의 가장 중요한 요소인 유연한 확장이라는 개념을 스프링 컨테이너 자신에게도 다양한 방법으로 적용하고 있다. 그래서 스프링은 컨테이너로서 제공하는 기능 중에서 변하지 않는 핵심적인 부분외에는 대부분 확장할 수 있는 확장 포인트를 제공해준다.

그중에서 관심을 가질 만한 확장 포인트는 바로 BeanPostProcessor 인터페이스를 구현해서 만드는 빈 후처리기다. 빈 후처리기는 이름 그대로 스프링 빈 오브젝트로 만들어지고 난 후에, 빈 오브젝트를 다시 가공할 수 있게 해준다. 여기서는 스프링이 제공하는 빈 후처리기 중의 하나인 DefaultAdvisorAutoProxyCreator를 살펴보겠다. 이름을 보면 알 수 있듯이 DefaultAdvisorAutoProxyCreator는 어드바이저를 이용한 자동 프록시 생성기다.

빈 후처리기를 스프링에 적용하는 방법은 간단하다. 빈 후처리기 자체를 빈으로 등록하는 것이다. 스프링은 빈 후처리기가 빈으로 등록되어 있으면 빈 오브젝트가 생성될 때마다 빈 후처리기에 보내서 후처리 작업을 요청한다. 빈 후처리기는 빈 오브젝트의 프로퍼티를 강제로 수정할 수도 있고 별도의 초기화 작업을 수행할 수도 있다. 심지어는 만들어진 빈 오브젝트 자체를 바꿔치기 할 수도 있다. 따라서 스프링이 설정을 참고해서 만든 오브젝트가 아닌 다른 오브젝트를 빈으로 등록시키는 것이 가능하다. 이를 잘 이용하면 스프링이 생성하는 빈 오브젝트의 일부를 프록시로 포장하고, 프록시를 빈으로 대신 등록할 수도 있다. 바로 이것이 자동 프록시 생성 빈 후처리기다.

![](/assets/tobyaop6.jpg)

위의 그림은 빈 후처리기를 이용한 자동 프록시 생성 방법을 설명한다. DefaultAdvisorAutoProxyCreator 빈 후처리기가 등록되어 있으면 스프링은 빈 오브젝트를 만들 때마다 후처리기에게 빈을 보낸다. DefaultAdvisorAutoProxyCreator는 빈으로 등록된 모든 어드바이저 내의 포인트컷을 이용해 전달받은 빈이 프록시 적용 대상인지 확인한다. 프록시 적용 대상이면 그때는 내장된 프록시 생성기에게 현재 빈에 대한 프록시를 만들게 하고, 만들어진 프록시에 어드바이저를 연결해준다. 빈 후처리기는 프록시가 생성되면 원래 컨테이너가 전달해준 빈 오브젝트 대신 프록시 오브젝트를 컨테이너에게 돌려준다. 컨테이너는 최종적으로 빈 후처리기가 돌려준 오브젝트를 빈으로 등록하고 사용한다.

적용할 빈을 선정하는 로직이 추가된 포인트컷이 담긴 어드바이저를 등록하고 빈 후처리기를 사용하면 일일이 ProxyFactoryBean 빈을 등록하지 않아도 타깃 오브젝트에 자동으로 프록시가 적용되게 할 수 있다. 마지막 남은 번거로운 ProxyFactoryBean 설정 문제를 말끔하게 해결해주는 놀라운 방법이다.

**확장된 포인트컷**

지금까지 포인트컷이란 타깃 오브젝트의 메서드 중에서 어떤 메서드에 부가기능을 적용할지를 선정해주는 역할을 한다고 했다. 그러나 포인트컷은 등록된 빈 중에서 어떤 빈에 프록시를 적용할지를 선택하는 기능이 또 있다.
```java
public interface Pointcut {
  ClassFilter getClassFilter(); // 프록시를 적용할 클래스인지 확인해준다.
  MethodMatcher getMethodMatcher(); // 어드바이스를 적용할 메서드인지 확인해준다.
}
```
ProxyFactoryBean에서는 굳이 클래스 레벨의 필터는 필요 없었지만, 모든 빈에 대해 프록시 자동 적용 대상을 선별해야 하는 빈 후처리기인 DefaultAdvisorAutoProxyCreator는 클래스와 메서드 선정 알고리즘을 모두 갖고 있는 포인트컷이 필요하다. 그런 포인트컷과 어드바이스가 결합되어 있는 어드바이저가 등록되어 있어야 한다.

**클래스 필터를 적용한 포인트컷 작성**

만들어야 할 클래스는 하나 뿐이다. 메서드 이름만 비교하던 포인트컷인 NameMethodPointcut을 상속해서 프로퍼티로 주어진 이름 패턴을 가지고 클래스 이름을 비교하는 ClassFilter를 추가하도록 만들 것이다.
```java
public class NameMatchClassMethodPointcut extends NameMatchMethodPointcut {
  public void setMappedClassName(String mappedClassName) {
    this.setClassFilter(new SimpleClassFilter(mappedClassName));
  }
  
  static class SimpleClassFilter implements ClassFilter {
    String mappedName;
    
    private SimpleClassFilter(String mappedName) {
      this.mappedName = mappedName;
    }
    
    public boolean matches(Class<?> clazz) {
      return PatternMatchUtils.simpleMatch(mappedName, clazz.getSimpleName());
    }
  }
}
```
**어드바이저를 이용하는 자동 프록시 생성기 등록**

적용할 자동 프록시 생성기인 DefaultAdvisorAutoProxyCreator는 등록된 빈 중에서 Advisor 인터페이스를 구현한 것을 모두 찾는다. 그리고 생성되는 모든 빈에 대해 어드바이저의 포인트컷을 적용해보면서 프록시 적용 대상을 선정한다. 빈 클래스가 프록시 선정 대상이라면 프록시를 만들어 원래 빈 오브젝트와 바꿔치기한다. 원래 빈 오브젝트는 프록시 뒤에 연결돼서 프록시를 통해서만 접근 가능하게 바뀌는 것이다. 따라서 타깃 빈에 의존한다고 정의한 다른 빈들은 프록시 오브젝트를 대신 DI 받게 될 것이다.
DefaultAdvisorAutoProxyCreator 등록은 다음 한 줄이면 충분하다. 
```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" />
```
**포인트컷 등록**

아래와 같이 기존의 포인트컷 설정을 삭제하고 새로 만든 클래스 필터 지원 포인트컷을 빈으로 등록한다. ServiceImpl로 이름이 끝나는 클래스와 upgrade로 시작하는 메서드를 선정해주는 포인트컷이다.
```xml
<bean id="transactionPointcut" class="xx.xx.NameMatchClassMethodPointcut">
  <property name="mappedClassName" value="*ServiceImpl" />
  <property name="mappedName" value="upgrade*" />
</bean>
```
**ProxyFactoryBean 제거와 서비스 빈의 원상복구**

프록시를 도입했던 때부터 아이디를 바꾸고 프록시에 DI 돼서 간접적으로 사용돼야 했던 userServiceImpl 빈의 아이디를 이제는 당당하게 userService로 되돌려놓을 수 있다. 더 이상 명시적인 프록시 팩토리 빈을 등록하지 않기 때문이다. 마지막으로 남았던 ProxyFactoryBean 타입의 빈은 삭제해버려도 좋다.
```xml
<bean id="userService class="xx.xx.UserServiceImpl">
  <property name="userDao" ref="userDao" />
  <property name="mailSender" ref="mailSender" />
</bean>
```
### AOP : 애스펙트 지향 프로그래밍
애스팩트란 그 자체로 애플리케이션의 핵심기능을 담고 있지는 않지만, 애플리케이션을 구성하는 중요한 한 가지 요소이고, 핵심기능에 부가되어 의미를 갖는 특별한 모듈을 가리킨다.

애스팩트는 그 단어의 의미대로 애플리케이션을 구성하는 한 가지 측면이라고 생각 할 수 있다.

![](/assets/tobyaop7.jpg)

위의 그림을 보면 왼쪽은 애스펙트로 부가기능을 분리하기 전의 상태다. 핵심기능은 깔끔한 설계를 통해서 모듈화되어 있고, 객체지향적인 장점을 잘 살릴 수 있도록 만들었지만, 부가기능이 핵심기능의 모듈에 침투해 들어가면서 설계와 코드가 모두 지저분해졌다.

오른쪽 그림은 이렇게 핵심기능 코드 사이에 침투한 부가기능을 독립적인 모듈인 애스펙트로 구분해낸 것이다. 이렇게 애플리케이션의 핵심적인 기능에서 부가적인 기능을 분리해서 애스펙트라는 독특한 모듈로 만들어서 설계하고 개발하는 방법을 AOP라고 부른다.

**AOP 네임스페이스**
```xml
<aop:config>
  <aop:pointcut id="transactionPointcut" expression="execution(* *..*ServiceImpl.upgrade*(..))" />
  <aop:advisor advice-reg="transactionAdvice" pointcut-ref="transactionPointcut" />
</aop:config>
```
`<aop:config>` : AspectJAdvisorAutoProxyCreator를 빈으로 등록해준다.

`<aop:pointcut>` : AspectJExpressionPointcut을 빈으로 등록해준다. 

`<aop:advisor>` : DefaultBeanFactoryPointcutAdvisor를 빈으로 등록해준다.
## 5. 트랜잭션 속성
PlatformTransactionManager로 대표되는 스프링의 트랜잭션 추상화를 설명하면서 그냥 넘어간 게 한 가지 있다. 트랜잭션 매니저에서 트랜잭션을 가져올 때 사용한 DefaultTransactionDefinition 오브젝트다.
```java
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
```
트랜잭션을 시작한다고 하지 않고 트랜잭션을 가져온다고 하는 이유는 차차 설명하기로 하고, 일단 트랜잭션을 가져올 때 파라미터로 트랜잭션 매니저에게 전달하는 DefaultTransactionDefinition의 용도가 무엇인지 알아보자.

DefaultTransactionDefinition이 구현하고 있는 TransactionDefinition 인터페이스는 트랜잭션의 동작방식에 영향을 줄 수 있는 네 가지 속성을 정의하고 있다.

**트랜잭션 전파**

트랜잭션의 경계에서 이미 진행 중인 트랜잭션이 있을 때 또는 없을 때, 어떻게 동작할 것인가를 결정하는 방식을 말한다.

![](/assets/tobyaop8.jpg)

위 그림의 트랜잭션 전파와 같이 각각 독립적인 트랜잭션 경계를 가진 두 개의 코드가 있다고 하자. 그런데 A의 트랜잭션이 시작되고 아직 끝나지 않은 시점에서 B를 호출했다면 B의 코드는 어떤 트랜잭션 안에서 동작해야 할까? 대표적으로 다음과 같은 트랜잭션 전파 속성을 줄 수 있다.

- PROPAGATION_REQUIRED

  가장 많이 사용되는 트랜잭션 전파 속성이다. 진행 중인 트랜잭션이 없으면 새로 시작하고, 이미 시작된 트랜잭션이 있으면 이에 참여한다. DefaultTransactionDefinition의 트랜잭션 전파 속성은 바로 이 PROPAGATION_REQUIRED다.
  
- PROPAGATION_REQUIRES_NEW

  항상 새로운 트랜잭션을 시작한다. 즉, 앞에서 시작된 트랜잭션이 있든 없든 상관없이 새로운 트랜잭션을 만들어서 독자적으로 동작하게 한다. 독립적인 트랜잭션이 보장돼야 하는 코드에 적용할 수 있다.
  
- PROPAGATION_NOT_SUPPORTED

  이 속성을 사용하면 트랜잭션 없이 동작하도록 만들 수도 있다.

이 외에도 다양한 트랜잭션 전파 속성을 사용할 수 있다. 트랜잭션 매니저를 통해 트랜잭션을 시작하려고 할 때 getTransaction()이라는 메서드를 사용하는 이유는 바로 이 트랜잭션 전파 속성이 있기 때문이다.

**격리수준**

모든 DB 트랜잭션은 격리수준(isolation level)을 갖고 있어야 한다. 서버환경에서는 여러 개의 트랜잭션이 동시에 진행될 수 있다. 따라서 적절하게 격리수준을 조정해서 가능한 한 많은 트랜잭션을 동시에 진행시키면서도 문제가 발생하지 않게 하는 제어가 필요하다.

**제한시간**

트랜잭션을 수행하는 제한시간(timeout)을 설정할 수 있다. DefaultTransactionDefinition의 기본 설정은 제한시간이 없다는 것이다. 제한시간은 트랜잭션을 직접 시작할 수 있는 PROPAGATION_REQUIRED나 PROPAGATION_REQUIRES_NEW와 함께 사용해야만 의미가 있다.

**읽기전용**

읽기전용으로 설정해두면 트랜잭션 내에서 데이터를 조작하는 시도를 막아줄 수 있다. 또한 데이터 액세스 기술에 따라서 성능이 향상될 수도 있다.

트랜잭션 정의를 수정하려면 어떻게 해야 할까? TransactionDefinition 오브젝트를 생성하고 사용하는 코드는 트랜잭션 경계설정 기능을 가진 TransactionAdvice다. 트랜잭션 정의를 바꾸고 싶으면 디폴트 속성을 갖고 있는 DefaultTransactionDefinition을 사용하는 대신 외부에서 정의된 TransactionDefinition 오브젝트를 DI 받아서 사용하도록 만들면 된다. TransactionDefinition 타입의 빈을 정의해두면 프로퍼티를 통해 원하는 속성을 지정해줄 수 있다. 하지만 이 방법으로 트랜잭션 속성을 변경하면 TransactionAdvice를 사용하는 모든 트랜잭션의 속성이 한꺼번에 바뀐다는 문제가 있다. 원하는 메서드만 선택해서 독자적인 트랜잭션 정의를 적용할 수 있는 방법은 없을까?

**트랜잭션 인터셉터와 트랜잭션 속성**

스프링에는 편리하게 트랜잭션 경계설정 어드바이스로 사용할 수 있도록 만들어진 TransactionInterceptor가 존재한다. TransactionInterceptor 어드바이스의 동작방식은 기존에 만들었던 TransactionAdvice와 다르지 않다. 다만 트랜잭션 정의를 메서드 이름 패턴을 이용해서 다르게 지정할 수 있는 방법을 추가로 제공해줄 뿐이다.

TransactionInterceptor는 PlatformTransactionManager와 Properties 타입의 두 가지 프로퍼티를 갖고 있다. 트랜잭션 매니저 프로퍼티는 잘 알고 있지만 Properties 타입의 프로퍼티는 처음 보는 것이다.

Properties 타입인 두 번째 프로퍼티 이름은 transactionAttributes로, 트랜잭션 속성을 정의한 프로퍼티다. 트랜잭션 속성은 TransactionDefinition의 네 가지 기본 항목에 rollbackOn()이라는 메서드를 하나 더 갖고 있는 TransactionAttribute 인터페이스로 정의된다. rollbackOn() 메서드는 어떤 예외가 발생하면 롤백을 할 것인가를 결정하는 메서드다. 이 TransactionAttribute를 이용하면 트랜잭션 부가기능의 동작 방식을 모두 제어할 수 있다.

![](/assets/tobyaop9.jpg)

위 트랜잭션 경계설정 코드를 다시 살펴보면 트랜잭션 부가기능의 동작방식을 변경할 수 있는 곳이 두 군데 있다는 사실을 알 수 있다. TransactionAdvice는 RuntimeException이 발생하는 경우에만 트랜잭션을 롤백시킨다. 하지만 런타임 예외가 아닌 경우에는 트랜잭션이 제대로 처리되지 않고 메서드를 빠져나가게 되어 있다. UserService는 런타임 예외만 던진다는 사실을 알기 때문에 일단 이렇게 정의해도 상관없지만, 체크 예외를 던지는 타깃에 사용한다면 문제가 될 수 있다. 그렇다면 런타임 예외만이 아니라 모든 종류의 예외에 대해 트랜잭션을 롤백 시키도록 해야 할까? 그래서는 안 된다. 비지니스 로직상의 예외 경우를 나타내기 위해 타깃 오브젝트가 체크 예외를 던지는 경우에는 DB 트랜잭션은 커밋시켜야 하기 때문이다. 2장에서 설명했듯이 일부 체크 예외는 정상적인 작업 흐름 안에서 사용될 수도 있다.

스프링이 제공하는 TransactionInterceptor에는 기본적으로 두 가지 종류의 예외 처리 방식이 있다. 런타임 예외가 발생하면 트랜잭션은 롤백된다. 반면에 타깃 메서드가 런타임 예외가 아닌 체크 예외를 던지는 경우에는 이것을 예외상황으로 해석하지 않고 일종의 비지니스 로직에 따른, 의미가 있는 리턴 방식의 한 가지로 인식해서 트랜잭션을 커밋해버린다.

그런데 TransactionInterceptor의 이러한 예외처리 기본 원칙을 따르지 않는 경우가 있을 수 있다. 그래서 TransactionAttribute는 rollbackOn()이라는 속성을 둬서 기본 원칙과 다른 예외처리가 가능하게 해준다. 이를 활용하면 특정 체크 예외의 경우는 트랜잭션을 롤백시키고, 특정 런타임 예외에 대해서는 트랜잭션을 커밋시킬 수도 있다.

TransactionInterceptor는 이런 TransactionAttribute를 Properties라는 일종의 맵 타입 오브젝트로 전달받는다. 아래 설정은 메서드 이름 패턴과 문자열로 된 트랜잭션 속성을 이용해서 정의한 TransactionInterceptor 타입 빈의 예다.

![](/assets/tobyaop10.jpg)

세 가지 메서드 이름 패턴에 대한 트랜잭션 속성이 정의되어 있다. get으로 시작하는 모든 메서드에는 PROPAGATION_REQUIRED이면서 읽기전용이고 시간제한은 30초다.

cf) readOnly나 timeout 등은 트랜잭션이 처음 시작될 때가 아니라면 적용되지 않는다.

**tx 네임스페이스를 이용한 설정 방법**

![](/assets/tobyaop11.jpg)

### 프록시 방식 AOP는 같은 타깃 오브젝트 내의 메서드를 호출할 때는 적용되지 않는다.
이건 전략이라기보다는 주의사항이다. 프록시 방식의 AOP에서는 프록시를 통한 부가기능의 적용은 클라이언트로부터 호출이 일어날 때만 가능하다. 여기서 클라이언트는 인터페이스를 통해 타깃 오브젝트를 사용하는 다른 모든 오브젝트를 말한다. 반대로 **타깃 오브젝트가 자기 자신의 메서드를 호출할 때는 프록시를 통한 부가기능의 적용이 일어나지 않는다.**

![](/assets/tobyaop12.jpg)

위 그림은 트랜잭션 프록시가 타깃에 적용되어 있는 경우의 메서드 호출 과정을 보여준다. delete()와 update()는 모두 트랜잭션 적용 대상인 메서드다. 따라서 [1]과 [3]처럼 클라이언트로부터 메서드가 호출되면 트랜잭션 프록시를 통해 타깃 메서드로 호출이 전달되므로 트랜잭션 경계설정 부가기능이 부여될 것이다.

하지만 [2]의 경우는 다르다. 일단 타깃 오브젝트 내로 들어와서 타깃 오브젝트의 다른 메서드를 호출하는 경우에는 프록시를 거치지 않고 직접 타깃의 메서드가 호출된다. 따라서 트랜잭션 속성이 전혀 반영되지 않는다.

만약 update() 메서드에 대해 트랜잭션 전파 속성을 REQUIRES_NEW라고 해놨더라도 같은 타깃 오브젝트에 있는 delete() 메서드를 통해 update()가 호출되면 트랜잭션 전파 속성이 적용되지 않으므로 REQUIRES_NEW는 무시되고 프록시의 delete() 메서드에서 시작한 트랜잭션에 단순하게 참여하게 될 뿐이다.

타깃 안에서의 호출에는 프록시가 적용되지 않는 문제를 해결할 수 있는 방법은 두 가지가 있다. 하나는 스프링 API를 이용해 프록시 오브젝트에 대한 레퍼런스를 가져온 뒤에 같은 오브젝트의 메서드 호출도 프록시를 이용하도록 강제하는 방법이다. 하지만 별로 추천되지 않는다(스프링 API와 프록시 호출 코드 등장은 바람직하지 않다). 다른 방법은 AspectJ와 같은 타깃의 바이트코드를 직접 조작하는 방식의 AOP 기술을 적용하는 것이다.
## 6. 애노테이션 트랜잭션 속성과 포인트컷
### @Transactional
```java
// 애노테이션을 사용할 대상을 지정한다. 여기에 사용된 메서드와 타입(클래스, 인터페이스)처럼 한 개 이상의 대상을 지정할 수 있다.
@Target({ElementType.METHOD, ElementType.TYPE})
// 애노테이션 정보가 언제까지 유지되는지를 지정한다. 이렇게 설정하면 런타임 때도 애노테이션 정보를 리플렉션을 통해 얻을 수 있다.
@Retention(RetentionPolicy.RUNTIME)
@Inherited //상속을 통해서도 애노테이션 정보를 얻을 수 있게 한다.
@Documented
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";
    @AliasFor("value")
    String transactionManager() default "";
    Propagation propagation() default Propagation.REQUIRED;
    Isolation isolation() default Isolation.DEFAULT;
    int timeout() default -1;
    boolean readOnly() default false;
    Class<? extends Throwable>[] rollbackFor() default {};
    String[] rollbackForClassName() default {};
    Class<? extends Throwable>[] noRollbackFor() default {};
    String[] noRollbackForClassName() default {};
}
```
@Transactional 애노테이션을 트랜잭션 속성정보로 사용하도록 지정하면 스프링은 @Transactional이 부여된 모든 오브젝트를 자동으로 타깃 오브젝트로 인식한다. 이때 사용되는 포인트컷은 TransactionAttributeSourcePointcut이다. TransactionAttributeSourcePointcut은 스스로 표현식과 같은 선정기준을 갖고 있진 않다. 대신 @Transactional이 타입 레벨이든 메서드 레벨이든 상관없이 부여된 빈 오브젝트를 모두 찾아서 포인트컷의 선정 결과로 돌려준다. @Transactional은 기본적으로 트랜잭션 속성을 정의하는 것이지만, 동시에 포인트컷의 자동등록에도 사용된다.

![](/assets/tobyaop13.jpg)

위의 그림은 @Transactional 애노테이션을 사용했을 때 어드바이저의 동작방식을 보여준다. TransactionInterceptor는 메서드 이름 패턴을 통해 부여되는 일괄적인 트랜잭션 속성정보 대신 @Transactional 애노테이션의 엘리먼트에서 트랜잭션 속성을 가져오는 AnnotationTransactionAttributeSource를 사용한다. @Transactional은 메서드마다 다르게 설정할 수도 있으므로 매우 유연한 트랜잭션 속성 설정이 가능해진다. 

동시에 포인트컷도 @Transactional을 통한 트랜잭션 속성정보를 참조하도록 만든다. @Transactional로 트랜잭션 속성이 부여된 오브젝트라면 포인트컷의 선정 대상이기도 하기 때문이다. 이 방식을 이용하면 포인트컷과 트랜잭션 속성을 애노테이션 하나로 지정할 수 있다. 트랜잭션 속성은 타입 레벨에 일괄적으로 부여할 수도 있지만, 메서드 단위로 세분화해서 트랜잭션 속성을 다르게 지정할 수도 있기 때문에 매우 세밀한 트랜잭션 속성 제어가 가능해진다.
### 대체 정책
스프링은 @Transactional을 적용할 때 4단계의 대체(fallback)정책을 이용하게 해준다. 메서드의 속성을 확인할 때 타깃 메서드, 타깃 클래스, 선언 메서드, 선언 타입의 순서에 따라서 @Transactional이 적용됐는지 차례로 확인하고, 가장 먼저 발견되는 속성정보를 사용하게 하는 방법이다. 이런 식으로 끝까지 발견되지 않으면 트랜잭션 적용 대상이 아니라고 판단한다.

![](/assets/tobyaop14.jpg)

위와 같이 정의된 인터페이스와 구현 클래스가 있다고 하자. @Transactional을 부여할 수 있는 위치는 총 6개다. 스프링은 트랜잭션 기능이 부여될 위치인 타깃 오브젝트의 메서드부터 시작해서 @Transactional 애노테이션이 존재하는지 확인한다. 따라서 [5]와 [6]이 @Transactional이 위치할 수 있는 첫 번째 후보다. 여기서 애노테이션이 발견되면 바로 애노테이션의 속성을 가져다 해당 메서드의 트랜잭션 속성으로 사용한다.

메서드에서 @Transactional을 발견하지 못하면, 다음은 타깃 클래스인 [4]에 @Transactional이 존재하는지 확인한다. 이를 통해서 @Transactional이 타입 레벨, 즉 클래스에 부여되면 해당 클래스의 모든 메서드의 공통적으로 적용되는 속성이 될 수 있다. 메서드 레벨에 @Transactional이 없다면 모두 클래스 레벨의 속성을 사용할 것이기 때문이다. 메서드가 여러 개라면 클래스 레벨에 @Transactional을 부여하는 것이 편리하다. 특정 메서드만 공통 속성을 따르지 않는다면 해당 메서드에만 추가로 @Transactional을 부여해주면 된다. 대체 정책에서 지정한 순서에 따라서 항상 메서드에 부여된 @Transactional이 가장 우선이기 때문에 @Transactional이 붙은 메서드는 클래스 레벨의 속성을 무시하고 메서드 레벨의 속성을 사용할 것이다. 반면에 @Transactional을 붙이지 않은 여타 메서드는 클래스 레벨에 부여된 공통 @Transactional을 따르게 된다.

타깃 클래스에서도 @Transactional을 발견하지 못하면, 스프링은 메서드가 선언된 인터페이스로 넘어간다. 인터페이스에서도 먼저 메서드를 확인한다. 따라서 [2]와 [3]에 @Transactional이 부여됐는지 확인하고 있다면 이 속성을 적용한다. 인터페이스 메서드에도 없다면 마지막 단계인 인터페이스 타입 [1]의 위치에 애노테이션이 있는지 확인한다.

기본적으로 @Transactional 적용 대상은 클라이언트가 사용하는 인터페이스가 정의한 메서드이므로 @Transactional도 타깃 클래스보다는 인터페이스에 두는 게 바람직하다. 하지만 인터페이스를 사용하는 프록시 방식의 AOP가 아닌 다른 방식으로 트랜잭션을 적용하면 인터페이스에 정의한 @Transactional은 무시되기 때문에 안전하게 타깃 클래스에 @Transactional을 두는 방법을 권장한다.

**트랜잭션 애노테이션 사용을 위한 설정**
```xml
<tx:annotation-driven />
```
```java
@EnableTransactionManagement
```