# 서비스 추상화
추상화란 하위 시스템의 공통점을 뽑아내서 분리시키는 것을 말한다. 그렇게 하면 하위 시스템이 어떤 것인지 알지 못해도, 또는 하위 시스템이 바뀌더라도 일관된 방법으로 접근할 수가 있다.

객체지향적인 코드는 다른 오브젝트의 데이터를 가져와서 작업하는 대신 데이터를 갖고 있는 다른 오브젝트에게 작업을 해달라고 요청한다. 오브젝트에게 데이터를 요구하지 말고 작업을 요청하라는 것이 객체지향 프로그래밍의 가장 기본이 되는 원리이다. 

일반적으로 서비스 추상화라고 하면 트랜잭션과 같이 기능은 유사하나 사용 방법이 다른, 로우레벨의 다양한 기술에 대해 추상 인터페이스와 일관성 있는 접근 방법을 제공해 주는 것을 말한다. 

**비지니스 로직 트랜잭션 문제점**

![](/assets/transactionproblem2.PNG)

`setAutoCommit(false)`로 트랜잭션의 시작을 선언하고 `commit()`또는 `rollback()`으로 트랜잭션을 종료하는 작업을 트랜잭션의 경계설정이라고 한다. 트랜잭션의 경계는 하나의 Connection이 만들어지고 닫히는 범위 안에 존재한다. 이렇게 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 로컬 트랜잭션이라고도 한다. 

그런데 위의 그림과 같이 트랜잭션의 문제점이 있다. DAO를 분리해놓았을 경우에는 이처럼 DAO 메서드를 호출할 때마다 하나의 새로운 트랜잭션이 만들어지는 구조가 될 수밖에 없다. **따라서 UserService의 모든 메서드 전체를 하나의 트랜잭션으로 묶을 방법이 없게된다.** 결국 비지니스 로직에 트랜잭션 경계설정을 해야하는데 그렇게되면 비지니스로직에 데이터 액세스 코드가 들어가게 되므로 단일책임원칙도 어긋나고, DB에 의존적인 코드가 돼버린다. 

이를 위해 스프링이 제안하는 방법은 독립적인 **트랜잭션 동기화** 방식이다. 트랜잭션 동기화란 UserService에서 트랜잭션을 시작하기 위해 만든 Connection 오브젝트를 특별한 저장소에 보관해두고, 이후에 호출되는 DAO의 메서드에서는 저장된 Connection을 가져다가 사용하게 하는 것이다. 정확히는 DAO가 사용하는 JdbcTemplate이 트랜잭션 동기화 방식을 이용하도록 하는 것이다. 그리고  트랜잭션이 모두 종료되면, 그때는 동기화를 마치면 된다.

![](/assets/transactionsync2.PNG)

(1) UserService는 Connection을 생성하고 (2) 이를 트랜잭션 동기화 저장소에 저장해두고 Connection의 `setAutoCommit(false)`를 호출해 트랜잭션을 시작시킨 후에 본격적으로 DAO의 기능을 이용하기 시작한다. (3) 첫 번째 update() 메서드가 호출되고, update() 메서드 내부에서 이용하는 JdbcTemplate 메서드에서는 가장 먼저 (4) 트랜잭션 동기화 저장소에 현재 시작된 트랜잭션을 가진 Connection 오브젝트가 존재하는지 확인한다. (2) upgradeLevels() 메서드 시작 부분에서 저장해둔 Connection을 발견하고 이를 가져온다. 가져온 (5) Connection을 이용해 PreparedStatememt를 만들어 수정 SQL을 실행한다. 트랜잭션 동기화 저장소에서 DB 커넥션을 가져왔을 때는 JdbcTemplate은 Connection을 닫지 않은 채로 작업을 마친다. 이렇게 해서 트랜잭션 안에서 첫 번째 DB 작업을 마쳤다. 여전히 Connection은 열려 있고 트랜잭션은 진행 중인 채로 트랜잭션 동기화 저장소에 저장되어 있다. (6) 두 번째 update()가 호출되면 이때도 마찬가지로 (7) 트랜잭션 동기화 저장소에서 Connection을 가져와 (8) 사용한다. (9) 마지막 update()도 (10) 같은 트랜잭션을 가진 Connection을 가져와 (11) 사용한다. 트랜잭션 내의 모든 작업이 정상적으로 끝났으면 UserService는 이제 (12) Connection의 `commit()`을 호출해서 트랜잭션을 완료시킨다. 마지막으로 (13) 트랜잭션 저장도가 더 이상 Connection 오브젝트를 저장해두지 않도록 이를 작업한다. 어느 작업 중에라도 예외상황이 발생하면 UserService는 즉시 Connection의 `rollback()`을 호출하고 트랜잭션을 종료할 수 있다. 

**트랜잭션 동기화 방식을 적용한 UserService**
```java
private DataSource dataSource;

//Connection을 생성할 때 사용할 DataSource를 DI 받도록 한다.
public void setDataSource(DataSource dataSource){
    this.dataSource = dataSource;
}

public void upgradeLevels() throws Exception{
    //트랜잭션 동기화 관리자를 이용해 동기화 작업을 초기화한다.
    TransactionSynchronizationManager.initSynchronization();
    
    //DB 커넥션을 생성하고 트랜잭션을 시작한다. 
    //이후의 DAO 작업은 모두 여기서 시작한 트랜잭션 안에서 진행된다.
    
    //DB 커넥션 생성과 동기화를 함께 해주는 유틸리티 메서드
    Connection c = DataSourceUtils.getConnection(dataSource);
    c.setAutoCommit(false);
    
    try{
        List<User> users = userDao.getAll();
        for(User user : users){
            if(canUpgradeLevel(user){
                upgradeLevel(user);
            }
        }
        c.commit(); //정상적으로 작업을 마치면 트랜잭션 커밋
    }catch(Exception e){
        c.rollback(); //예외가 발생하면 롤백한다.
        throw e;
    }finally{
        //스프링 유틸리티 메서드를 이용해 DB 커넥션을 안전하게 닫는다.
        DataSourceUtils.releaseConnection(c,dataSource);
        //동기화 작업 종료 및 정리
        TransactionSynchronizationManager.unbindResource(this.dataSource);
        TransactionSynchronizationManager.clearSynchronization();
    }
}
```
그런데 새로운 문제가 있다. 만약 하나의 트랜잭션 안에서 여러 개의 DB에 데이터를 넣는 작업을 해야한다면 JDBC의 Connection을 이용한 트랜잭션 방식인 로컬 트랜잭션으로는 불가능하다. 왜냐하면 로컬 트랜잭션은 하나의 DB Connection에 종속되기 때문이다. 따라서 별도의 트랜잭션 관리자를 통해 트랜잭션을 관리하는 글로벌 트랜잭션 방식을 사용해야 한다.(JTA) 그리고 하이버네이트를 이용한 트랜잭션 관리코드는 JDBC나 JTA의 코드와는 또 다르다. 결국 트랜잭션의 경계설정을 담당하는 코드의 유사패턴(공통점)을 추출해 추상화 시켜야 한다.

**스프링의 트랜잭션 서비스 추상화**

![](/assets/abstractlayer.PNG)

스프링이 제공하는 트랜잭션 추상화 방법을 UserService에 적용해보면 다음과 같다.
```java
public void upgradeLevels() {
  PlatformTransactionManager transactionManager = 
    new DataSourceTransactionManager(dataSource);
    
  // 트랜잭션 시작
  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
  
  try {
    List<User> users = userDao.getAll();
    for (User user : users) {
      if (canUpgradeLevel(user))
        upgradeLevel(user);
    }
    transactionManager.commit(status);
  } catch (RuntimeException e) {
    transcationManager.rollback(status);
    throw e;
  }
}
```
스프링이 제공하는 트랜잭션 경계설정을 위한 추상 인터페이스는 PlatformTransactionManager다. JDBC의 로컬 트랜잭션을 이용한다면 DataSourceTransactionManager 구현체를 사용하면 된다.

트랜잭션 추상화 API를 적용한 UserService 코드를 JTA를 이용하는 글로벌 트랜잭션으로 변경하려면 어떻게 해야 할까? 방법은 간단하다. PlatformTransactionManager 구현 클래스를 DataSourceTransactionManager에서 JTATransactionManager로 바꿔주기만 하면 된다. 만약 하이버네이트로 UserDao를 구현했다면 HibernateTransactionManager를, JPA를 적용했다면 JPATransactionManager를 사용하면 된다.

하지만 어떤 트랜잭션 매니저 구현 클래스를 사용할지 UserService 코드가 알고 있는 것은 DI 원칙에 위배된다. 컨테이너를 통해 외부에서 제공받게 하는 스프링 DI의 방식으로 바꾸자.
```java
public class UserService{
	...
	private PlatformTransactionManager transactionManager;

	public void setTransactionManager(PlatformTransactionManager transactionManager){
		this.transactionManager = transactionManager;
	}

	public void upgradeLevels(){
		TransactionStatus status = 
		this.transactionManager.getTransaction(new DefaultTransactionDefinition());

		try{
			List<User> users = userDao.getAll();
			for(User user : users){
				if(canUpgradeLevel(user){
					upgradeLevel(user);
				}
			}
			this.transactionManager.commit(status);
		}catch(RuntimeException e){
			this.transactionManager.rollback(status);
			throw e;
		}
	}
	...
}

```

```xml
<bean id="userService" class="springbook.user.service.UserService">
	<property name="userDao" ref="userDao" />
	<property name="transactionManager" ref="transactionManager"/>
</bean>

<bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>
<bean id="transactionManager" 
    class="org.springframework.transaction.jta.JtaTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>
```

**테스트 대역**
테스트 대상이 되는 오브젝트의 기능에만 충실하게 수행하면서 빠르게, 자주 테스트를 실행할 수 있도록 사용하는 오브젝트를 통틀어서 테스트 대역(test double)이라고 부른다. 대표적인 테스트 대역은 스텁(stub)이다.

스텁은 테스트 대상 오브젝트의 의존객체로서 존재하면서 테스트 동안에 코드가 정상적으로 수행할 수 있도록 돕는 것을 말한다.

때론 테스트 대상 오브젝트가 의존 오브젝트에게 출력한 값에 관심이 있을 경우가 있다. 또는 의존 오브젝트를 얼마나 사용했는가 하는 커뮤니케이션 행위 자체에 관심이 있을 수 있다. 문제는 이 정보는 테스트에서는 직접 알 수가 없다는 점이다. 이때 목 객체를 만들어서 사용해야 한다.
```java
static class MockMailSender implements MailSender {
  private List<String> requests = new ArrayList<>();
  
  public List<String> getRequests() {
    return requests;
  }
  
  public void send(SimpleMailMessage mailMessage) throws MailException {
    // 전송 요청을 받은 이메일 주소를 저장해둔다. 간단하게 첫 번째 수신자 메일 주소만 저장했다.
    requests.add(mailMessage.getTo()[0]);
  }
  
  ...
}
```
위 코드는 테스트 대상 오브젝트가 목 오브젝트에게 전달하는 출력정보를 저장해 두는 것이다. 
```java
테스트 코드

@Test
@DirtiesContext //컨텍스트의 DI 설정을 변경하는 테스트라는 것을 알려준다.
public void upgradeLevels() trows Exception {
  ...
  
  MockMailSender mockMailSender = new MockMailSender();
  userServce.setMailSender(mockMailSender);
  ...
  
  // Assertion
  List<String> request = mockMailSender.getRequests();
  assertThat(request.size(), is(2));
  assertThat(request.get(0), is(users.get(1).getEmail());
  ...
}
```
보통의 테스트 방법으로는 검증하기가 매우 까다로운 테스트 대상 오브젝트의 내부에서 일어나는 일이나 다른오브젝트 사이에서 주고받는 정보까지 검증하는 일이 손쉽다.