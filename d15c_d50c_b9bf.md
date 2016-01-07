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
        PreparedStatement ps = c.PreparedStatement("delete from users");
        return ps;
    }
}
```