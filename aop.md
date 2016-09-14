스프링에 적용된 가장 인기 있는 AOP 적용 대상은 바로 선언적 트랜잭션 기능이다. 
##트랜잭션 코드의 분리
###1. 데코레이터 패턴을 이용한 분리
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

###2. 다이내믹 프록시
프록시를 이용하는 방법은 유용하지만 매번 새로운 클래스를 정의해야 하고, 인터페이스의 구현해야 할 메서드가 많으면 모든 메서드를 일일히 구현해서 위임하는 코드를 넣어야하는 번거로움이 있다. 하지만 자바의 리플렉션 기능을 이용하면 손쉽게 구현할 수 있다.<br>
![](스크린샷 2016-09-14 오후 4.49.29.jpg)

다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트다. 다이내믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로 만들어진다. 클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용할 수 있다. 이 덕분에 프록시를 만들 때 인터페이스를 모두 구현해가면서 클래스를 정의하는 수고를 덜 수 있다. 프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를자동으로 만들어주기 때문이다.<br>

다이내믹 프록시가 인터페이스 구현 클래스의 오브젝트는 만들어주지만, 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야 한다. 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다. InvocationHandler 인터페이스는 다음과 같은 메서드 한 개만 가진 간단한 인터페이스다.






