# enum과 어노테이션
### 규칙34 : int 상수 대신 enum을 사용하라
```java
// int를 사용한 enum 패턴
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
위의 코드는 형안전성 측면에서도 그렇고, 편의성 관점에서도 단점이 많다. String enum 패턴이라 불리는 것은 더 나쁜 패턴이다.
상수 비교를 할 때 문자열 비교를 해야 하므로 성능이 떨어질 수 있고, 사용자가 필드 이름 대신 하드코딩된 문자열 상수를 클라이언트 코드 안에 박어버릴 수 있다는 점이다. 하드코딩된 문자열 상수에 오타가 있는 경우, 컴파일 할 때는 오류를 발견할 수 없기 때문에 실행 도중에 문제가 생기게 될 것이다.

자바 1.5부터 enum 자료형이 생겼다.
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
다른 언어들(C, C++, C#)의 enum은 int 값이지만 자바의 enum 자료형은 완전한 기능을 갖춘 클래스다.

enum자료형에 메서드나 필드를 추가하는 이유는 상수에 데이터를 연계시키면 좋기 때문이다. 풍부한 기능을 갖춘 enum 자료형 예제로, 태양계의
여덟 행성을 모델링하는 사례를 살펴보자.
```java
public enum Planet {
    MERCURY(3.33, 2.22),
    VENUS(2.22, 3.33),
    MARS(6.66, 7.77),
    URANUS(8.88,9.99);
	...

    private final double mass; // 킬로그램 단위
    private final double radius; // 미터단위
    private final double surfaceGravity;

	// 중력 상수
    private final double G = 6.67;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {return mass;}
    public double radius() {return radius;}
    public double surfaceGravity() {return surfaceGravity;}

    public double surfaceWeigt(double mass){
        return mass * surfaceGravity; // F = ma
    }
```
enum은 원래 변경 불가능하므로 모든 필드는 final로 선언되어야 한다. 필드는 public으로 선언할 수도 있지만, private로 선언하고 public 접근자를 두는 편이 더 낫다.

enum 자료형에는 자동 생성된 valueOf(String) 메서드가 있는데, 이 메서드는 상수의 이름을 상수 그 자체로 변환하는 역할을 한다. enum 자료형의 toString 메서드를 재정의 할 경우에는 fromString 메서드를 작성해서 toString이 뱉어내는 문자열을 다시 enum 상수로 변환할 수단을 제공해야 할지 생각해 봐야 한다.
```java
// enum 자료형에 대한 fromString 메서드 구현
private static final Map<String, Operation> stringToEnum = new HashMap<>();

static { // 상수 이름을 실제 상수로 대응시키는 맵 초기화
	for (Operation op : values())
		stringToEnum.put(op.toString(), op);
}

// 문자열이 주어지면 그에 대한 Operation 상수 반환. 잘못된 문자열이면 null 반환
public static Operation fromString(String symbol) {
	return stringToEnum.get(symbol);
}
```
Operation 상수를 stringToEnum 맵에 넣는 것은 상수가 만들어진 다음에 실행되는 static 블록 안에서 한다는 것에 주의하자. 각각의 상수가 생성자 안에서 맵에 자기 자신을 넣도록 하면 컴파일 할 때 오류가 발생한다. enum 생성자 안에서는 enum의 static 필드를 접근할 수 없다(컴파일 시점에 상수인 static 필드는 제외).
생성자가 실행될 때 static 필드는 초기화된 상태가 아니기 때문에 필요한 제약이다.

3rd Edition에서 추가된 부분
```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol) {
	return Optional.ofNullable(stringToEnum.get(symbol));
}
```

상수별 메서드 구현의 단점은 enum 상수끼리 공유하는 코드를 만들기가 어렵다는 것이다. 예를 들어, 급여 명세서에 찍히는 요일을 표현하는 enum 자료형이 있다고 하자.
이 enum 자료형 상수, 그러니까 요일을 나타내는 상수에는 직원의 시급과 해당 요일에 일한 시간을 인자로 주면 해당 요일의 급여를 계산하는 메서드가 있다.
그런데 주중에는 초과근무 시간에 대해서만 초과근무 수당을 주어야 하고, 주말에는 몇 시간을 일했건 전부 초과근무 수당으로 처리해야 한다.
switch 문을 만들 때 case 레이블을 경우에 따라 잘 붙이기만 하면 쉽게 원하는 계산을 할 수 있을 것이다.
```java
public enum PayrollDay {
	MONDAY,	TUESDAY, WEDNESDAY,	THURSDAY, FRIDAY, SATURDAY,	SUNDAY;
	private static final int HOURS_PER_SHIFT = 8;

	double pay(double hourWorked, double payRate) {
		double basePay = hourWorked * payRate;

		double overtimePay; // 초과근무수당 계산
		switch (this) {
			case SATURDAY: case SUNDAY:
				overtimePay = hourWorked * payRate /2;
				break;
			default:
				overtimePay = hourWorked <= HOURS_PER_SHIFT ? 0 : (hourWorked - HOURS_PER_SHIFT) * payRate / 2;
		}

		return basePay + overtimePay;
	}
}
```
분명 간결한 코드다. 하지만 유지보수 관점에서는 위험한 코드다. enum에 새로운 상수를 추가한다고 하자. 아마도 휴가 등을 나타내는 특별한 값일 것이다.
그런데 switch 문에 해당 상수에 대한 case를 추가하는 것을 잊었다면? 컴파일은 되겠지만 휴가 때 일한 시간에 대해서는 같은 급여를 지급하는
프로그램이 되어버릴 것이다.

정말 좋은 방법은 새로운 enum 상수를 추가할 때 초과근무 수당 계산 정책을 반드시 선택하도록 하는 것이다. 기본적인 아이디어는 초과근무 수당을
계산하는 부분을 private로 선언된 중첩 enum 자료형에 넣고, PayrollDay enum 생성자가 이 전략 enum 상수를 인자로 받게 하는 것이다.
PayrollDay enum 상수가 초과근무 수당 계산을 이 정책 enum 상수에 위임하도록 하면 switch문이나 상수별 메서드 구현은 없앨 수 있다.
이 패턴을 적용한 코드가 switch 문을 써서 만든 코드보다는 복잡하지만 안전할 뿐더러 유연성도 높다.
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
    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }

    // 정책 enum 자료형
    private enum PayType {
        WEEKDAY {
          double overtimePay(double hours, double payRate) {
              return hours <= HOURS_PER_SHIFT ? 0 : (hours - HOURS_PER_SHIFT) * payRate / 2;
          }
        },
        WEEKEND {
            double overtimePay(double hours, double payRate) {
                return hours * payRate / 2;
            }
        };

        private static final int HOURS_PER_SHIFT = 8;

        abstract double overtimePay(double hrs, double payRate);

        double pay(double hoursWorked, double payRate) {
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}
```

### 규칙35 : ordinal 대신 객체 필드를 사용하라
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
### 규칙36 : 비트 필드 대신 EnumSet을 사용하라
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

### 규칙37 : ordinal을 배열 첨자로 사용하는 대신 EnumMap을 이용하라
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

### 규칙38 : 확장 가능한 enum을 만들어야 한다면 인터페이스를 이용하라
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

### 규칙39 : (Prefer annotations to naming patterns)작명 패턴 대신 애노테이션을 사용하라
이번 예제는 Junit의 @Test 애노테이션 기능을 간단하게 직접 구현해보면서, 작명 패턴(naming pattern) 보다 애노테이션이 어떻게 더 좋은지를 설명한다.

작명 패턴의 예로 과거 JUnit은 테스트 메서드 이름을 test로 시작해야 했다.
이러한 작명 패턴에는 몇 가지 문제점이 있는데 첫째, 오타났을 때 프로그램 상 문제가 없기 때문에 알아차리기 어렵다.
둘째, 특정한 프로그램 요소에만 적용되도록 만들 수 없다. 예를 들어 testSafetyMechanisms라는 이름의 클래스를 만들었다 해도 그 클래스의 모든 메서드를 테스트 실행시키지 않는다(클래스 이름 까지는 확인하지 않기 때문에 의미가 없다).
셋째, 프로그램 요소에 인자를 전달할 마땅한 방법이 없다. 메서드 이름에 포함된 문자열로 예외를 알려주는 방법이 있지만 보기 흉할 뿐 아니라 컴파일러가 문자열이 예외 이름인지 알 도리가 없다.

그러므로 애노테이션을 사용하자.
```java
// 표식 애노테이션 자료형(markder annotation type) 선언
import java.lang.annotation.*;

/**
* 애노테이션이 붙은 메서드가 테스트 메서드임을 표시.
* 무인자 정적 메서드(parameterless)에만 사용 가능.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface BongTest {
}
```
애노테이션 자료형 BongTest 선언부에도 Retention과 Target이라는 애노테이션이 붙어 있다. 애노테이션 자료형 선언부에 붙는 애노테이션은 메타-애노테이션이라 부른다.
@Retention(RetentionPolicy.RUNTIME)은 BongTest가 실행시간(runtime)에도 유지되어야 하는 애노테이션이라는 뜻이다. 그렇지 않으면 BongTest는 테스트 도구에게는 보이지 않는다.
@Target(ElementType.METHOD)은 BongTest가 메서드 선언부에만 적용할 수 있는 애노테이션이라는 뜻이다.

```java
public class Sample {

	@BongTest
	public static void noParamStaticMethod() { // 성공해야함
	}

	@BongTest
	public static void oneParamMethod() { // 실패해야함
		throw new RuntimeException("Boom");
	}

	@BongTest
	public void noParamMethod() { // 실패해야함
	}

	@BongTest
    private void privateNoParamMethod() { // 실패해야함
    }

	@BongTest
	public static void oneParamStaticMethod(String ii) { // 실패해야함
	}
}
```
위와 같이 @BongTest 애노테이션을 적용한 메서드를 Sample 클래스에 선언해 놓고 테스트 실행기를 돌려보자.
@BongTest 애노테이션은 Sample 클래스가 동작하는 데 직접적 영향을 미치지 않는다. 해당 애노테이션에 관심 있는 프로그램에게 유용한 정보를 제공할 뿐이다.
```java
public class RunTests {
	public static void main(String[] args) throws Exception {
		int tests = 0;
		int passed = 0;
		Class testClass = Sample.class;
		for (Method m : testClass.getDeclaredMethods()) {
			if (m.isAnnotationPresent(BongTest.class)) {
				tests++;
				try {
					m.invoke(null);
					passed++;
				} catch (InvocationTargetException wrappedExc) {
					Throwable exc = wrappedExc.getCause();
					System.out.println(m + " failed:" + exc);
				} catch (Exception exc) {
					System.out.println("INVALID @BongTest" + m);
					System.out.println(exc);
				}
			}
		}

		System.out.println("Passed :" + passed);
		System.out.println("Failed :" + (tests - passed));
	}
}
```
이 테스트 실행기는 Sample 클래스의 메서드들 가운데 @BongTest 애노테이션이 붙은 메서드를 전부 찾아내서 리플렉션 기능을 활용해 실행한다(Method.invoke 호출).
isAnnotationPresent 메서드는 실행해야 하는 테스트 메서드를 찾는 용도로 사용되었다. 리플렉션을 통해 호출된 메서드가 예외를 발생시키면 해당 예외는
InvocationTargetException으로 wrapping된다. 이 예외가 아닌 다른 예외가 발생되었다면 그것은 컴파일 시에 발견하지 못한, 잘못 사용된 애노테이션이 있다는 뜻이다.
인스턴스 메서드나 private 메서드, 인자가 있는 메서드에 애노테이션을 붙이면 그런일이 생긴다.

이제 특정한 예외가 발생했을 경우만 성공하는 테스트도 지원 가능하도록 고쳐보자. 새로운 애노테이션 자료형이 필요하다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface BongTest {
	Class<? extends Exception> value() default BongTest.None.class;

	public static class None extends Exception {
		private None() {
		}
	}
}
```
추가로 None 클래스를 만들어 default로 놓음으로써 애노테이션의 인자가 없을 때 컴파일 에러가 발생하는것을 막았다.

```java
	@BongTest(ArithmeticException.class)
	public static void arithmeticExceptionTest() {
		int i = 0;
		i = i / i;
	}

	@BongTest(ArrayIndexOutOfBoundsException.class)
	public static void arrayIndexOutOfBoundsExceptionTest() {
		int[] a = new int[0];
		int i = a[1];
	}
```
위와 같이 발생할 예외를 인자로 보내주면 아래의 테스트 실행기에서 통과 됨을 확인할 수 있다.
```java
public class RunTests {
	public static void main(String[] args) throws Exception {
		int tests = 0;
		int passed = 0;
		Class testClass = Sample.class;
		for (Method m : testClass.getDeclaredMethods()) {
			if (m.isAnnotationPresent(BongTest.class)) {
				tests++;
				try {
					m.invoke(null);
					passed++;
				} catch (InvocationTargetException wrappedExc) {
					Throwable exc = wrappedExc.getCause();
					Class<? extends Exception> excType = m.getAnnotation(BongTest.class).value();

					if (excType.isInstance(exc))
						passed++;
					else
						System.out.println(m + " failed:" + exc);
				} catch (Exception exc) {
					System.out.println("INVALID @BongTest" + m);
					System.out.println(exc);
				}
			}
		}

		System.out.println("Passed :" + passed);
		System.out.println("Failed :" + (tests - passed));
	}
}
```

좀 더 발전 시켜서 지정된 예외들 가운데 하나라도 테스트 메서드 안에서 발생하면 테스트가 통과하도록 할 수도 있다.
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface BongTest {
	Class<? extends Exception>[] value() default BongTest.None.class;

	public static class None extends Exception {
		private None() {
		}
	}
}

@BongTest({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad() {
	List<String> list = new ArrayList<>();
	// 자바 명세에는 아래와 같이 addAll을 호출하면 IndexOutOfBoundsException이나 NullPointerException이 발생한다고 명시되어 있다.
	list.addAll(5, null);
}
```
```java
public class RunTests {
	public static void main(String[] args) throws Exception {
		int tests = 0;
		int passed = 0;
		Class testClass = Sample.class;
		for (Method m : testClass.getDeclaredMethods()) {
			if (m.isAnnotationPresent(BongTest.class)) {
				tests++;
				try {
					m.invoke(null);
					passed++;
				} catch (InvocationTargetException wrappedExc) {
					Throwable exc = wrappedExc.getCause();
					Class<? extends Exception>[] excTypes = m.getAnnotation(BongTest.class).value();

					for (Class<? extends Exception> excType : excTypes) {
						if (excType.isInstance(exc)) {
							passed++;
							break;
						}
					}

					System.out.println(m + " failed:" + exc);

				} catch (Exception exc) {
					System.out.println("INVALID @BongTest" + m);
					System.out.println(exc);
				}
			}
		}

		System.out.println("Passed :" + passed);
		System.out.println("Failed :" + (tests - passed));
	}
}
```
자바8부터 multivalued annotations 하는 또다른 방법이 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(BongTestContainer.class)
public @interface BongTest {
	Class<? extends Exception> value() default BongTest.None.class;

	public static class None extends Exception {
		private None() {
		}
	}
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface BongTestContainer {
	BongTest[] value();
}

@BongTest(NullPointerException.class)
@BongTest(IndexOutOfBoundsException.class)
public static void doublyBad() {
	List<String> list = new ArrayList<>();
	list.addAll(5, null);
}
```
@Repeatable 메타 애노테이션으로 단일 요소에 반복적으로 적용할 수 있다. containing annotation type 인자를 받고 그 containing annotation type은
annotation 배열 타입을 갖는다. 주의할 점은 containing annotation type도 반드시 retention 정책과 target에 대한 메타 애노테이션이 있어야 한다. 그렇지 않으면 컴파일이 안된다.

repeatable annotation을 처리하려면 주의가 필요하다. getAnnotationsByType 메서드는 repeated와 non-repeated 애노테이션에 접근하는데 모두 사용될 수 있다.
그러나 isAnnotationPresent 메서드는 BongTest 타입을 검사할 때 BongTestContainer 타입은 자동으로 무시한다. 마찬가지로 BongTestContainer 타입을 검사할 때도 BongTest
타입은 무시한다. 그래서 아래와 같이 두개의 타입 모두를 검사해줘야 한다.
```java
public class RunTests {
	public static void main(String[] args) throws Exception {
		int tests = 0;
		int passed = 0;
		Class testClass = Sample.class;
		for (Method m : testClass.getDeclaredMethods()) {
			if (m.isAnnotationPresent(BongTest.class) || m.isAnnotationPresent(BongTestContainer.class)) {
				tests++;
				try {
					m.invoke(null);
					passed++;
				} catch (InvocationTargetException wrappedExc) {
					Throwable exc = wrappedExc.getCause();
					BongTest[] excTests = m.getAnnotationsByType(BongTest.class);
					for (BongTest excTest : excTests) {
						if (excTest.value().isInstance(exc)) {
							passed++;
							break;
						}
					}

					System.out.println(m + " failed:" + exc);

				} catch (Exception exc) {
					System.out.println("INVALID @BongTest" + m);
					System.out.println(exc);
				}
			}
		}

		System.out.println("Passed :" + passed);
		System.out.println("Failed :" + (tests - passed));
	}
}
```
Repeatable 애노테이션은 가독성을 향상시키지만, 애노테이션을 처리하는데 더 많은 상용구(boilerplate)가 있으며 처리하는데 오류를 발생시키기 쉽다.

### 규칙 40 : Override 애노테이션은 일관되게 사용하라
상위 클래스에 선언된 메서드를 재정의할 때는 반드시 선언부에 Override 애노테이션을 붙여라. 그래야 실수 했을 때 컴파일러에서 검출될 수 있다.

그런데 비-abstract 클래스에서 abstract 메서드를 재정의할 때는 Override 애노테이션을 붙이지 않아도 된다(상위 클래스 메서드를 재정의한다는 사실을 명시적으로 표현하고 싶다면 붙여도 상관 없다).

버전 1.6 이상의 자바를 사용한다면 Override 애노테이션을 통해 찾을 수 있는 버그는 더 많다. 클래스 뿐 아니라 
인터페이스에 선언된 메서드를 구현할 때도 Override를 사용할 수 있게 되었기 때문이다. 하지만 인터페이스를 구현할 때 모든 메서드에 반드시 Override를 붙여야 하는 것은 아니다. 인터페이스에 선언된 메서드를 재정의 하지 않으면 어차피 컴파일러가 오류를 내기 때문이다. (마찬가지로 특정 인터페이스 메서드를 재정의하는 메서드라는 사실을 명시적으로 알리고 싶다면 애노테이션을 붙여도 되나, 반드시 필요한 것은 아니다).

### 규칙 41 : 자료형을 정의할 때 표식 인터페이스를 사용하라
표식 인터페이스(marker interface)는 아무 메서드도 선언하지 않는 인터페이스다. Serializable 인터페이스가 그 예다. 
```java
public interface Serializable {
}
```
이 인터페이스를 구현하는 클래스를 만든다는 것은, 해당 클래스로 만든 객체들은 ObjectOutputStream으로 출력할 수 있다는(“직렬화”할 수 있다는) 뜻이다. 다시 말해 해당 클래스가 어떤 속성을 만족한다는 사실을 표시하는 것과 같다. 

표식 애노테이션과 비교했을 때 표식 인터페이스는 두 가지 장점이 있다. 첫 번째 장점은, **표식 인터페이스는 결국 표식 붙은 클래스가 만드는 객체들이 구현하는 자료형이라는 점이다.** 표식 애노테이션은 자료형이 아니다. 표식 인터페이스는 자료형이므로, 표식 애노테이션을 쓴다면 프로그램 실행 중에나 발견하게 될 오류를 컴파일 시점에 발견할 수 있도록 한다. 표식 인터페이스 Serializable의 경우를 살펴보자. ObjectOutputStream.write(Object) 메서드는 인자가 Serializable 인터페이스를 구현하지 않은 객체면 오류를 낸다. 두 번째 장점은, 적용 범위를 좀 더 세밀하게 지정할 수 있다는 것이다. 애노테이션 자료형을 선언할 때 target을 ElementType.TYPE으로 지정하면 해당 애노테이션은 어떤 클래스나 인터페이스에도 적용 가능하다. 그런데 특정한 인터페이스를 구현한 클래스에만 적용할 수 있어야 하는 표식이 필요하다고 해 보자. 표식 인터페이스를 쓴다면, 그 특정 인터페이스를 extends 하도록 선언하기만 하면 된다.

표식 애노테이션의 주된 장점은 프로그램 안에서 애노테이션 자료형을 쓰기 시작한 뒤에도 더 많은 정보를 추가할 수 있다는 것이다. 기본값(default)을 갖는 애노테이션 자료형 요소들을 더해 나가면 된다. 표식 인터페이스를 쓰는 경우에는 이런 진화가 불가능하다. 일단 구현이 이루어지고 난 다음에는 새로운 메서드를 추가하는 것이 일반적으로 불가능하기 때문이다(자바8부터 default 메서드를 통해 불가능하지는 않음).

그렇다면 표식 애노테이션과 표식 인터페이스는 각각 어떤 상황에 걸맞나? 클래스나 인터페이스 이외의 프로그램 요소에 적용되어야 하는 표식은 애노테이션으로 만들어야 한다. 하지만 만약 표식이 붙은 객체만 인자로 받을 수 있는 메서드를 만든다면 표식 인터페이스를 사용해야 한다. 그러면 해당 메서드의 인자 자료형으로 해당 인터페이스를 사용할 수 있어서, 컴파일 시간에 형 검사를 진행할 수 있게 된다. 요약하자면, 표식 인터페이스와 표식 애노테이션은 쓰임새가 다르다. 새로운 메서드가 없는 자료형을 정의하고자 한다면 표식 인터페이스를 이용해야 한다. 클래스나 인터페이스 이외의 프로그램 요소에 표식을 달아야 하고, 앞으로 표식에 더 많은 정보를 추가할 가능성이 있다면, 표식 애노테이션을 사용해야 한다. 