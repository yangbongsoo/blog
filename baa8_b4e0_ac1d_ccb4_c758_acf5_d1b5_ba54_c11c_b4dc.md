# 모든 객체의 공통 메서드
Object에 정의된 비-final 메서드(equals, hashCode, toString, clone, finalize)의 명시적인 일반 규약들에 대해서 알아보자.(종료자 finalize는 제외) 추가로 Object의 메서드는 아니지만 특성이 비슷한 Comparable.compareTo도 알아보자.

###규칙8 : equals를 재정의할 때는 일반 규약을 따르라
**equals를 재정의하지 않아도 되는 경우**
1. 각각의 객체가 고유하다.
    1. 값(value) 대신 활성 개체(active entity)를 나타내는 Thread 같은 클래스가 이 조건에 부합.
2. 클래스에 논리적 동일성 검사 방법이 있건 없건 상관없다. 
    1. Random 클래스는 equals 메서드가 큰 의미 없다.  
3. 상위 클래스에서 재정의한 equals가 하위 클래스에서 사용하기에도 적당하다. 
4. 클래스가 private 또는 package-private로 선언되었고, equals 메서드를 호출할 일이 없다. 
    1. 하지만 저자는 재정의해서 `throw new AssertionError();`를 선언하라고 한다. 
5. 최대 하나의 객체만 존재하도록 제한하는 클래스.


**equals를 재정하는 것이 바람직할 때** : 객체 동일성(object equality)이 아닌 논리적 동일성(logical equality)의 개념을 지원하는 클래스일 때, 그리고 상위 클래스의 equals가 하위 클래스의 필요를 충족하지 못할 때 재정의해야 한다. 

**equals 메서드는 동치 관계를 구현한다.**<br>

**반사성:** null이 아닌 참조 x가 있을 때, x.equals(x)는 true를 반환한다.<br> 모든 객체는 자기 자신과 같아야 한다는 뜻이다. 

**대칭성:** null이 아닌 참조 x와 y가 있을 때, x.equals(y)는 y.equals(x)가 true일 때만 true를 반환한다. <br>
두 객체에게 서로 같은지 물으면 같은 답이 나와야 한다는 것이다. 

```
//대칭성 위반 클래스!!
public final class CaseInsensitiveString{
    private final String s;
    
    public CaseInsensitiveString(String s){
        if( s == null)
            throw new NullPointerException();
        this.s = s; 
    }
    
    //대칭성 위반 !! 
    @Override
    public boolean equals(Object o){
        if(o instanceof CaseInsensitiveString){
            return s.equalsIgnoreCase(((CaseInsensitiveString)o).s);
        }
        if(o instanceof String){ //한 방향으로만 정상 동작! 
            return s.equalsIgnoreCase((String)o); 
        }
        return false; 
    }
    ... //이하 생략 
}
```

```
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

**cis.equals(s)는 True**를 반환할 것이다. 하지만 **s.equals(cis)는 false**를 반환한다. 왜냐하면 String은 CaseInsensitiveString이 뭔지 모르기 때문이다. 





