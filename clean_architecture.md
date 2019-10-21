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
