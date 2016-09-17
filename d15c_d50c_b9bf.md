# 템플릿
**템플릿이란 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜서 효과적으로 활용할 수 있도록 하는 방법이다.**

예외처리 기능을 갖춘 DAO
```
public void deleteAll() throws SQLException{
    Connection c = null;
    PreparedStatement ps = null;
    
    //예외가 발생할 가능성이 있는 코드를 모두 try 블록으로 묶어준다. 
    try{

        c = dataSource.getConnection();
        ps = c.prepareStatement("delete from users");
        ps.executeUpdate();

    }catch(SQLException e){
        throw e;
    }finally{//예외 발생하건 안하건 항상 실행 

        if(ps != null){ //ps가 null일때 close 호출하면 NullPointerException 
            try{
                ps.close(); //close()도 예외 발생할 수 있다. 
            }catch(SQLException e){
            }
        }

        if(c != null){
            try{
                c.close();
            }catch(SQLException e){
            }
        }

    }// finally end
}
```
close()는 만들어진 순서의 반대로 하는 것이 원칙이다. 

이제 이 deleteAll() 메서드에 담겨 있던 변하지 않는 부분, 자주 변하는 부분을 전략 패턴을 사용해 깔끔하게 분리해보자.
![](strategypattern.PNG)

클라이언트 책임을 담당할 deleteAll() 메서드
```
public void deleteAll() throws SQLException{
    StatementStrategy st = new DeleteAllStatement();
    jdbcContextWithStatementStrategy(st);
}
```
메서드로 분리한 try/catch/finally 컨텍스트 코드

```
public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException{
    Connection c = null;
    PreparedStatement ps = null; 
    
    try{
        c = dataSource.getConnection();
        
        ps = stmt.makePreparedStatement(c);
        
        ps.executeUpdate();
    }catch(SQLException e){
        throw e;
    }finally{
        if(ps != null) { try { ps.close(); } catch (SQLException e) {} }
        if(c != null) { try { c.close(); } catch (SQLException e) {} }
    }
}
```
StatementStrategy 인터페이스
```
public interface StatementStrategy{
    PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}
```
deleteAll() 메서드의 기능을 구현한 StatementStrategy 전략 클래스 
```
public class DeleteAllStatement implements StatementStrategy{

    public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
        PreparedStatement ps = c.preparedStatement("delete from users");
     
        return ps;
    }
}
```
이 방법은 두 가지 개선할 부분이 있다. 첫째는 DAO 메서드마다 새로운 StatementStrategy 구현 클래스를 만들어야 한다는 점이다. 이렇게 되면 상속을 사용하는 템플릿 메서드 패턴을 적용했을 때보다 그다지 나을게 없다. 두 번째는 add() 메서드 같은 경우, 새로운 user에 대한 부가적인 정보가 있어서 번거롭게 인스턴스 변수를 만들어야 한다는 점이다. 
```
public class AddStatement implements StatementStrategy {
  User user;
  
  public AddStatement(User user) {
    this.user = user;
  }
  
  public PreparedStatement makePreparedStatement(Connection c) {
    ...
    ps.setString(1, user.getId());
    ps.setString(2, user.getName());
    ps.setString(3, user.getPassword());
    ...
  }
}
```
```
public void add(User user) throws SQLException {
  StatementStrategy st = new AddStatement(user);
  jdbcContextWithStatementStrategy(st);
}
```
이 두 가지 문제를 해결할 수 있는 방법을 생각해보자.
###로컬 클래스
StatementStrategy 전략 클래스를 매번 독립된 파일로 만들지 말고 UserDao 클래스 안에 내부 클래스로 정의해버리는 간단한 방법이 있다. 

```
//내부 클래스에서 외부의 변수를 사용할 때는 외부변수를 반드시 final로 선언해줘야된다.
public void add(final User user) throws SQLException{

    class AddStatement implements StatementStrategy{
        public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
            
            PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
            ps.setString(1, user.getId());
            ps.setString(2, user.getName());
            ps.setString(3, user.getPassword());
            
            return ps;
        }
    }// class end 
    
    StatementStrategy st = new AddStatement(); // 생성자 파라미터로 user 전달하지 않아도 된다.
    jdbcContextWithStatementStrategy(st); 
    
}
```
로컬 클래스로 만들어두니 장점이 많다. AddStatement는 복잡한 클래스가 아니므로 메서드 안에서 정의해도 그다지 복잡해 보이지 않는다. 메서드마다 추가해야 했던 클래스 파일을 하나 줄일 수 있다는 것도 장점이고, 내부 클래스의 특징을 이용해 로컬 변수를 바로 가져다 사용할 수 있다는 것도 큰 장점이다.
###익명 내부 클래스
클래스 선언과 오브젝트 생성이 결합된 형태로 만들어지며, 클래스를 재사용할 필요가 없고 구현한 인터페이스 타입으로만 사용할 경우에 유용하다. 

익명 내부 클래스는 선언과 동시에 오브젝트를 생성한다. 이름이 없기 때문에 클래스 자신의 타입을 가질 수 없고, 구현한 인터페이스 타입의 변수에만 저장할 수 있다. 
![](template-callback2.PNG)

```
public void add(final User user) throws SQLExecption{
    jdbcContextWithStatementStrategy(
        new StatementStrategy(){
               public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
                
                PreparedStatement ps = c.prepareStatement(
                    "insert into users(id, name, password) values(?,?,?)");
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());
                
                return ps;
            } 
        }
    );
}

```

```
public void deleteAll() throws SQLException{
    jdbcContextWithStatementStrategy(
        new StatementStrategy(){
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
            
                return c.preparedStatement("delete from users");
            }
        }
    );
}
```
###컨텍스트와 DI
**jdbcContext의 분리**<br>
전략 패턴의 구조로 보자면 UserDao의 메서드가 클라이언트이고, 익명 내부 클래스로 만들어지는 것이 개별적인 전략이고, jdbcContextWithStatementStrategy() 메서드는 컨텍스트다. 컨텍스트 메서드는 UserDao 내의 PreparedStatement를 실행하는 기능을 가진 메서드에서 공유할 수 있다. 그런데 JDBC의 일반적인 작업 흐름을 담고 있는 jdbcContextWithStatementStrategy()는 다른 DAO에서도 사용 가능하다. 그러니 jdbcContextWithStatementStrategy()를 UserDao 클래스 밖으로 독립시켜서 모든 DAO가 사용할 수 있게 해보자.<br>

아래 코드는 분리해서 만든 JdbcContext 클래스다.
```
public class JdbcContext { 
  private DataSource dataSource;
  
  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
  }
  
  public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;
    
    try {
      c = this.dataSource.getConnection();
      ps = stmt.makePreparedStatement(c);
      ps.executeUpdate();
    } catch(SQLException e) P
      throw e;
    } finally {
        if(ps != null) { try { ps.close(); } catch (SQLException e) {} }
        if(c != null) { try { c.close(); } catch (SQLException e) {} }
    }
  }
}
```
다음은 UserDao가 분리된 JdbcContext를 DI 받아서 사용할 수 있게 만든다.
```
public class UserDao {
  ...
  
  private JdbcContext jdbcContext;
  
  public void setJdbcContext(JdbcContext jdbcContext) {
    this.jdbcContext = jdbcContext;
  }
  
  public void add(final User user) throws SQLException {
    this.jdbcContext.workWithStatementStrategy(
      new StatementStrategy() { ... }
    );
  }
  
  public void deleteAll() throws SQLExcetion {
    this.jdbcContext.workWithStatementStrategy(
      new StatementStrategy() { ... }
    );
  }
}
```
###템플릿과 콜백
전략 패턴의 기본 구조에 익명 내부 클래스를 활용한 방식을 스프링에서는 템플릿/콜백 패턴이라고 부른다. 전략 패턴의 컨텍스트를 템플릿이라 부르고, 익명 내부 클래스로 만들어지는 오브젝트를 콜백이라고 부른다.<br>

cf) 콜백 : 실행되는 것을 목적으로 다른 오브젝트의 메서드에 전달되는 오브젝트를 말한다. 파라미터로 전달되지만 값을 참조하기 위한 것이 아니라 특정 로직을 담은 메서드를 실행시키기 위해 사용한다.<br>
![](스크린샷 2016-09-16 오후 11.09.22.jpg)

그런데 템플릿/콜백 방식에서 한 가지 아쉬운 점이 있다. DAO 메서드에서 매번 익명 내부 클래스를 사용하기 때문에 상대적으로 코드를 작성하고 읽기가 조금 불편하다는 점이다. 그래서 이번에는 복잡한 익명 내부 클래스의 사용을 최소화할 수 있는 방법을 찾아보자.<br>

재사용 가능한 콜백을 템플릿 클래스 안으로 옮기자. 엄밀히 말해서 템플릿은 JdbcContext 클래스가 아니라 workWithStatementStrategy() 메서드이므로 옮긴다고 해도 문제 될 것은 없다.
```
public class JdbcContext {
  ...
  public void executeSql(final String query) throws SQLException {
    workWithStatementStrategy(
      new StatementStrategy() {
        public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
          return c.prepareStatement(query);
        }
      }
    );
  }
}
```
executeSql() 메서드가 JdbcContext로 이동했으니 UserDao의 메서드에서도 아래와 같이 jdbcContext를 통해 executeSql() 메서드를 호출하도록 수정해야 한다.
```
public void deleteAll() throws SQLExcetpion {
  this.jdbcContext.executeSql("delete froms users");
}
```
###스프링의 JdbcTemplate
스프링이 제공하는 JDBC 코드용 기본 템플릿은 JdbcTemplate이다. 앞에서 만들었던 JdbcContext와 유사하지만 훨씬 강력하고 편리한 기능을 제공해준다. 
```
public class UserDao {
  ...
  private JdbcTemplate jdbcTemplate;
  
  public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
    this.dataSource = dataSource;
  }
}
```
이제 템플릿을 사용할 준비가 됐다.<br>

**update()**<br>
deleteAll()에 먼저 적용해보자. deleteAll()에 처음 적용했던 콜백은 StatementStrategy 인터페이스의 makePreparedStatement() 메서드다. 이에 대응되는 JdbcTemplate의 콜백은 PreparedStatementCreator 인터페이스의 createPreparedStatement() 메서드다. 
```
public void deleteAll() {
  this.jdbcTemplate.update(
    new PreparedStatementCreator() {
      public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
        return con.prepareStatement("delete from users");
      }
    }
  );
}
```
앞에서 만들었던 executeSql()은 SQL 문장만 전달하면 미리 준비된 콜백을 만들어서 템플릿을 호출하는 것까지 한 번에 해주는 편리한 메서드였다. JdbcTemplate에도 기능이 비슷한 메서드가 존재한다. 콜백을 받는 update() 메서드와 이름은 동일한데 파라미터로 SQL 문장을 전달한다는 것만 다르다.
```
public void deleteAll() {
  this.jdbcTemplate.update("delete from users");
}
```

**queryForInt()**<br>
다음은 아직 템플릿/콜백 방식을 적용하지 않았던 getCount 메서드에 JdbcTemplate을 적용해보자.
```
public int getCount() throws SQLException {
  Connection c = dataSource.getConnection();
  
  PreparedStatement ps = c.prepareStatement("select count(*) from users");
  
  ResultSet rs = ps.executeQuery();
  rs.next();
  int count = rs.getInt(1);
  
  rs.close();
  ps.close();
  c.close();
  
  return count;
}
```
getCount()는 SQL 쿼리를 실행하고 ResultSet을 통해 결과 값을 가져오는 코드다. 이런 작업 흐름을 가진 코드에서 사용할 수 있는 템플릿은 PreparedStatementCreator 콜백과 ResultSetExtractor 콜백을 파라미터로 받는 query() 메서드다. ResultSetExtractor 콜백은 템플릿이 제공하는 ResultSet을 이용해 원하는 값을 추출해서 템플릿에 전달하면, 템플릿은 나머지 작업을 수행한 뒤에 그 값을 query() 메서드의 리턴 값으로 돌려준다.
```
public int getCount() {
  return this.jdbcTemplate.query(new PreparedStatementCreator() {
    public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
      return con.prepareStatement("select count(*) from users");
    }
    }, new ResultSetExtractor<Integer>() {
      public Integer extractData(ResultSet rs) throws SQLException, DataAccessException {
        rs.next();
        return rs.getInt(1);
      }
    }
  );
}
```
