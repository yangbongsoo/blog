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

