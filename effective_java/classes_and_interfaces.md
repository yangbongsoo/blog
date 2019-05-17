# 클래스와 인터페이스

### 규칙15 : 클래스와 멤버의 접근 권한은 최소화하라
각 클래스와 멤버는 가능한 한 접근 불가능하도록 만들어라. 정보은닉 또는 캡슐화는 시스템을 구성하는 모듈 사이의 의존성을 낮춰서 개발 속도 및 유지보수에 효율적이다.

**클래스나 인터페이스 또는 멤버를 패키지의 공개 API로 만들어서는 곤란하다.**

**객체 필드(instance field)는 절대로 public으로 선언하면 안된다.** 변경 가능 public 필드를 가진 클래스는 다중 스레드에 안전하지 않다. 변경 불가능 객체를 참조하는 final 필드라 해도 public으로 선언하면 클래스 내부 데이터 표현 형태를 유연하게 바꿀 수 없게 된다. 공개 API의 일부가 되어버리므로, 삭제하거나 수정할 수 없게 되는 것이다. 

static으로 선언된 필드에도 똑같이 적용되지만 한 가지 예외가 있다. 어떤 상수들이 클래스로 추상화된 결과물의 핵심적 부분을 구성한다고 판단되는 경우, 해당 상수들을 `public static final` 필드들로 선언하여 공개할 수 있다. 
(public static final 필드가 참조하는 객체는 변경 불가능 객체로 만들어라.) 

길이가 0 아닌 배열은 언제나 변경 가능하므로, public static final 배열 필드를 두거나, 배열 필드를 반환하는 접근자를 정의하면 안 된다. 그런 멤버를 두면 클라이언트가 배열 내용을 변경할 수 있게 되므로, 보안에 문제가 생긴다. 
```java
//보안 문제를 초래할 수 있는 코드 
public static final Thing[] VALUES { ... };
```
이 문제를 고치는 방법 중 첫 번째는 public으로 선언되었던 배열은 private으로 바꾸고, 변경이 불가능한 public 리스트를 하나 만드는 것이다. 
```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUE));

```
두 번째 방법은 배열은 private으로 선언하고, 해당 배열을 복사해서 반환하는 public 메서드를 하나 추가하는 것이다.
```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values(){
    return PRIVATE_VALUES.clone();
}
```

### 규칙16 : public 클래스 안에는 public 필드를 두지 말고 접근자 메서드를 사용하라
cf) package-private(default) 클래스나 private 중첩클래스는 데이터 필드를 공개하더라도 잘못이라 말할 수 없다. 클래스가 추상화하려는 내용을 제대로 기술하기만 한다면 말이다. 

이 접근법은 시각적으로 깔끔해 보인다. 클라이언트 코드가 클래스의 내부 표현에 종속된다는 문제가 있긴 하지만, 클라이언트 코드가 같은 패키지 안에 있을 수밖에 없다는 점을 고려해야 한다. 

### 규칙17 : 변경 가능성을 최소화하라
**변경 가능한 클래스로 만들 타당한 이유가 없다면, 반드시 변경 불가능 클래스로 만들어야 한다.** 

자바 플랫폼 라이브러리중 String, 기본 자료형 클래스 BigInteger, BigDecimal 등은 변경 불가능 클래스여서 그 객체를 수정할 수 없다. 

변경 불가능 클래스를 만드는 이유는 설계하기 쉽고, 구현하기 쉬우며, 사용하기도 쉽다. 

**변경 불가능 클래스 만드는 5가지 규칙**
1. 객체 상태를 변경하는 메서드(setter 등)를 제공하지 않는다. 
2. 계승할 수 없도록 한다. (계승을 금지하려면 보통 클래스를 final로 선언) 
3. 모든 필드를 final로 선언한다. 
4. 모든 필드를 private으로 선언한다. 
5. 변경 가능 컴포넌트에 대한 독점적 접근권을 보장한다. 

다음 코드를 보자
```java
public final class Complex{
    private final double re;
    private final double im;
    
    public Complex(double re, double im){
        this.re = re;
        this.im = im;
    }
    
    //수정자는 없고 접근자만 있다. 
    public Complex add(Complex c){
        return new Complex(re + c.re, im + c.im);
    }
    
    public Complex subtract(Complex c){
        return new Complex(re - c.re, im - c.im);
    }
    
    ...
}
```
**사칙연산 각각은 this 객체를 변경하는 대신 새로운 Complex 객체를 만들어 반환하도록 구현되어 있음에 유의해라. 대부분의 변경 불가능 클래스가 따르는 패턴이다.**

함수형 접근법으로도 알려져 있는데 피연산자를 변경하는 대신, 연산을 적용한 결과를 새롭게 만들어 반환하기 때문이다. 생성될 때 부여된 한 가지 상태만 갖는다. 따라서 변경 불가능 객체는 단순하다. 

또한 변경 불가능 객체는 스레드에 안전할 수 밖에 없다. 어떤 동기화도 필요 없으며, 변경 불가능한 객체는 자유롭게 공유 할 수 있다. 그렇게 하는 한 가지 쉬운 방법은, 자주 사용되는 값을 public static final 상수로 만들어 제공하는 것이다. 

한 단계 더 개선하자면, 자주 사용하는 객체를 캐시하여 정적 팩터리(규칙1)를 제공할 수 있다. 이런 정적 팩토리 메서드를 사용하면 클라이언트는 새로운 객체를 만드는 대신 기존 객체들을 공유하게 되므로 메모리 요구량과 쓰레기 수집 비용이 줄어든다.

변경 불가능한 객체를 자유롭게 공유할 수 있다는 것은, 방어적 복사본을 만들 필요없고 clone 메서드나 복사 생성자 또한 만들 필요도 없다. 

**변경 불가능 객체의 유일한 단점은 값마다 별도의 객체를 만들어야 한다는 점이다.**
ex) String 클래스
```java
String s1 = "a";
String s2 = "b"; 
String s3 = s1 + s2; //새로운 객체 생성
```
변경 가능한 public 동료 클래스를 제공하는 것이 최선이다. `StringBuilder`

하위 클래스 정의가 불가능하도록 하려면 보통 클래스를 final로 선언하면 되지만, 그보다 더 유연한 방법도 있다. 모든 생성자를 private나 package-private로 선언하고 public 생성자 대신 public 정적 팩토리를 제공하는 것이다. 
```java
//생성자 대신 정적 팩토리 메서드를 제공하는 변경 불가능 클래스
public class Complex{
    private final double re;
    private final double im;
    
    //private 생성자
    private Complex(double re, double im){
        this.re = re;
        this.im = im;
    }
    
    //정적 팩토리 메서드 
    public static Complex valueOf(double re, double im){
        return new Complex(re, im); 
    }
    
    ...
}
```
추가적으로 한가지 더 주의할 것은 직렬화에 관계된 부분이다. 변경 불가능 클래스가 Serializable 인터페이스를 구현하도록 했고, 해당 클래스에 변경 가능 객체를 참조하는 필드가 있다면, readObject 메서드나 readResolve 메서드를 반드시 제공해야 한다. 아니면 ObjectOutputStream.writeUnshared나 ObjectInputStream.readInshared 메서드를 반드시 사용해야 한다. 

### 규칙18 : Favor composition over inheritance(계승 대신 구성하라)
이 책에서 inheritance라는 용어는 extends 한다는 소리다. 어떤 클래스가 다른 인터페이스를 implements 하거나 어떤 인터페이스가 다른
인터페이스를 extends하는 경우에는 적용되지 않는다는 얘기다.

일반적인 객체 생성 가능 클래스라면, 해당 클래스가 속한 패키지 밖에서 계승을 시도하는 것은 위험하다.

메서드 호출과 달리, 계승은 캡슐화 원칙을 위반한다. 하위 클래스가 정상 동작하기 위해서는 상위 클래스의 구현에 의존할 수밖에 없다.
상위 클래스의 구현은 릴리즈가 거듭되면서 바뀔 수 있는데 그러다 보면 하위 클래스 코드는 수정된 적이 없어도 망가질 수 있다.

구체적인 사례를 보자. HashSet 객체가 생성된 이후 얼마나 많은 요소가 추가 되었는지를 질의한다고 했을 때,
계승을 이용해 HashSet에 삽입된 요소의 수를 추적하는 필드와 그 필드에 대한 접근자를 갖는 클래스를 만들었다.
HashSet 클래스에는 원소를 추가하는 데 쓰이는 add와 addAll이라는 메서드가 있으므로, 그 두 메서드를 재정의하였다.
```java
//계승을 잘못 사용한 사례
class InstrumentedHashSet<E> extends HashSet<E> {
	//요소를 삽입하려 한 횟수
	private int addCount = 0;

	//생성자
	public InstrumentedHashSet() {
	}

	public InstrumentedHashSet(int initCap, float loadFactor) {
		super(initCap, loadFactor);
	}

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}

public class TestMain {
	public static void main(String[] args) {
		InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
		s.addAll(List.of("Snap","Crackle","Pop")); // - third edition
		// s.addAll(Arrays.asList("1","2","3")); - second edition
	}
}
```
cf) Note that we create a list using the static factory method `List.of`, which was added in Java 9

위 코드를 실행하면 gettAddCount 메서드는 3이 아닌 6을 반환한다. **HashSet의 addAll 메서드는 add 메서드를 통해 구현되어 있기 때문이다.**
addAll 메서드에서 super.addAll을 호출하면, InstrumentedHashSet에서 재정의한 add 메서드를 삽입할 원소마다 호출한다.
결국 중복해서 카운트가 진행하게 되는것이다.

```java
	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		try {
			for (E next : c) {
				add(next);
			}
			return true;
		} catch (Exception e) {
			return false;
		}
	}

	public int getAddCount() {
		return addCount;
	}
```
addAll 메서드가 인자에 담긴 각 원소마다 add를 호출하도록 변경하면 조금 더 낫긴 할 것이다. 그렇게 하면 HashSet의 addAll 메서드가
add를 이용하는지는 상관없이 올바른 결과가 나올 것이다. HashSet의 addAll을 호출하지 않기 때문이다.

하지만 이 기법이 모든 문제를 해결하는 것은 아니다. 결국 자기 메서드를 호출 할 수도 있고 그렇지 않을 수도 있는 상위 클래스 메서드를
수정하는 것인데, 어려울 뿐 아니라 시간도 많이 걸리고, 오류가 나기도 쉬운 작업이다. 또한 이 기법은 항상 사용할 수 있는 것은 아니다.
어떤 메서드는 private 필드를 사용하지 않으면 구현될 수 없는 경우도 있기 때문이다(하위 클래스에서 접근 못함).

이와 관련해서 하위 클래스 구현을 망가뜨릴 수 있는 또 한 가지 요인은, 다음 릴리스에는 상위 클래스에 새로운 메서드가 추가될 수 있다는 것이다.
새 릴리스에 추가된 상위 클래스 메서드가 재수 없게도 하위 클래스에 정의한 메서드와 같은 시그너처(signature)인데 반환값 자료형만 다를 경우,
우리가 만든 하위 클래스는 더 이상 컴파일 되지 않을 것이다.

다행히도 지금껏 설명한 모든 문제를 피할 방법이 있다. 기존 클래스를 계승하는 대신, 새로운 클래스에 기존 클래스 객체를 참조하는 private 필드를
하나 두는 것이다. 이런 설계 기법을 구성(composition) 이라고 부르는데, 기존 클래스가 새 클래스의 일부(component)가 되기 때문이다.
```java
class InstrumentedSet<E> extends ForwardingSet<E> {
	private int addCount = 0;

	public InstrumentedSet(Set<E> s) {
		super(s);
	}

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}

class ForwardingSet<E> implements Set<E> {
	private final Set<E> s;
	public ForwardingSet(Set<E> s) {this.s = s;}
	@Override
	public int size() {	return s.size();}
	@Override
	public boolean isEmpty() {return s.isEmpty();}
	@Override
	public boolean contains(Object o) {return s.contains(o);}
	@Override
	public Iterator<E> iterator() {return s.iterator();}
	@Override
	public Object[] toArray() {return s.toArray();}
	@Override
	public <T> T[] toArray(T[] a) {return s.toArray(a);}
	@Override
	public boolean add(E e) {return s.add(e);}
	@Override
	public boolean remove(Object o) {return s.remove(o);}
	@Override
	public boolean containsAll(Collection<?> c) {return s.containsAll(c);}
	@Override
	public boolean addAll(Collection<? extends E> c) {return s.addAll(c);}
	@Override
	public boolean retainAll(Collection<?> c) {return s.retainAll(c);}
	@Override
	public boolean removeAll(Collection<?> c) {return s.removeAll(c);}
	@Override
	public void clear() {s.clear();}
}

public class TestMain {
	public static void main(String[] args) {
		InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>(10));
		s.addAll(Arrays.asList("1", "2", "3"));
		System.out.println(s.getAddCount());
	}
}
```
기존 코드를 유지하고, extends만 ForwardingSet으로 바꿔주면 정상적으로 동작한다. 그런데 기존에 내가 생각했던 구성(composition)은
아래 코드다.
```java
class InstrumentedSet<E> {
	private Set<E> set;
	private int addCount = 0;

	public InstrumentedSet(Set<E> set) {
		this.set = set;
	}

	public boolean add(E e) {
		addCount++;
		return set.add(e);
	}

	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return set.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}

public class TestMain {
	public static void main(String[] args) {
		InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>(10));
		s.addAll(Arrays.asList("1", "2", "3"));
		System.out.println(s.getAddCount());
	}
}
```
포장(wrapper) 클래스에는 단점이 별로 없으나, 콜백 프레임워크와 함께 사용하기에는 적합하지 않다. 콜백 프레임워크에서 객체는
자기 자신에 대한 참조를 다른 객체에 넘겨, 나중에 필요할 때 콜백하도록 요청한다. 포장된(wrapped) 객체는 포장 객체에 대해서는 모르기 때문에,
자기 자신에 대한 참조(this)를 전달할 것이다. 따라서 역호출(callback) 과정에서 포장 객체는 제외된다.

계승은 하위 클래스가 상위 클래스의 하위 자료형(subtype)이 활실한 경우에만 바람직하다. 만일 A를 계승하고 싶다면,
B는 확실히 A인가? 질문에 확실하게 그렇다고 답할 수 있어야 한다. 자바 플랫폼 라이브러리에는 이 원칙을 위반하는 사례들이 많다.
예를 들어 스택(stack)은 벡터(vector)가 아니므로 Stack은 Vector를 계승하면 안 된다. 마찬가지로 속성 리스트(property list)는
해시 테이블이 아니므로 Properties는 Hashtable을 계승하면 안 된다. 모두 구성 기법이 더 적절하다.

구성 기법에 적절한 곳에 계승을 사용하면 구현 세부사항이 쓸데없이 노출된다. 더 심각한 이슈는 클라이언트가 내부 구현 세부사항에
접근할 수 있다는 것이다. 예를 들어 아래와 같이 Properties 객체에 대한 참조 변수 p가 있을 때
```java
Properties p = new Properties();
p.getProperty("ybs"); // properties
p.get("ybs"); // hashtable
```
두 개의 메서드는 다른 결과를 반환할 수도 있다. `p.getProperty("ybs");`는 기본값(default)를 고려하지만, Hashtable에서
계승된 두 번째 메서드는 그렇지 않기 때문이다. 가장 심각한 문제는 클라이언트가 상위 클래스 상태를 직접 변경하여 하위 클래스의 불변식을
깰 수 있다는 것이다. 상위 클래스 Hashtable에 직접 접근해서 상태를 변경시키면 Properties 클래스의 불변식은 깨질 수 있다.

구성 대신 계승을 사용하려 할 때 반드시 물어야 할 마지막 질문들은 이런 것이다. 계승할 클래스의 API에 문제가 있는가? 그렇다면,
그 문제들이 계속 새 API의 일부가 되어도 상관없겠는가? 계승 메커니즘은 상위클래스의 문제를 하위 클래스에 전파시킨다. 반면 구성 기법은
그런 약점을 감추는 새로운 API를 설계할 수 있도록 해 준다.

### 규칙19 : 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라
계승을 허용하려면 첫째, 생성자는 직접적이건 간접적이건 재정의 가능 메서드를 호출해서는 안된다.
```java
public class Super{
    //생성자가 재정의 가능 메서드를 호출하는 잘못된 사례
    public Super(){
        overrideMe();
    }
    
    public void overrideMe(){ }
}

public final class Sub extends Super{

    private final Date date; 
    
    Sub(){
        date = new Date();
    }
    
    //상위 클래스 생성자가 호출하게 되는 재정의 메서드 
    @Override public void overrideMe(){
        System.out.println(date);
    }
    
    public static void main(String[] args){
        Sub sub = new Sub();
        sub.overrideMe(); 
    }
}
```
상위 클래스 생성자는 하위 클래스 생성자보다 먼저 실행되므로, 하위 클래스에서 재정의한 overrideMe 메서드는 하위 클래스 생성자가 실행되기 전에 호출될 것이다. 그러므로 하위 클래스 생성자에서 date 객체를 생성하지만 그  전에 상위 클래스 생성자에서 overrideMe 메서드를 호출했고, date 객체는 생성 전이기 때문에 null을 출력하게 된다. 

**계승을 반드시 허용해야 한다고 느껴지면, 재정의 가능 메서드는 절대로 호출하지 않도록 하고 그 사실을 반드시 문서에 남겨라.**

### 규칙20 : 추상 클래스 대신 인터페이스를 사용하라
인터페이스는 믹스인(mixin)을 정의하는 데 이상적이다. 믹스인은 클래스가 주 자료형(primary type)이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다. 예를 들어 Comparable은 어떤 클래스가 자기 객체는 다른 객체와의 비교 결과에 따른 순서를 갖는다고 선언할 때 쓰는 믹스인 인터페이스다. 

이런 인터페이스를 믹스인이라 부르는 것은, 자료형의 주된 기능에 선택적인 기능을 "혼합(mix in)"할 수 있도록 하기 때문이다. 

추상 골격 구현 클래스를 중요 인터페이스마다 두면, 인터페이스의 장점과 추상 클래스의 장점을 결합 할 수 있다. 예를 들어, 컬렉션 프레임워크에는 인터페이스별로 골격 구현 클래스들이 하나씩 제공된다. (AbstractCollection, AbstractSet, AbstractList, AbstractMap)

골격 구현 클래스가 있다면 해당 클래스를 사용해 인터페이스를 구현하는 것이 가장 분명한 프로그래밍 방법이다. (골격 구현 클래스는 계승 용도로 설계하는 클래스이다.)

추상 클래스가 인터페이스보다 나은 점이 한 가지 있는데, 인터페이스는 추상 클래스가 발전시키기 쉽다는 것이다. 새로운 메서드를 추가하고 싶다면 적당한 기본 구현 코드를 담은 메서드를 언제든 추가할 수 있고 해당 추상 클래스를 계승하는 모든 클래스는 그 즉시 새로운 메서드를 제공하게 될 것이다. 일반적으로 인터페이스를 구현하는 기존 클래스를 깨뜨리지 않고 새로운 메서드를 인터페이스에 추가할 방법은 없다.(자바 1.8부터는 'default' 메서드를 통해 기존 클래스를 깨뜨리지 않고 새 메서드를 추가할 수 있다.)

### item21 : Design interfaces for posterity

### 규칙22 : 인터페이스는 자료형을 정의할 때만 사용하라
인터페이스를 구현하는 클래스를 만들게 되면, 그 인터페이스는 해당 클래스의 객체를 참조할 수 있는 자료형(type) 역할을 하게 된다. 인터페이스를 구현해 클래스를 만든다는 것은, 해당 클래스의 객체로 어떤 일을 할 수 있는지 클라이언트에게 알리는 행위다. 다른 목적은 적절치 못하다. (상수 인터페이스, 메서드가 없고 static final 필드만 있는 형태) 

### 규칙23 : Prefer class hierarchies to tagged classes(태그 달린 클래스 대신 클래스 계층을 활용하라)
때로 우리는 두 가지 이상의 기능을 가지고 있으며, 그 중 어떤 기능을 제공하는지 표시하는 태그(tag)가 달린 클래스를 만날 때가 있다.
```java
class Figure {
	enum Shape {RECTANGLE, CIRCLE};

	// 어떤 모양인지 나타내는 태그 필드
	final Shape shape;

	// 태그가 RECTANGLE일 때만 사용되는 필드들
	double length;
	double width;

	// 태그가 CIRCLE일 때만 사용되는 필드들
	double radius;

	// 원을 만드는 생성자
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	// 사각형을 만드는 생성자
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}

	double area() {
		switch (shape) {
			case CIRCLE:
				return length * width;

			case RECTANGLE:
				return Math.PI * (radius * radius);

			default:
				throw new AssertionError();
		}
	}
}
```
태그 달린 클래스에는 다양한 문제가 있다. enum 선언, 태그 필드, switch 문 등의 상투적인(boilerplate) 코드가 반복되는 클래스가 만들어지며, 서로 다른 기능을 위한 코드가 한 클래스에 모여 있으니 가독성도 떨어진다.

그래서 아래 코드와 같이 클래스 계층을 활용하면 단순 명료하며, 상투적인 코드도 없다.
```java
abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;

	public Circle(double radius) {
		this.radius = radius;
	}

	@Override
	double area() {
		return Math.PI * (radius * radius);
	}
}

class Rectangle extends Figure {
	final double length;
	final double width;

	public Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}

	@Override
	double area() {
		return length * width;
	}
}
```
클래스 계층의 또 다른 장점은 자료형 간의 자연스러운 계층 관계를 반영할 수 있어서 유연성이 높아지고 컴파일 시에 형 검사(type checking)를 하기 용이하다는 것이다.
그리고 태그 기반 클래스 대신 클래스 계층을 이용하면 정사각형이 특별한 유형의 사각형임을 표현할 수 있다(사각형과 정사각형이 전부 변경 불가능 객체를 만드는 클래스라고 가정).
```java
class Square extends Rectangle {

	public Square(double length) {
		super(length, length);
	}

	@Override
	double area() {
		return length * length;
	}
}
```


한 클래스에 여러 기능이 있어서 태그로써 구분하는 방법쓰지 말고 계층 형태로 분리시켜라.

### 규칙24 : 멤버 클래스는 가능하면 static으로 선언하라
중첩 클래스는 다른 클래스 안에 정의된 클래스다. 
**1.정적(static) 멤버 클래스**

    1. 정적 멤버 클래스를 제외한 나머지는 전부 내부 클래스다. 정적(static) 멤버 클래스는 어쩌다 다른 클래스 안에 선언된 일반 클래스라고 생각해도 된다. 정적 멤버 클래스는 바깥 클래스의 모든 멤버에(private로 선언된 것까지도) 접근할 수 있다. 정적 멤버 클래스를 private로 선언했다면 해당 중첩 클래스에 접근할 수 있는 것은 바깥 클래스뿐일 것이다.
    2. 중첩된 클래스의 객체가 바깥 클래스 객체와 독립적으로 존재할 수 있도록 하려면 중첩 클래스는 반드시 정적 멤버 클래스로 선언해야 한다.
    3. private 정적 멤버 클래스는 바깥 클래스 객체의 컴포넌트를 표현하는 데 흔히 쓰인다. 예를 들어 키와 값을 연관 짓는 Map 객체를 생각해 보자. Map을 구현하는 많은 클래스는 내부적으로 키-값 쌍을 보관하는 Entry 객체를 사용한다. 각 Entry 객체는 특정 맵에 속할 것이지만, Entry 객체의 메서드(getKey, getValue, setValue 등)는 맵 객체에 접근할 필요가 없다. 따라서 Entry를 비-정적 멤버 클래스로 표현하는 것은 낭비다. private 정적 멤버 클래스가 최선이다. 실수로 static 키워드를 빼먹어도 맵은 여전히 동작할 것이지만, 각 Entry 객체 안에는 바깥 객체, 그러니까 Map을 가리키는 참조가 생겨서 공간과 시간이 낭비될 것이다.
    
**2.비-정적 멤버 클래스**

    1. 비-정적 멤버 클래스 객체는 바깥 클래스 객체와 자동적으로 연결된다. 비-정적 멤버 클래스 안에서는 바깥 클래스의 메서드를 호출할 수도 있고, this 한정 구문을 통해 바깥 객체에 대한 참조를 획득할 수도 있다.  
    2. 비-정적 멤버 클래스의 객체는 바깥 클래스 객체 없이는 존재할 수 없다.
    3. 비-정적 멤버 클래스 객체와 바깥 객체와의 연결은 비-정적 멤버 클래스의 객체가 만들어지는 순간에 확립되고, 그 뒤에는 변경할 수 없다.
    4. 비-정적 멤버 클래스는 어댑터를 정의할 때 많이 쓰인다. 바깥 클래스 객체를 다른 클래스 객체인 것처럼 보이게 하는 용도다. 예를 들어 Map 인터페이스를 구현하는 클래스들은 비-정적 멤버 클래스를 사용해 컬렉션 뷰를 구현한다.
     
**3.익명 클래스**

    1. 멤버로 선언하지 않으며, 사용하는 순간에 선언하고 객체를 만든다. 익명 클래스는 비-정적 문맥 안에서 사용될 때만 바깥 객체를 갖는다. 그러나 정적 문맥 안에서 사용된다 해도 static 멤버를 가질 수는 없다.
     
**4.지역 클래스**

1. 네 종류의 중첩 클래스 가운데 사용 빈도가 가장 낮다. 지역 변수가 선언될 수 있는 곳이라면 어디서든 선언가능하다. 익명 클래스처럼 비-정적 문맥에서 정의했을 때만 바깥 객체를 갖는다. 

**바깥 클래스 객체에 접근할 필요가 없는 멤버 클래스를 정의할 때는 항상 선언문 앞에 static을 붙여서 비-정적 멤버 클래스대신 정적 멤버 클래스로 만들자.**<br>
static을 생략하면 모든 객체는 내부적으로 바깥 객체에 대한 참조를 유지하게 된다. 그 덕분에 시간과 공간 요구량이 늘어나며, 바깥 객체에 대한 쓰레기 수집이 힘들어진다. 비-정적 멤버 클래스객체는 바깥 객체 없이는 생성할 수 없다는 문제도 있다. 

멤버 클래스가 API 클래스의 public이나 protected 멤버인 경우에는, 정적 멤버 클래스로 만들 것인지 아니면 비-정적 멤버 클래스로 만들 것인지가 더욱 중요하다. 일단 API에 포함되고 나면, 이진 호환성을 깨지 않고는 다음 릴리스에서 비-정적 멤버 클래스를 정적 멤버 클래스로 바꿀 방법이 없다.

### item 25 : Limit source files to a single top-level class