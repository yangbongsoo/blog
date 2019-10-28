# 클린아키텍처

## 2부 벽돌부터 시작하기: 프로그래밍 패러다임
함수형 프로그래밍 패러다임은 구조적 프로그래밍, 객체 지향 프로그래밍보다도 먼저
만들어졌지만 최근에 들어서야 겨우 도입되기 시작했다(사실 함수형 프로그래밍은 컴퓨터 프로그래밍 자체보다 먼저 등장했다).

과학은 근본적으로 수학과는 다른데, 과학 이론과 법칙은 그 올바름을 절대 증명할 수 없기 때문이다.
과학적 방법은 반증은 가능하지만 증명은 불가능하다.

과학은 서술된 내용이 사실임을 증명하는 방식이 아니라 서술이 틀렸음을 증명하는 방식으로 동작한다.
각고의 노력으로도 반례를 들 수 없는 서술이 있다면 목표에 부합할 만큼은 참이라고 본다.

다익스트라는 "테스트는 버그가 있음을 보여줄 뿐, 버그가 없음을 보여줄 수는 없다"고 말한 적이 있다.
다시 말해 프로그램이 잘못되었음을 테스트를 통해 증명할 수 는 있지만, 프로그램이 맞다고 증명할 수는 없다.

## 3부 설계원칙
### SRP 단일책임 원칙
단일 모듈은 변경의 이유가 하나, 오직 하나뿐이어야 한다.

![](/assets/srp5.png)

Employee 클래스는 SRP를 위반하는데, 이들 세 가지 메서드가 서로 매우 다른 세 명의 액터를 책임지기 때문이다.
calculatePay 메서드는 CFO, reportHours 메서드는 COO, save 메서드는 CTO를 위해 사용된다.

이 결합으로 CFO 팀에서 결정한 조치가 COO 팀이 의존하는 무언가에 영향을 줄 수 있다.
예를 들어 calculatePay 메서드와 reportHours 메서드가 공통적으로 regularHours 메서드를 사용한다고 해보자.

![](/assets/srp6.png)

만약 CFO 팀에서 업무 시간을 계산하는 방식을 수정하여 regularHours 로직이 변경된다면 COO팀이 사용하는
reportHours 로직에 문제가 생기게 된다.

이러한 문제는 서로 다른 액터가 의존하는 코드를 너무 가까이 배치했기 때문에 발생한다. SRP는 서로 다른 액터가
의존하는 코드를 서로 분리하라고 말한다.

해결책은 다양하다. 가장 확실한 방법은 데이터와 메서드를 분리하는 방식이다.
즉 아무런 메서드가 없는 간단한 데이터 구조인 EmployeeData 클래스를 만들어, 세 개의 클래스가 공유하도록 한다.

![](/assets/srp7.png)

```java
public class SrpMain {
	public static void main(String[] args) {
		Employee employee = new Employee();
		employee.calculatePay();
		employee.reportHours();
		employee.save();

		EmployeeData employeeData = new EmployeeData(1,"YangBongSoo");
		PayCalculator payCalculator = new PayCalculator(employeeData);
		HourReporter hourReporter = new HourReporter(employeeData);
		EmployeeSaver employeeSaver = new EmployeeSaver(employeeData);
	}
}
```
```java
public class EmployeeData {

	private int id;
	private String name;

	public EmployeeData(int id, String name) {
		this.id = id;
		this.name = name;
	}

	public int regulrHours() {
		return 8;
	}
}

public class PayCalculator {

	private EmployeeData employeeData;

	public PayCalculator(EmployeeData employeeData) {
		this.employeeData = employeeData;
	}

	public int calculatePay() {
		return employeeData.regulrHours() * 1000;
	}
}

public class HourReporter {

	private EmployeeData employeeData;

	public HourReporter(EmployeeData employeeData) {
		this.employeeData = employeeData;
	}



	public int reportHours() {
		return employeeData.regulrHours() + 1;
	}
}

public class EmployeeSaver {

	private EmployeeData employeeData;

	public EmployeeSaver(EmployeeData employeeData) {
		this.employeeData = employeeData;
	}

	public void saveEmployee() {

	}
}
```
반면 이 해결책은 개발자가 세 가지 클래스를 인스턴스화하고 추적해야 한다는 게 단점이다. 이러한 난관에서 빠져나올 때
흔히 쓰는 기법으로 Facade 패턴이 있다.

![](/assets/srp8.png)

```java
public class SrpMain {
	public static void main(String[] args) {
		EmployeeData employeeData = new EmployeeData(1,"YBS");
		EmployeeFacade employeeFacade = new EmployeeFacade(employeeData);
		employeeFacade.calculatePay();
		employeeFacade.reportHours();
		employeeFacade.save();
	}
}

public class EmployeeFacade {

	private EmployeeData employeeData;

	public EmployeeFacade(EmployeeData employeeData) {
		this.employeeData = employeeData;
	}

	public int calculatePay() {
		return new PayCalculator(employeeData).calculatePay();
	}

	public int reportHours() {
		return new HourReporter(employeeData).reportHours();
	}

	public void save() {
		new EmployeeSaver(employeeData).saveEmployee();
	}
}
```

EmployeeFacade에 코드는 거의 없다. 이 클래스는 세 클래스의 객체를 생성하고, 요청된 메서드를 가지는 객체로 위임하는 일을 책임진다.

### OCP 개방 폐쇄 원칙

![](/assets/ocp1.png)

여기서 주목할 점은 FinancialDataMapper는 구현 관계를 통해 FinancialDataGateway를 알고 있지만, FinancialDataGateway는 FinancialDataMapper에 
대해 아무것도 알지 못한다. 그러므로 FinancialDataMapper에서 발생한 변경으로부터 FinancialReportGenerator를 보호하려면 FinancialDataMapper가 
FinancialDataGateway에 의존해야 하고 FinancialReportGenerator는 FinancialDataGateway를 사용해야 한다.

FinancialDataGateway 인터페이스가 없었다면, Database 관련 의존성(low level detail)이 비지니스 로직을 담당하는 
FinancialReportGenerator(high level policy)로 바로 향하게 된다.

### LSP 리스코프 치환 원칙

![](/assets/lsp1.png)

이 설계는 LSP를 준수하는데, Billing 애플리케이션의 행위가 License 하위 타입 중 무엇을 사용하는지에 전혀 의존하지 않기 때문이다. 이들 하위 타입은
모두 License 타입을 치환할 수 있다.

정사각형/직사각형 문제는 '객체지향과 디자인 패턴' 에서 자세하게 정리했다.
https://yangbongsoo.gitbook.io/study/oo_and_design_patterns

### ISP 인터페이스 분리 원칙

![](/assets/isp1.png)

위의 그림에서 다수의 사용자가 OPS 클래스의 오퍼레이션을 사용한다. User1은 오직 op1을, User2는 op2만을, User3는 op3만을 사용한다고 가정해 보자.
그리고 OPS가 정적 타입 언어로 작성된 클래스라고 해보자. 이경우 User1에서는 op2, op3를 전혀 사용하지 않음에도 User1의 소스 코드는 이 두 메서드에 의존하게 된다.
이러한 의존성으로 인해 OPS 클래스에서 op2의 소스 코드가 변경되면 User1도 다시 컴파일 한 후 새로 배포해야 한다.

이러한 문제는 오퍼레이션을 인터페이스 단위로 분리하여 해결할 수 있다.

![](/assets/isp2.png)

User1의 소스 코드는 U1Ops와 op1에는 의존하지만 OPS에는 의존하지 않게 된다. 따라서 OPS에서 발생한 변경이 User1과는 전혀 관계없는 변경이라면,
User1을 다시 컴파일하고 새로 배포하는 상황은 초래되지 않는다.

필요 이상으로 많은 걸 포함하는 모듈에 의존하는 것은 해로운 일이다. 소스 코드 의존성의 경우 이는 분명한 사실인데, 불필요한 재컴파일과 재배포를 강제하기 때문이다.

### DIP 의존성 역전 원칙

DIP에서 말하는, 유연성이 극대화된 시스템이란 소스 코드 의존성이 추상(abstraction)에 의존하며 구체(concretion)에는 의존하지 않는 시스템이다.
하지만 이 아이디어를 규칙으로 보기는 확실히 비현실적이다. String 클래스는 구체 클래스이며 이를 애써 추상 클래스로 만들려는 시도는 현실성이 없다.
java.lang.String 구체 클래스에 대한 소스 코드 의존성은 벗어날 수 없고, 벗어나서도 안된다.

이러한 이유로 DIP를 논할 때 운영체제나 플랫폼 같이 안정성이 보장된 환경에 대해서는 무시하는 편이다. 우리가 의존하지 않도록 피하고자 하는 것은 바로 변동성이 큰
구체적인 요소다(개발하는 중이라 자주 변경될 수밖에 없는 모듈들).

안정된 소프트웨어 아키텍처란 변동성이 큰 구현체에 의존하는 일은 지양하고, 안정된 추상 인터페이스를 선호하는 아키텍처라는 뜻이다.
이 원칙에서 전달하려는 내용은 다음과 같이 매우 구체적인 코딩 실천법으로 요약할 수 있다.

**변동성이 큰 구체 클래스를 참조하지 말라**
대신 추상 인터페이스를 참조하라. 또한 이 규칙은 객체 생성 방식을 강하게 제약하며, 일반적으로 추상 팩토리를 사용하도록 강제한다.

**변동성이 큰 구체 클래스로부터 파생하지 말라**
정적 타입 언어에서 상속은 소스 코드에 존재하는 모든 관계 중에서 가장 강력한 동시에 뻣뻣해서 변경하기 어렵다. 따라서 상속은 아주 신중하게 사용해야 한다. 

**구체 함수를 오버라이드 하지 말라**
대체로 구체 함수는 소스 코드 의존성을 필요로 한다. 구체 함수를 오버라이드 하면 이러한 의존성을 제거할 수 없게 되며, 실제로는 그 의존성을 상속하게 된다.


![](/assets/dip1.png)

위의 규칙들을 준수하려면 변동성이 큰 구체적인 객체는 특별히 주의해서 생성해야 한다. 이러한 점은 조심하는 게 당연한데, 사실상 모든 언어에서 객체를 생성하려면
해당 객체를 구체적으로 정의한 코드에 대해 소스 코드 의존성이 발생하기 때문이다. 자바 등 대다수의 객체 지향 언어에서 이처럼 바람직하지 못한 의존성을 처리 할 때
추상 팩토리를 사용하곤 한다.

Application은 Service 인터페이스를 통해 ConcreteImpl을 사용하지만, Application에서는 어떤 식으로든 ConcreteImpl의 인스턴스를 생성해야 한다.
ConcreteImpl에 대해 소스 코드 의존성을 만들지 않으면서 이 목적을 이루기 위해 Application은 ServiceFactory 인터페이스의 makeSvc 메서드를 호출한다.
이 메서드는 ServiceFactory로부터 파생된 ServiceFactoryImpl에서 구현된다. 그리고 ServiceFactoryImpl 구현체가 ConcreteImpl의 인스턴스를 생성한 후 
Service 타입으로 반환한다.

위의 굵은 곡선은 아키텍처 경계를 뜻한다. 이 곡선은 구체적인 것들로부터 추상적인 것들을 분리한다.

제어흐름은 소스 코드 의존성과는 정반대 방향으로 곡선을 가로지른다는 점에 주목하자. 다시말해 소스 코드 의존성은 제어흐름과는 반대 방향으로 역전된다. 이러한 이유로
이 원칙을 의존성 역전이라고 부른다.

cf) ServiceFactoryImpl 구체 클래스가 ConcreteImpl 구체 클래스에 의존하기 때문에 DIP에 위배되지만 이는 일반적인 일이다. DIP 위배를 모두 없앨 수는 없다.
하지만 DIP를 위배하는 클래스들은 적은 수의 구체 컴포넌트 내부로 모을 수 있고, 이를 통해 시스템의 나머지 부분과는 분리할 수 있다.