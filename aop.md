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









