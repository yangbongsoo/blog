# 객체의 생성과 삭제
객체를 만들어야하는 시점과 그 방법, 객체 생성을 피해야 하는 경우와 그 방법, 적절한 순간에 객체가 삭제되도록 보장하는 방법, 그리고 삭제 전에 반드시 이루어져야 하는 청소 작업들을 관리하는 방법을 살펴본다. 

###규칙1 : 생성자 대신 정적 팩터리 메서드를 사용할 수 없는지 생각해보라

클래스를 통해 객체를 만드는 일반적인 방법(public 생성자 이용)말고 또 다른 방법이 있다. 바로 **public static factory method**를 만드는 것이다. 
```
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

