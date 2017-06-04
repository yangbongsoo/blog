## Story06 static 제대로 한번 써 보자
자바에서 static으로 지정했다면, 해당 메서드나 변수는 정적이다.
```java
public class VariableTypes {
	int instance Variable;
	static int classVariable;
	public void method(int parameter) {
		int localVariable;
	}
}
```
여기서 static으로 선언한 classVariable은 클래스 변수라고 한다. 왜냐하면 그 변수는 '객체의 변수'가 되는 것이 아니라 '클래스의 변수'가 되기 때문이다. 100개의 VariableTypes 클래스의 인스턴스를 생성하더라도, 모든 객체가 classVariable에 대해서는 동일한 주소의 값을 참조한다.

static 초기화 블록에 대해서도 다시한번 알아보자.
```java
public class StaticTest {

    static String staticVal;
    static {
        staticVal = "Static Value";
        staticVal = StaticTest2.staticInt + "";
    }

    public static void main(String[] args) {
        System.out.println(StaticTest.staticVal);
    }

    static {
        staticVal = "Performance is important !!!";
    }
}
```
static 초기화 블록은 위와 같이 클래스 어느 곳에나 지정할 수 있다. 이 static 블록은 클래스가 최초 로딩될 때 수행되므로 생성자 실행과 상관없이 수행된다. 또한 여러 번 사용할 수 있으며, 이와 같이 사용했을 때 staticVal 값은 마지막에 지정한 값이 된다. static 블록은 순차적으로 읽혀진다는 의미이다.

static의 특징은 다른 JVM 에서는 static이라고 선언해도 다른 주소나 다른 값을 참조하지만, 하나의 JVM이나 WAS 인스턴스에서는 같은 주소에 존재하는 값을 참조한다는 것이다. 그리고 GC의 대상도 되지 않는다. 그러므로 static을 잘 사용하면 성능을 뛰어나게 향상시킬 수 있지만, 잘못 사용하면 예기치 못한 결과를 초래하게 된다.

특히 웹 환경에서 static을 잘못 사용하다가는 여러 쓰레드에서 하나의 변수에 접근할 수도 있기 때문에 데이터가 꼬이는 큰 일이 발생할 수도 있다.
###static 잘못 쓰면 이렇게 된다
```java
public class BadQueryManager {
    private static String queryURL = null;

    public BadQueryManager(String badUrl) {
        queryURL = badUrl;
    }
    
    public static String getSql(String idSql) {
        try {
            FileReader reader = new FileReader();
            HashMap<String, String> document = reader.read(queryURL);
            return document.get(idSql);
        } catch (Exception ex) {
            System.out.println(ex);
        }
        return null;
    }
}
```
만약 어떤 화면에서 BadQueryManager의 생성자를  통해서 queryURL을 설정하고, getSql을 호출하기 전에, 다른 queryURL을 사용하는 화면의 스레드에서 BadQueryManager의 생성자를 호출하면 어떤 일이 발생할까? 그때부터는 시스템이 오류를 발생시킨다.

## Story07 클래스 정보, 어떻게 알아낼 수 있나?
reflection 관련 클래스를 어떻게 사용해야 하는지 간단한 예를 통해서 살펴보자.
```java
public class DemoClass {
    private String privateField;
    String field;
    protected String protectedField;
    public String publicField;

    public DemoClass() {}
    public DemoClass(String arg) {}

    public void publicMethod() throws IOException, Exception {}

    public String publicMethod(String s, int i) {
        return "s="+s+ "i ="+i;
    }

    protected void protectedMethod() {}

    private void privateMethod() {}

    void method() {}

    public String publicRetMethod() {
        return null;
    }

    public InnerClass getInnerClass() {
        return new InnerClass();
    }

    public class InnerClass {
    }
}
```
```java
public class DemoTest {
    public static void main(String[] args) {
        DemoClass dc = new DemoClass(); // 점검 대상 클래스 객체
        DemoTest dt = new DemoTest();
        dt.getClassInfos(dc);
    }

    public void getClassInfos(Object clazz) {
        Class demoClass = clazz.getClass();
        getClassInfo(demoClass);
    }

    public void getClassInfo(Class demoClass) {
        String className = demoClass.getName();
        System.out.format("Class Name : %s \n", className);
        String classCanonicalName = demoClass.getCanonicalName();
        System.out.format("Class Canonical Name : %s \n", classCanonicalName);
        String classSimpleName = demoClass.getSimpleName();
        System.out.format("Class Simple Name : %s \n", classSimpleName);
        String packageName = demoClass.getPackage().getName();
        System.out.format("Package Name : %s \n", packageName);
        String toString = demoClass.toString();
        System.out.format("toString : %s \n", toString);
    }
}
```
```
Class Name : org.sample.DemoClass 
Class Canonical Name : org.sample.DemoClass 
Class Simple Name : DemoClass 
Package Name : org.sample 
toString : class org.sample.DemoClass 
```
이 코드는 클래스 정보만을 가져오는 부분이다. 이제 필드 정보를 읽는 부분을 보자.
```java
    public void getFieldInfo(Class demoClass) {
        Field[] field1 = demoClass.getDeclaredFields();
        Field[] field2 = demoClass.getFields();
        System.out.format("Declared Fields : %d, Fields: %d\n", field1.length, field2.length);

        for (Field field : field1) {
            String fieldName = field.getName();
            int modifier = field.getModifiers();
            String modifierStr = Modifier.toString(modifier);
            String type = field.getType().getSimpleName();
            System.out.format("%s %s %s \n", modifierStr, type, fieldName);
        }
    }
```
```
Declared Fields : 4, Fields: 1
private String privateField 
 String field 
protected String protectedField 
public String publicField 
```
여기서 가장 어려운 부분은 식별자 데이터를 가져오는 부분이다. getModifiers() 메서드에서는 int 타입으로 리턴을 하기 때문에 간단하게 변환을 하기가 어렵다. 그에 대비해서 Modifier 클래스에 static으로 선언되어 있는 Modifier.toString() 메서드가 있다. 이 메서드에 int 타입의 값을 보내면 식별자 정보를 문자열로 리턴한다. 

이제 메서드 정보를 가져오는 부분을 보자.
```java
    public void getMethodInfo(Class demoClass) {
        Method[] method1 = demoClass.getDeclaredMethods();
        Method[] method2 = demoClass.getMethods();
        System.out.format("Declared methods : %d, Methods : %d\n", method1.length, method2.length);

        for (Method met1 : method1) {
            // method name info
            String methodName = met1.getName();
            // method modifier info
            int modifier = met1.getModifiers();
            String modifierStr = Modifier.toString(modifier);
            // method return type info
            String returnType = met1.getReturnType().getSimpleName();
            // method parameter info
            Class params[] = met1.getParameterTypes();
            StringBuilder paramStr = new StringBuilder();
            int paramLen = params.length;
            if (paramLen != 0) {
                paramStr.append(params[0].getSimpleName()).append(" arg");
                for (int loop = 1; loop < paramLen; loop++) {
                    paramStr.append(",")
                            .append(params[loop].getName())
                            .append(" arg")
                            .append(loop);
                }
            }

            // method exception info
            Class exceptions[] = met1.getExceptionTypes();
            StringBuilder exceptionStr = new StringBuilder();
            int exceptionLen = exceptions.length;
            if (exceptionLen != 0) {
                exceptionStr.append("throws").append(exceptions[0].getSimpleName());
                for (int loop = 1; loop < exceptionLen; loop++) {
                    exceptionStr.append(",")
                            .append(exceptions[loop].getSimpleName());
                }
            }

            // print result
            System.out.format("%s %s %s(%s) %s\n", modifierStr, returnType, methodName, paramStr, exceptionStr);
        }
    }
```
```
Declared methods : 7, Methods : 13
public String publicMethod(String arg,int arg1) 
public void publicMethod() throwsIOException,Exception
protected void protectedMethod() 
private void privateMethod() 
public String publicRetMethod() 
public InnerClass getInnerClass() 
 void method() 
```
메서드 가져오는 부분에서 중요한 것은 예외와 매개변수를 처리하는 부분이다. 이 두 가지 데이터는 일반적으로 하나가 아니기 때문에 위와 같이 반복하면서 해당 부분의 정보를 읽어 와야 한다.

###reflection 클래스를 잘못 사용한 사례
```java
public String checkClass(Object src) {
	if (src.getClass().getName().equals("java.math.BigDecimal")) {
		// 데이터 처리
	}

	// 이하 생략
}
```
이렇게 사용할 경우 응답 속도에 그리 많은 영향을 주지는 않지만, 많이 사용하면 필요 없는 시간을 낭비하게 된다. getClass() 메서드를 호출할 때 Class 객체를 만들고, 그 객체의 이름을 가져오는 메서드를 수행하는 시간과 메모리를 사용한다.
```java
public String checkClass(Object src) {
	if (src instance java.math.DigDecimal) {
		// 데이터 처리
	}

	// 이하 생략
}
```
이러한 부분에서 개선이 필요할 때는 자바의 기본으로 돌아가자.

JMH를 이용하여 얼마나 성능 차이가 있는지 비교해 보자.
```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class Reflection {

    int LOOP_COUNT = 100;
    String result;

    @Benchmark
    public void withEquals() {
        Object src = new BigDecimal("6");
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            if (src.getClass().getName().equals("java.math.BigDecimal")) {
                result = "BigDecimal";
            }
        }
    }

    @Benchmark
    public void withInstanceof() {
        Object src = new BigDecimal("6");
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            if (src instanceof java.math.BigDecimal) {
                result = "BigDecimal";
            }
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(Reflection.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```
# Run complete. Total time: 00:00:17

Benchmark                  Mode  Cnt  Score   Error  Units
Reflection.withEquals      avgt    5  0.276 ± 0.024  us/op
Reflection.withInstanceof  avgt    5  0.028 ± 0.006  us/op
```
큰 차이는 발생하지 않지만, 이런 부분이 모여 큰 차이를 만들기 때문에 작은 것부터 생각하면서 코딩하는 습관을 가지는 것이 좋다. 추가로 클래스의 메타 데이터 정보는 
JVM의 Perm 영역에 저장된다는 사실을 기억해 주기 바란다. 만약 Class 클래스를 사용하여 엄청나게 많은 클래스를 동적으로 생성하는 일이 벌어지면 Perm 영역이 더 이상 
사용할 수 없게 되어 OutOfMemoryError가 발생할 수도 있으니, 조심해서 사용하자.