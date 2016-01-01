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
}
```

```
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

**cis.equals(s)는 True**를 반환할 것이다. 하지만 **s.equals(cis)는 false**를 반환한다. 왜냐하면 String은 CaseInsensitiveString이 뭔지 모르기 때문이다. 

그러므로 이 문제를 방지하려면 CaseInsensitiveString의 equals 메서드가 String 객체와 상호작용하지 않도록 해야 한다. 
```
@Override
public boolean equals(Object o){
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString)o).s.equalsIgnoreCase(s); 
}
```

**추이성:** null이 아닌 참조 x,y,z가 있을 때, x.equals(y)가 true이고 y.equals(z)가 true이면 x.equals(z)도 true이다. <br>

상위 클래스에 없는 새로운 값 컴포넌트를 하위 클래스에 추가하는 상황을 생각해 보자. 다시 말해 equals가 비교할 새로운 정보를 추가한다는 뜻이다. 

```
public class Point{
    private final int x;
    private final int y;
    
    public Point(int x, int y){
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o){
        if(!(o instanceof Point)){
            return false;
        }
        Point p = (Point)o;
        return p.x == x && p.y ==y; 
    }
}

```
이 클래스를 계승하여, 색상 정보를 추가해보자. 
```
public class ColorPoint extends Point{
    private final Color color; 
    
    public ColorPoint(int x, int y, Color color){
        super(x, y);
        this.color = color; 
    }
}
```
**ColorPoint 클래스의 equals 구현은 어떻게 해야 할까?** 구현을 생략한다면 Point의 equals가 그대로 상속되어서 추가된 색상 정보는 비교하지 못한다. 
```
//대칭성 위반 !!
@Override
public boolean equals(Object o){
    if(!(o instanceof ColorPoint)){
        return false; 
    }
    return super.equals(o) && ((ColorPoint)o).color == color;
}
```

위와 같이 equals를 구현했다면, 아래 코드에서 대칭성이 어떻게 위반되는것이 명확하게 확인된다.
```
Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1,2,Color.RED);

p.equals(cp); // true 왜냐면 Pointer의 equals는 color를 비교하지 않으니까  
cp.equals(p); // false 왜냐면 ColorPoint의 equals에서 color가 같을 수가 없으니까 
```

그렇다면 ColorPoint의 equals를 수정해서 Point 객체와 비교할때는 색상 정보를 무시하도록 하면 어떻게 될까? 
```
//추이성 위반!!
@Override
public boolean equals(Object o){
    if(!(o instanceof Point))
        return false; 
        
        //o가 Point 객체이면 색상은 비교하지 않는다. 
        if(!(o instanceof ColorPoint))
            return o.equals(this);
            
        //o가 ColorPoint이므로 모든 정보를 비교
        return super.equals(o) && ((ColorPoint) o).color == color; 
}
```
이렇게 하면 대칭성은 보존되지만 추이성은 깨진다. 
```
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```
p1.equals(p2)와 p2.equals(p3)는 모두 true를 반환하지만 p1.equals(p3)는 false를 반환한다. 

**이 문제의 해결책은 무엇인가?**<br>
사실 이것은 객체 지향 언어에서 동치 관계를 구현할 때 발생하는 본질적 문제다. `객체 생성가능 클래스를 계승하여 새로운 값 컴포넌트를 추가하면서 equals 규악을 어기지 않을 방법은 없다.`

equals 메서드를 구현할 때 instanceof 대신 getClass 메서드를 사용하면 기존 클래스를 확장하여 새로운 값 컴포넌트를 추가하더라도 equals 규약을 준수할 수 있다?
```
//리스코프 대체 원칙 위반
@Override public boolean equals(Object o){
    if(o == null || o.getClass() != getClass())
        return false;
    Point p = (Point)o;
    return p.x == x && p.y == y;
}
```
이렇게 하면 같은 클래스의 객체만 비교하게 된다. 하지만 아래의 예제를 보면 올바르지 않다는 것을 알 수 있다. 
```
//단위 원 상의 모든 점을 포함하도록 unitCircle 초기화 
private static final Set<Point> uniCircle;
static{
    unitCircle = new HashSet<Point>();
    unitCircle.add(new Point(1, 0));
    unitCircle.add(new Point(0, 1));
    unitCircle.add(new Point(-1, 0));
    unitCircle.add(new Point(0, -1));
}

public static boolean onUnitCircle(Point p){
    return unitCircle.contatins(p); 
}
```
```
public class CounterPoint extends Point{
    private static final AtomicInteger counter = new AtomicInteger();
    
    public CounterPoint(int x, int y){
        super(x, y);
        counter.incrementAndGet();
    }   
    
    public int numberCreated() { return counter.get(); } 
}
```
**리스코프 대체 원칙은 어떤 자료형의 중요한 속성은 하위 자료형에도 그대로 유지되어서, 그 자료형을 위한 메서드는 하위 자료형에도 잘 동작해야 한다는 원칙이다.** 그런데 CounterPoint 객체를 onUnitCircle 메서드의 인자로 넘기는 경우를 생각해보자. Point 클래스의 equals 메서드가 getClass를 사용하고 있다면, onUnitCircle 메서드는 CounterPoint 객체의 x나 y값에 상관없이 무조건 false를 반환할 것이다. 

이는 onUnitCircle 메서드가 이용하는 HashSet 같은 컬렉션이 객체 포함여부를 판단할 때 equals를 사용하기 때문이며, CounterPoint객체는 어떤 Point객체와도 같을 수 없기 때문이다. 

객체 생성 가능 클래스를 계승해서 새로운 값 컴포넌트를 추가할 만족스러운 방법이 없긴 하지만, 문제를 깔끔하게 피할 수 있는 방법은 하나 있다. **Point를 계승해서 ColorPoint를 만드는 대신, ColorPoint안에 private Point 필드를 두고, public 뷰(view) 메서드를 하나 만드는 것이다.** 이 뷰 메서드는 ColorPoint가 가리키는 위치를 Point 객체로 반환한다. 
```
//equals 규약을 위반하지 않으면서 값 컴포넌트 추가 
public class ColorPoint{
    private final Point point; 
    private final Color color;
    
    public ColorPoint(int x, int y, Color color){
        if(color == null)
            throw new NullPointerException();
        point = new Point(x, y);
        this.color = color; 
    }
    
    //ColorPoint의 Point 뷰 반환
    public Point asPoint(){
        return point;
    }
    
    @Override
    public boolean equals(Object o){
        if(!(o instanceof ColorPoint))
            return false; 
        
    }
}
```