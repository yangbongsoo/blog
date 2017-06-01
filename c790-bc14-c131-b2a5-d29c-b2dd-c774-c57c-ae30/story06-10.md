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


