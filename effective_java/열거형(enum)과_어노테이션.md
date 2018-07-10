# 열거형(enum)과 어노테이션
### 규칙34 : int 상수 대신 enum을 사용하라
enum자료형에 메서드나 필드를 추가하는 이유는 상수에 데이터를 연계시키면 좋기 때문이다. 
```java
public enum Planet {
    MERCURY(3.33, 2.22),
    VENUS(2.22, 3.33),
    MARS(6.66, 7.77),
    URANUS(8.88,9.99);

    private final double mass;
    private final double radius;
    private final double G = 6.67;
    private final double surfaceGravity;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {return mass;}
    public double radius() {return radius;}
    public double surfaceGravity() {return surfaceGravity;}

    public double surfaceWeigt(double mass){
        return mass * surfaceGravity;
    }
```
enum은 원래 변경 불가능하므로 모든 필드는 final로 선언되어야 한다.
enum생성자 안에서는 enum의 static 필드를 접근할 수 없다(컴파일 시점에 상수인 static 필드는 제외).
```java
public enum PayrollDay {

    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    //Constructor
    PayrollDay(PayType payType){
        this.payType = payType;
    }

    double pay(double hoursWorked, double payRate){
        return payType.pay(hoursWorked, payRate);
    }

    // 정책 enum 자료형
    private enum PayType{
        WEEKDAY{
          double overtimePay(double hours, double payRate){
              return hours <= HOURS_PER_SHIFT ? 0 : (hours - HOURS_PER_SHIFT) * payRate / 2;
          }
        },
        WEEKEND{
            double overtimePay(double hours, double payRate){
                return hours * payRate /2;
            }
        };

        private static final int HOURS_PER_SHIFT = 8;

        abstract double overtimePay(double hrs, double payRate);

        double pay(double hoursWorked, double payRate){
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked,payRate);
        }
    }
}
```

### 규칙31 : ordinal 대신 객체 필드를 사용하라 
```java
//ordinal을 남용한 사례 
public enum Ensemble{
    SOLO, DUET, TRIO;
    
    public int numberOfMusicians(){
        return ordinal() + 1;
    }
}
```
모든 enum에는 ordinal이라는 메서드가 있는데, enum 자료형 안에서 enum 상수의 위치를 나타내는 정수값을 반환한다. 하지만 객체필드를 사용해라 
```java
public enum Ensemble{
    SOLO(1), DUET(2), TRIO(3);
    
    private final int num;
    
    public Ensemble(int size){
        this.num = size;
    }
    
    public int numberOfMusicians(){
        return num; 
    }
}
```
### 규칙32 : 비트 필드 대신 EnumSet을 사용하라 
```java
//비트 필드 열거형 상수 - 이제는 피해야 할 구현법
public class Text{
    public static final int STYLE_BOLD          = 1 << 0; //1
    public static final int STYLE_ITALIC        = 1 << 1; //2
    public static final int STYLE_UNDERLINE     = 1 << 2 //4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; //8
    
    //이 메서드의 인자는 STYLE_상수를 비트별 OR한 값
    public void applyStyles(int styles) { ... } 
}
```
`text.applyStyles(STYLE_BOLD | STYLE_ITALIC);` 이렇게 하면 상수들을 집합에 넣을 때 비트별 OR 연산을 사용할 수 있다. 하지만 EnumSet 이라는 더 좋은 방법이 있다. 
```java
//EnumSet - 비트필드를 대신할 현대적 기술
public class Text{
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHROUGH
    }
    
    //어떤 Set 객체도 인자로 전달할 수 있으나, EnumSet이 분명 최선 
    public void applyStyles(Set<Style> styles){ ... }
}
```
`text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));`
EnumSet의 단점이 하나 있는데 변경 불가능 EnumSet객체를 만들 수 없다. 그래서 EnumSet 객체를 Collections.unmodifiableSet으로 포장하면 되는데, 성능이나 코드 가독성 측면에서 좀 손해를 보게 된다. 

### 규칙33 : ordinal을 배열 첨자로 사용하는 대신 EnumMap을 이용하라 
```java
class Herb{
	enum Type { ANNUAL, PERENNIAL, BIENNIAL }

	final String name;
	final Type type;

	Herb(String name, Type type){
		this.name = name;
		this.type = type;
	}

	@Override
	public String toString(){
		return name; 
	}
}
```
```java
//EnumMap을 사용해  enum 상수별 데이터를 저장하는 프로그램
Herb[] garden = …; 

Map<Herb.Type, Set<Herb>> herbsByType =
	new EnumMap<Herb.Type, Set<Herb>>(Herb.Type.class);

for(Herb.Type t : Herb.Type.values())
	herbsByType.put(t, new HashSet<Herb>());

for(Herb h : garden)
	herbsByType.get(h.type).add(h);

System.out.println(herbsByType);
```
EnumMap 생성자가 키의 자료형을 나타내는 Class 객체를 인자로 받는다는 것에 주의하자. 이런 Class 객체를 한정적 자료형 토큰이라고 부르는데, 실행시점 제네릭 자료형 정보를 제공한다.

두 번째 예제는 상전이(phase transition) 관계를 표현하기 위해서 중첩 EnumMap을 사용했다. 
```java
// EnumMap을 중첩해서 enum 쌍에 대응되는 데이터를 저장한다
public enum Phase{
	SOLID, LIQUID, GAS;

	public enum Transition{
		MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
		BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
		SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

		private final Phase src;
		private final Phase dat;

		Transition(Phase src, Phase dst){
			this.src = src;
			this.dst = dat;
		}

		//상 전이 맵 초기화 
		private static final Map<Phase, Map<Phase, Transition>> m =
			new EnumMap<Phase, Map<Phase, Transition>>(Phase.class);
		static{
			for(Phase p : Phase.values())
				m.put(p, new EnumMap<Phase, Transition>(Phase.class));

			for(Transition trans : Transition.values())
				m.get(trans.src).put(trans.dst, trans);
		}

		public static Transition from(Phase src, Phase dat) {
			return m.get(src).get(dst);
		}
	}
}
```
![](/assets/EnumMapexample.jpg)

LIQUID쪽을 보면 액체 LIQUID에서 고체 SOLID로 변하는 것은 언다FREEZE라고 한다. 이 맵의 자료형은 `Map<Phase, Map<Phase, Transition>>`인데, “상전이 이전 상태를, 상전이 이후 상태와 상전이 명칭 사이의 관계를 나타내는 맵에 대응시키는 맵”이라는 뜻이다. 

### 규칙34 : 확장 가능한 enum을 만들어야 한다면 인터페이스를 이용하라
일반적으로 enum 자료형을 계승한다는 것은 바람직하지 않다. 확장된 자료형의 상수들이 기본 자료형의 상수가 될 수 있지만 그 반대가 될 수 없다는 것은 혼란스럽기 때문이다. 또한 기본 자료형과 그 모든 하위 자료형의 enum 상수들을 순차적으로 살펴볼 좋은 방법도 없고 설계와 구현에 관계된 많은 부분이 까다로워진다. 

**하지만 열거 자료형의 확장이 가능하면 좋은 경우가 적어도 하나 있다. 연산 코드(opcode)를 만들어야 할 때다.** 연산 코드는 어떤 기계에서 사용되는 연산을 표현하기 위해 쓰이는 열거 자료형이다. 기본 아이디어는 enum 자료형이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하는 것이다.

먼저 연산 코드 자료형에 대한 인터페이스를 정의한다. 그리고 해당 인터페이스를 구현하는 enum 자료형을 만든다. 
```java
// 인터페이스를 이용해 확장 가능하게 만든 enum 자료형 
public interface Operation {
	double apply(double x, double y);
}

public enum BasicOperation implements Operation { 
	PLUS(“+”) {
		public double apply(double x, double y) { return x + y; }
	},
	MINUS(“-“) {
		public double apply(double x, double y) { return x - y; }
	},
	TIMES(“*“) {
 		public double apply(double x, double y) { return x * y; }
	},
	DIVIDE(“/“) {
 		public double apply(double x, double y) { return x / y; }
	};

	private final String symbol;

	BasicOperation(String symbol) {
		this.symbol = symbol;
	}

	@Override public String toString(){
		return symbol; 
	}
}
```
BasicOperation은 enum 자료형이라 계승할 수 없지만 Operation은 인터페이스가 확장이 가능하다. 따라서 이 인터페이스를 계승하는 새로운 enum 자료형을 만들면 Operation 객체가 필요한 곳에 해당 enum 자료형의 상수를 이용할 수 있게 된다.
```java
// 인터페이스를 이용해 기존 enum 자료형을 확장하고 테스트하는 프로그램
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	// Operation을 상속한ExtendedOperation이라는 enum을 새롭게 만든껏임. P224 
	test(ExtendedOperation.class, x, y); 
}

private static <T extends Enum<T> & Operation> void test( Class<T> opSet, double x, double y){
	for (Operation op : opSet.getEnumConstants())
		System.out.printf(“%f %s %f = %f%n”, x, op, y, op.apply(x, y));
}
```
확장된 연산을 나타내는 자료형의 class 리터럴인 ExtendedOperation.class가 main에서 test로 전달되고 있음에 유의하자. 확장된 연산 집합이 무엇인지 알리기 위한 것이다. 이 class 리터럴은 한정적 자료형 토큰 구실을 한다. opSet의 형인자 T는 굉장히 복잡하게 선언되어 있는데 Class 객체가 나타내는 자료형이 enum 자료형인 동시에 Operation의 하위 자료형이 되도록 한다 라는 뜻이다. 모든 enum 상수를 순차적으로 살펴보면서 해당 상수가 나타내는 연산을 실제로 수행할 수 있으려면 반드시 그래야 한다.

두 번째 방법은 한정적 와일드카드 자료형 `Collection<? extends Operation>`을 opSet 인자의 자료형으로 사용하는 것이다.
```java
public static void main(String[] args) {
double x = Double.parseDouble(args[0]);
double y = Double.parseDouble(args[1]);
test(Arrays.asList(ExtendedOperation.values()), x, y); 
}

private static void test(Collection<? extends Operation> opSet, double x, double y){
	for(Operation op : opSet) {
		System.out.printf(“%f %s %f = %f%n”, x, op, y, op.apply(x, y));
	}
}
```
test 메서드의 인자 형태는 메서드를 호출할 때, 여러 enum 자료형에 정의한 연산들을 함께 전달할 수 있도록 하기 위한 것이다. 그러나 이렇게 하면 EnumSet이나 EnumMap을 사용할 수 없기 때문에, 여러 자료형에 정의한 연산들을 함께 전달할 수 있도록 하는 유연성이 필요 없다면, 첫 번째 방식인 한정적 자료형 토큰을 쓰는게 낫다.

인터페이스를 사용해 확장 가능한 enum 자료형을 만드는 방법에는 한 가지 사소한 문제가 있다. enum 구현 자체는 계승할 수 없다는 것이다. 

### 규칙35 : 작명 패턴 대신 애노테이션을 사용하라 
작명 패턴의 예로 JUnit에서는 테스트 메서드 이름을 test로 시작해야 했다. 이러한 작명 패턴에는 몇 가지 문제점이 있는데 첫째, 오타났을 때 프로그램 상 문제가 없기 때문에 알아차리기 어렵다. 둘째, 특정한 프로그램 요소에만 적용되도록 만들 수 없다. 예를 들어 testSafetyMechanisms라는 이름의 클래스를 만들었다 해도 클래스 이름 까지는 확인하지 않기 때문에 의미가 없다. 셋째, 프로그램 요소에 인자를 전달할 마땅한 방법이 없다. 예를 들어 특정 예외가 발생해야 성공으로 판정하는 테스트에서 메서드 이름에 포함된 문자열로 예외를 알려주는 방법이 있다. 그러나 보기 흉할 뿐 아니라 컴파일러가 문자열이 예외 이름인지 알 도리가 없다.

애노테이션을 사용하자 
```java
// 표식 애노테이션 자료형(markder annotation type) 선언
import java.lang.annotation.*;

/**
*애노테이션이 붙은 메서드가 테스트 메서드임을 표시.
*무인자 정적 메서드에만 사용 가능.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test { 
}
```
애노테이션 자료형 Test 선언부에도 Retention과 Target이라는 애노테이션이 붙어 있다. 애노테이션 자료형 선언부에 붙는 애노테이션은 메타-애노테이션이라 부른다. @Retention(RetentionPolicy.RUNTIME)은 Test가 실행시간(runtime)에도 유지되어야 하는 애노테이션이라는 뜻이다. 그렇지 않으면 Test는 테스트 도구에게는 보이지 않는다. @Target(ElementType.METHOD)은 Test가 메서드 선언부에만 적용할 수 있는 애노테이션이라는 뜻이다.

이 책에서는 Test 애노테이션을 '무인자 정적 메서드’에만 사용 가능하다고 설명한다. static 메서드가 아니면 잘못됐다고 한다. P229

특정한 예외가 발생했을 경우만 성공하는 테스트도 지원 가능하도록 고쳐보자. 새로운 애노테이션이 자료형이 필요하다. 
```java
// 인자를 취하는 애노테이션 자료형 
import java.lang.annotation.*;

/**
*이 애노테이션이 붙은 메서드는 테스트 메서드이며,
*테스트에 성공하려면 지정된 예외를 발생시켜야 한다. 
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest { 
	Class<? extends Exception> value();
}
```
이 애노테이션에 사용할 수 있는 인자의 자료형은 `Class<? extends Exception>`이다. “Exception을 계승한 클래스에 대한 Class 객체”라고 풀이할 수 있겠다. Class 객체를 이렇게 사용하는 용법을 한정적 자료형 토큰이라고 한다. 
```java
// 인자를 받는 애노테이션의 사용 예제
@ExceptionTest(ArithmeticException.class)
public static void m1() { // 이 테스트는 성공 해야 함
	int i = 0;
	i = i / i; 
}
```
좀 더 발전 시키면, 지정된 예외들 가운데 하나라도 테스트 메서드 안에서 발생하면 테스트가 통과하도록 할 수도 있다. 
```java
// 배열을 인자로 받는 애노테이션 자료형
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest { 
	Class<? extends Exception>[] value();
}

//배열을 인자로 받는 애노테이션 사용 예
@ExceptionTest({ IndexOutofBoundsException.class, NullPointerException.class})
public static void doublyBad(){
	List<String> list = new ArrayList<String>();

	// 자바 명세에는 아래와 같이 addAll을 호출하면 IndexOutofBoundsException이나NullPointerException이 발생한다고 명시되어 있다.
	list.addAll(5, null);
}
```

### 규칙36 : Override 애노테이션은 일관되게 사용하라 
상위 클래스에 선언된 메서드를 재정의할 때는 반드시 선언부에 Override 애노테이션을 붙여라. 그래야 실수 했을 때 컴파일러에서 검출될 수 있다.

그런데 비-abstract 클래스에서 abstract 메서드를 재정의할 때는 Override 애노테이션을 붙이지 않아도 된다(상위 클래스 메서드를 재정의한다는 사실을 명시적으로 표현하고 싶다면 붙여도 상관 없다).

버전 1.6 이상의 자바를 사용한다면 Override 애노테이션을 통해 찾을 수 있는 버그는 더 많다. 클래스 뿐 아니라 
인터페이스에 선언된 메서드를 구현할 때도 Override를 사용할 수 있게 되었기 때문이다. 하지만 인터페이스를 구현할 때 모든 메서드에 반드시 Override를 붙여야 하는 것은 아니다. 인터페이스에 선언된 메서드를 재정의 하지 않으면 어차피 컴파일러가 오류를 내기 때문이다. (마찬가지로 특정 인터페이스 메서드를 재정의하는 메서드라는 사실을 명시적으로 알리고 싶다면 애노테이션을 붙여도 되나, 반드시 필요한 것은 아니다).

### 규칙 37 : 자료형을 정의할 때 표식 인터페이스를 사용하라 
표식 인터페이스(marker interface)는 아무 메서드도 선언하지 않는 인터페이스다. Serializable 인터페이스가 그 예다. 
```java
public interface Serializable {
}
```
이 인터페이스를 구현하는 클래스를 만든다는 것은, 해당 클래스로 만든 객체들은 ObjectOutputStream으로 출력할 수 있다는(“직렬화”할 수 있다는) 뜻이다. 다시 말해 해당 클래스가 어떤 속성을 만족한다는 사실을 표시하는 것과 같다. 

표식 애노테이션과 비교했을 때 표식 인터페이스는 두 가지 장점이 있다. 첫 번째 장점은, **표식 인터페이스는 결국 표식 붙은 클래스가 만드는 객체들이 구현하는 자료형이라는 점이다.** 표식 애노테이션은 자료형이 아니다. 표식 인터페이스는 자료형이므로, 표식 애노테이션을 쓴다면 프로그램 실행 중에나 발견하게 될 오류를 컴파일 시점에 발견할 수 있도록 한다. 표식 인터페이스 Serializable의 경우를 살펴보자. ObjectOutputStream.write(Object) 메서드는 인자가 Serializable 인터페이스를 구현하지 않은 객체면 오류를 낸다. 두 번째 장점은, 적용 범위를 좀 더 세밀하게 지정할 수 있다는 것이다. 애노테이션 자료형을 선언할 때 target을 ElementType.TYPE으로 지정하면 해당 애노테이션은 어떤 클래스나 인터페이스에도 적용 가능하다. 그런데 특정한 인터페이스를 구현한 클래스에만 적용할 수 있어야 하는 표식이 필요하다고 해 보자. 표식 인터페이스를 쓴다면, 그 특정 인터페이스를 extends 하도록 선언하기만 하면 된다.

표식 애노테이션의 주된 장점은 프로그램 안에서 애노테이션 자료형을 쓰기 시작한 뒤에도 더 많은 정보를 추가할 수 있다는 것이다. 기본값(default)을 갖는 애노테이션 자료형 요소들을 더해 나가면 된다. 표식 인터페이스를 쓰는 경우에는 이런 진화가 불가능하다. 일단 구현이 이루어지고 난 다음에는 새로운 메서드를 추가하는 것이 일반적으로 불가능하기 때문이다(자바8부터 default 메서드를 통해 불가능하지는 않음).

그렇다면 표식 애노테이션과 표식 인터페이스는 각각 어떤 상황에 걸맞나? 클래스나 인터페이스 이외의 프로그램 요소에 적용되어야 하는 표식은 애노테이션으로 만들어야 한다. 하지만 만약 표식이 붙은 객체만 인자로 받을 수 있는 메서드를 만든다면 표식 인터페이스를 사용해야 한다. 그러면 해당 메서드의 인자 자료형으로 해당 인터페이스를 사용할 수 있어서, 컴파일 시간에 형 검사를 진행할 수 있게 된다. 요약하자면, 표식 인터페이스와 표식 애노테이션은 쓰임새가 다르다. 새로운 메서드가 없는 자료형을 정의하고자 한다면 표식 인터페이스를 이용해야 한다. 클래스나 인터페이스 이외의 프로그램 요소에 표식을 달아야 하고, 앞으로 표식에 더 많은 정보를 추가할 가능성이 있다면, 표식 애노테이션을 사용해야 한다. 