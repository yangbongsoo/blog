# IoC 컨테이너와 DI
**핵심 : Singleton 빈이 주인 스프링에서는 원칙적으로 싱글톤 보다 작은 lifecycle을 가지는 빈을 DI하는 것이 의미가 없고 DL을 사용해야 한다는 것이 DI의 원칙이자 자바언어의 기본 sematics이다.**

기본적으로 스프링의 빈은 싱글톤으로 만들어진다. 애플리케이션 컨텍스트마다 빈의 오브젝트는 한 개만 만들어진다는 뜻이다. 사용자의 요청이 있을 때마다 매번 애플리케이션 로직을 담은 오브젝트를 새로 만드는 건 비효율적이기 때문이다. 하나의 빈 오브젝트에 동시에 여러 스레드가 접근하기 때문에 상태 값을 인스턴스 변수에 저장해두고 사용할 수 없다. 따라서 싱글톤의 필드에는 의존관계에 있는 빈에 대한 레퍼런스나 읽기전용 값만 저장해두고 오브젝트의 변하는 상태를 저장하는 인스턴스 변수는 두지 않는다.

그런데 때로는 빈을 싱글톤이 아닌 다른 방법으로 만들어 사용해야 할 때가 있다. 빈 당 하나의 오브젝트만을 만드는 싱글톤 대신, 하나의 빈 설정으로 여러 개의 오브젝트를 만들어서 사용하는 경우다.

cf) scope : 존재할 수 있는 범위를 가리키는 말이다. 빈의 스코프는 빈 오브젝트가 만들어져 존재할 수 있는 범위다. 빈 오브젝트의 생명주기는 스프링 컨테이너가 관리하기 때문에 대부분 정해진 범위(스코프)의 끝까지 존재한다. 싱글톤 스코프는 컨테이너 스코프라고 하기도 한다. 단일 컨테이너 구조에서는 컨테이너가 존재하는 범위와 싱글톤이 존재하는 범위가 일치하기 때문이다. 요청(request) 스코프는 하나의 요청이 끝날 때까지만 존재한다.

싱글톤 스코프는 컨텍스트당 한 개의 빈 오브젝트만 만들어지게 한다. 따라서 하나의 빈을 여러 개의 빈에서 DI 하더라도 매번 동일한 오브젝트가 주입된다. DI 설정으로 자동주입하는 것 말고 컨테이너에 getBean() 메서드를 사용해 DL 하더라도 매번 같은 오브젝트가 리턴됨이 보장된다.

## 프로토타입 스코프
프로토타입 스코프는 컨테이너에게 빈을 요청할 때마다 매번 새로운 오브젝트를 생성해준다. DI, DL 상관없이 매번 새로운 오브젝트가 만들어진다.

### 프로토타입 빈의 생명주기와 종속성
IoC의 기본 개념은 애플리케이션을 구성하는 핵심 오브젝트를 코드가 아니라 컨테이너가 관리한다는 것이다. 그래서 스프링이 관리하는 오브젝트인 빈은 그 생성과 다른 빈에 대한 의존관계 주입, 초기화, DI와 DL을 통한 사용, 제거에 이르기까지 모든 오브젝트의 생명주기를 컨테이너가 관리한다. 빈에 대한 정보와 오브젝트에 대한 레퍼런스는 컨테이너가 계속 갖고 있고 필요할 때마다 요청해서 빈 오브젝트를 얻을 수 있다.

그런데 프로토타입 빈은 독특하게 이 IoC의 기본 원칙을 따르지 않는다. 프로토타입 스코프를 갖는 빈은 요청이 있을 때마다 컨테이너가 생성하고 초기화하고 DI까지 해주기도 하지만 일단 빈을 제공하고 나면 컨테이너는 더 이상 빈 오브젝트를 관리하지 않는다. 따라서 프로토타입 빈 오브젝트는 한번 DL이나 DI를 통해 컨테이너 밖으로 전달 되면 그 후부터는 더 이상 스프링이 관리하는 빈이 아니게 된다. 이때부터는 DL을 통해서 오브젝트를 가져간 코드나 DI로 주입받은 다른 빈이 사실상 컨테이너가 제공한 빈 오브젝트를 관리하게 된다. 한번 만들어진 프로토타입 빈 오브젝트는 다시 컨테이너를 통해 가져올 방법이 없고, 빈이 제거되기 전에 빈이 사용한 리소스를 정리하기 위해 호출하는 메서드도 이용할 수 없다.

프로토타입 빈은 컨테이너가 초기 생성 시에만 관여하고 DI 한 후에는 더 이상 신경 쓰지 않기 때문에 빈 오브젝트의 관리는 전적으로 DI 받은 오브젝트에 달려 있다. 그래서 프로토타입 빈은 이 빈을 주입받은 오브젝트에 종속적일 수밖에 없다. 프로토타입 빈을 주입받은 빈이 싱글톤이라면, 이 빈에 주입된 프로토타입 빈도 역시 싱글톤 생명 주기를 따라서 컨테이너가 종료될 때까지 유지될 것이다. 프로토타입 빈을 DI 받은 빈의 스코프가 더 작아서 일찍 제거돼야 한다면, DI 된 프로토타입 빈도 함께 제거될 것이다. 만약 DL 방식으로 직접 컨테이너에 getBean() 메서드를 통해서 프로토타입 빈을 요청했다면, 그 요청한 코드가 유지시켜주는 만큼 빈 오브젝트가 존재할 것이다. 메서드 안에서 사용하고 따로 저장해두지 않는다면, 메서드가 끝나면서 프로토타입 빈 오브젝트도 함께 제거된다.

### 프로토타입 빈의 용도
사용자의 요청에 따라 매번 독립적인 오브젝트를 만들어야 하는데, 매번 새롭게 만들어지는 오브젝트가 컨테이너 내의 빈을 사용해야 하는 경우가 있다. DI가 필요한 오브젝트라는 뜻이다. 오브젝트에 DI를 적용하려면 컨테이너가 오브젝트를 만들게 해야 한다. 바로 이런 경우에 프로토타입 빈이 유용하다. 프로토타입 빈은 오브젝트의 생성과 DI 작업까지 마친 후에 컨테이너가 돌려준다.

콜센터에서 고객의 A/S 신청을 받아서 접수하는 기능을 만든다고 생각해보자. 이때 등록 폼에서 고객번호를 입력받는다. 이렇게 입력받은 고객번호는 다른 입력 필드와 함께 폼 정보를 담는 오브젝트에 담겨서 서비스 계층으로 전달되어 A/S 신청 접수 기능에서 사용될 것이다.
```java
A/S 신청 폼 클래스

public class ServiceRequest {
  String customerNo;
  String productNo;
  String description;
}
```
ServiceRequest의 오브젝트는 매번 신청을 받을 때마다 새롭게 만들어지고, 폼의 정보를 담아서 서비스 계층으로 전달될 것이다. 웹 요청을 받아 처리하는 웹 컨트롤러에서는 아래와 같이 매번 new 연산자로 ServiceRequest 클래스의 오브젝트를 생성하고, 폼 요청 정보를 넣은 뒤 서비스 계층으로 전달해줘야 한다.
```java
ServiceRequest 웹 컨트롤러 

public void serviceRequestFormSubmit (HttpServletRequest request) {
    ServiceRequest serviceRequest = new ServiceRequest(); // 매 요청마다 새로운 객체를 생성한다.
    serviceRequest.setCustomerNo(request.getParameter("custno"));
    ...
    this.serviceRequestService.addNewServiceRequest(serviceRequest);
    ...
}
```
이 웹 컨트롤러는 매우 단순하고 원시적이다. 스프링의 웹 프레임워크를 사용하면 훨씬 세련되고 깔끔하게 만들 수 있지만, 일단은 어떤 식으로 동작하는지 설명하기 위한 코드라고 생각하고 보자. 일단 여기까지는 아무런 문제가 없다. 폼으로부터 요청이 있을 때마다 새로운 오브젝트를 만들고 폼의 필드에 입력된 고객번호를 저장하는 것은 자연스러운 일이다. 

이번엔 서비스 계층의 구현을 살펴보자. 콜 센터의 업무를 담당하는 서비스 오브젝트에서는 새로운 A/S 요청이 접수되면 접수된 내용을 DB에 저장하고 신청한 고객에게 이메일로 접수 안내 메일을 보내주도록 되어 있다. 폼에서는 단지 문자열로 된 고객번호를 받았을 뿐이지만 CustomerDao에게 요청하면 고객정보를 모두 가져올 수 있다. CustomerDao에서 가져온 고객정보는 Customer 오브젝트에 담겨 있을 것이고, 이를 이용해 이메일을 발송할 수도 있다. 서비스 계층의 ServiceRequestService 클래스에는 아래와 같은 코드가 만들어질 것이다.
```java
ServiceRequest 서비스 계층
public void addNewServiceRequest(ServiceRequest serviceRequest) {
  Customer customer = this.customerDao.findCustomerByNo(serviceRequest.getCustomberNo());
  ...
  this.serviceRequestDao.add(serviceRequest, customer);
  
  this.emailService.sendEmail(customer.getEmail(), 
    "A/S 접수가 정상적으로 처리되었습니다.");
}
```
이런 코드가 자연스럽게 느껴질지도 모르겠다. ServiceRequest를 단지 폼의 정보를 전달해주는 DTO와 같은 데이터 저장용 오브젝트로 취급하고, 그 정보를 이용해 실제 비지니스 로직을 처리할 때 필요한 정보는 다시 서비스 계층의 오브젝트가 직접 찾아오게 만드는 것이다.
![](/assets/오브젝트사용방식.jpg)
위 그림은 ServiceRequest가 폼의 정보를 담고 사용되는 구조를 나타낸다. 코드에서 new로 생성하는 ServiceRequest를 제외한 나머지 오브젝트는 스프링이 관리하는 싱글톤 빈이다. 이 방식의 장점은 처음 설계하고 만들기는 편하다는 것이다. 웹 페이지의 등록 폼에서 어떤 식으로 사용자 정보가 입력될지를 미리 정해두고, 그 입력 방식에 따라서 컨트롤러와 서비스 오브젝트까지 만들면 된다. 서비스 오브젝트는 폼에서 문자열로 입력된 고객번호가 ServiceRequest 오브젝트에 담겨 전달된다는 사실을 미리 알고 있다.

문제는 폼의 고객정보 입력 방법이 모든 계층의 코드와 강하게 결합되어 있다는 점이다. 만약 고객정보를 텍스트로 입력받는 대신 AJAX를 써서 이름을 이용한 자동완성 기능을 이용한 후에 Customer 테이블의 id를 폼에서 전달하는 식으로 바뀌면 어떻게 될까? ServiceRequest의 필드와 이를 처리하는 컨트롤러는 물론이고, A/S 서비스 신청을 처리하는 서비스 오브젝트인 ServiceRequestService의 코드도 다음과 같이 id 값을 이용해 Customer 오브젝트를 가져오는 방법으로 수정돼야 할 것이다.
```java
Customer customer =  this.customerDao.getCustomer(serviceRequest.getCustomerId());
```
이는 전형적인 데이터 중심의 아키텍처가 만들어내는 구조다. 비록 ServiceRequest 오브젝트에 폼 정보가 담겨 있긴 하지만, 도메인 모델을 반영하고 있다고 보기 힘들다. 모델 관점으로 보자면 서비스 요청 클래스인 ServiceRequest는 Customer라는 고객 클래스와 연결되어 있어야지, 폼에서 어떻게 입력받는지에 따라 달라지는 customerNo나 customerId 같은 값에 의존하고 있으면 안된다.

그렇다면 이 구조를 좀 더 오브젝트 중심의 구조로 만들고, 좀 더 객체지향적으로 바꾸려면 어떻게 해야 할까? 일단 웹 컨트롤러는 같은 웹 프레젠테이션 계층의 뷰에서 만들어주는 폼과 밀접하게 연결되어 있는 것이 자연스럽고 별문제가 되지 않는다. 대신 서비스 계층의 ServiceRequestService는 ServiceRequest 오브젝트에 담긴 서비스 요청 내역과 함께 서비스를 신청한 고객정보를 Customer 오브젝트로 전달받아야 한다. 그래야만 프레젠테이션 계층의 입력 방식에 따라서 비지니스 로직을 담당하는 코드가 휘둘리지 않고 독립적으로 존재할 수 있다. 따라서 ServiceRequest를 다음과 같이 변경해야 한다.
```java
public class ServiceRequest {
  Customer customer;
  String productNo;
  String description;
  ...
}
```
ServiceRequest는 customerNo 값 대신 Customer 오브젝트 자체를 참조하게 한다. ServiceRequest가 좀 더 도메인 모델에 가깝게 만들어졌으니, 서비스 계층의 코드는 다음과 같이 바꿀 수 있다.
```java
수정된 서비스 계층 코드

public void addNewServiceRequest(ServiceRequest serviceRequest) {
  this.serviceRequestDao.add(serviceRequest);
  this.emailService.sendEmail(serviceRequest.getCustomer().getEmail(),
    "A/S 접수가 정상적으로 처리되었습니다.");
}
```
폼에서 입력받은 고객번호로 고객을 찾아오는 번거로운 작업을 생략할 수 있게 됐다. serviceRequestDao에도 ServiceRequest 타입의 오브젝트만 전달하면 된다. DAO가 A/S 신청정보를 저장할 때 필요한 id와 같은 고객정보는 ServiceRequest의 customer 필드를 통해 가져올 수 있다. DAO는 물론이고 서비스 오브젝트도 폼의 입력방식에서 완전히 자유로워졌다.

그러나 아직 해결해야 할 가장 큰 문제가 남아 있다. 폼에서는 문자열로 된 고객번호를 입력받을 텐데 그것을 어떻게 Customer 오브젝트로 바꿔서 ServiceRequest에 넣어 줄 수 있을까? 답은 간단하다. customerNo를 가지고 CustomerDao에 요청해서 Customer 오브젝트를 찾아오면 된다. 이전에는 그것을 ServiceRequestService의 메서드에서 처리했는데, 이제는 어디서 해야 할까? 일단 생각해볼 수 있는 건, 웹 컨트롤러에서 CustomerDao를 사용해 Customer를 찾은 뒤에 이를 ServiceRequest에 전달하는 것이다. 이것도 그리 나쁜 방법은 아니다. 하지만 그보다 나은 방법은 ServiceRequest 자신이 처리하는 것이다.

만약 ServiceRequest가 CustomerDao에 접근할 수 있다면 어떨까? 그렇다면 다음과 같이 ServiceRequest 코드를 만들 수 있다.
```java
Customer를 검색할 수 있는 기능을 가진 ServiceRequest

public class ServiceRequest {
  Customer customer;
  ...
  @Autowired
  CustomerDao customerDao; 
  
  public void setCustomerByCustomerNo(String customerNo) {
    this.customer = customerDao.findCustomerByNo(customerNo);
  }
}
```
ServiceRequest가 CustomerDao를 DI 받아서 사용할 수 있다면 문제는 간단해진다. 폼에서 고객번호를 입력받았다면 웹 컨트롤러에서는 setCustomerByCustomerNo() 메서드를 통해 ServiceRequest 오브젝트에 전달해주기만 하면 된다. 이렇게 하면 ServiceRequestService는 ServiceRequest의 customer 오브젝트가 어떻게 만들어졌는지에 대해서는 전혀 신경쓰지 않아도 된다. 단지 A/S 신청정보에는 그것을 신청한 고객정보가 도메인 모델을 따르는 오브젝트로 만들어져 있으리라 기대하고 사용할 뿐이다.

폼에서 입력받는 것이 고객번호가 아니라 고객검색 팝업이나 AJAX를 통해 구한 고객의 ID라면, 다음과 같은 메서드를 ServiceRequest에 추가해주고 컨트롤러를 통해 id 값을 넣어주게만 하면 그만이다.
```java
public void setCustomerByCustomerId(int customerId) {
  this.customer = this.customerDao.getCustomer(customerId);
}
```
폼에서 고객정보를 입력받는 방법을 어떻게 변경하든 ServiceRequest를 사용하는 서비스 계층이나 DAO의 코드는 전혀 영향을 받지 않는다. 이제 남은 문제는 컨트롤러에서 new 키워드로 직접 생성하는 ServiceRequest 오브젝트에 어떻게 DI를 적용해서 CustomerDao를 주입할 것인가이다. DI를 적용하려면 결국 컨테이너에 오브젝트 생성을 맡겨야 한다. 또한 컨테이너가 만드는 빈이지만 매번 같은 오브젝트를 돌려주는 것이 아니라 new로 생성하듯이 새로운 오브젝트가 만들어지게 해야 한다. 바로 프로토타입 스코프 빈이 필요할 때다. 
```java
@Component
@Scope("prototype")
public class ServiceRequest {
  ...
}
```
```xml
<bean id="serviceRequest" class="...ServiceRequest" scope="prototype">
```
다음으로는 컨트롤러에서 ServiceRequest 오브젝트를 new로 생성하는 대신 프로토타입으로 선언된 serviceRequest 빈을 가져오게 만들어야 한다. 프로토타입 빈은 컨테이너에 빈을 요청할 때마다 새로운 오브젝트가 생성된다고 했다. 컨테이너에 빈을 요청하는 방법이 여러 가지가 있겠지만, 일단 가장 간단하게 아래와 같이 컨트롤러에 애플리케이션 컨텍스트를 DI 받아둔 다음 getBean() 메서드로 요청하도록 만들어보자.
```java
컨텍스트를 이용해 프로토타입 빈을 가져오는 코드

@Autowired
ApplicationContext context;

public void serviceRequestFormSubmit(HttpServletRequest request) {
  ServiceRequest serviceRequest = this.context.getBean(ServiceRequest.class);
  serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
  ...
}
```
애플리케이션 컨텍스트에서 가져온 ServiceRequest 오브젝트는 CustomerDao가 DI된 상태이기 때문에 setCustomerByCustomerNo()가 호출되면 DAO를 이용해 Customer 오브젝트를 저장해주게 만들 수 있다.

이번엔 EmailService에 대해서도 생각해보자. 고객의 A/S 신청이 접수된 것을 통보 해주는 방법을 ServiceRequestService 대신 ServiceRequest가 담당하면 어떨까? 고객이 가입할 때 A/S 관련 통보 방법을 지정할 수 있게 해뒀다면 Customer 정보에서 이를 확인하고, 적절한 방법으로 고객에게 메세지를 보내주는 작업을 ServiceRequest에 두는 것도 나쁘지 않다. ServiceRequest도 이제 자유롭게 DI 받을 수 있는 빈이 됐으니 EmailService를 이용할 수 있다.
```java
public class ServiceRequest {
  Customer customer;
  @Autowired
  EmailService emailService;
  ... 
 
 public void notifyServiceRequestRegistration() { // A/S 요청이 등록됐음을 통보해주는 기능을 가진 메서드
   if (this.customer.serviceNotificationMethod == NotificationMethod.EMAIL) { 
     this.emailService.sendEmail(customer.getEmail(),
       "A/S 접수가 정상적으로 처리되었습니다.");
   }
 }
}
```
이제 ServiceRequestService의 A/S 신청 접수를 처리하는 메서드는 아래와 같이 구체적인 통보 방식에 매이지 않고 ServiceRequest 오브젝트에게 통보를 보내라는 요청만 하는 깔끔한 코드로 만들 수 있다.
```java
public void addNewServiceRequest(ServiceRequest serviceRequest) {
  this.serviceRequestDao.add(serviceRequest);
  serviceRequest.notifyServiceRequestRegistration(); // 구체적인 통보 작업은 ServiceRequest에서 알아서 담당하게 한다.
}
```
![](/assets/ioc_contatiner.jpg)
ServiceRequest를 프로토타입 빈으로 변경하면서 새롭게 바뀐 의존관계다. 이렇게 매번 새롭게 오브젝트를 만들면서 DI도 함께 적용하려고 할 때 사용할 수 있는게 바로 프로토타입 빈이다. 한번 컨테이너로부터 생성해서 가져온 이후에는 new로 직접 생성한 오브젝트처럼 평범하게 사용하면 된다. 빈으로 만들어진 오브젝트이기 때문에 DI를 통해 주입된 다른 빈을 자유롭게 이용할 수 있다.

### 프로토타입 빈의 DL 전략
앞에서 ServiceRequest를 프로토타입 빈으로 만들고 컨트롤러에서 가져오도록, ApplicationContext를 이용해 getBean() 메서드를 호출하는 방식을 이용했다. 즉 DL을 사용한 것이다. 번거롭게 DL 방식을 쓰지 않고 프로토타입 빈을 직접 DI 해서 사용하는 건 어떨까? 예를 들어 아래 처럼 컨트롤러에서 ServiceRequest를 직접 DI 받게 만들고 이를 사용하면 어떻게 될까?
```java
@Autowired
ServiceRequest serviceRequest;

public void serviceRequestFormSubmit(HttpServletRequest request) {
  this.serviceRequest.setCustomerNo(request.getParameter("custno"));
  ...
}
```
이 코드를 테스트해보면 일단 정상적으로 동작하는 것처럼 보이지만 운영 시스템에 적용하면 매우 심각한 문제가 발견된다. 왜 그럴까? 웹 컨트롤러도 다른 대부분의 빈처럼 싱글톤이다. 따라서 단 한 번만 만들어진다. 문제는 DI 작업은 빈 오브젝트가 처음 만들어질 때 단 한 번만 진행된다는 점이다. 따라서 아무리 ServiceRequest 빈을 프로토타입으로 만들었다고 하더라도 컨트롤러에 DI 하기 위해 컨테이너에 요청할 때 딱 한번만 오브젝트가 생성되고 더 이상 새로운 ServiceRequest 오브젝트는 만들어지지 않는다. 결국 여러 사용자가 동시에 요청을보내면 serviceRequest 오브젝트 하나가 공유되어 서로 데이터를 덮어써 버리는 문제가 발생한다.

프로토타입 빈은 DI 될 대상이 여러 군데라면 각기 다른 오브젝트가 생성된다. 하지만 ServiceRequest 처럼 같은 컨트롤러에서도 매번 요청이 있을 때마다 새롭게 오브젝트가 만들어져야 하는 경우에는 적합하지 않다. new 키워드를 대신하기 위해 사용되는 것이 프로토타입의 용도라고 본다면, DI는 프로토타입 빈을 사용하기에 적합한 방법이 아니다. 따라서 코드 내에서 필요할 때마다 컨테이너에게 요청해서 새로운 오브젝트를 만들어야 한다. DL 방식으로 사용해야 한다는 뜻이다. **프로토타입 빈이 DI 방식으로 사용되는 경우는 매우 드물다.**

앞에서 ApplicationContext를 DI 받아둔 뒤에 코드에서 getBean() 메서드를 직접 호출하는 방법을 사용했다. 가장 단순하고 직접적인 방식이며, 사용하기도 별로 어렵지 않다. 반면에 스프링의 API가 일반 애플리케이션 코드에서 사용된다는 사실이 불편하게 느껴질 수도 있다. 게다가 단위 테스트를 작성하려면 ApplicationContext라는 거대한 인터페이스의 목 오브젝트를 만들어야 하는 부담도 뒤따른다. 스프링은 프로토타입 빈처럼 DL 방식을 코드에서 사용해야 할 경우를 위해 직접 ApplicationContext를 이용하는 것 외에도 다양한 방법을 제공하고 있다.

**ApplicationContext, BeanFactory**
이미 사용했던 방법이다. @Autowired나 @Resorce를 이용해 ApplicationContext 또는 BeanFactory를 DI 받은 후에 getBean() 메서드를 직접 호출해서 빈을 가져오는 방법이다.

**ObjectFactory, ObjectFactoryCreatingFactoryBean**
직접 애플리케이션 컨텍스트를 사용하지 않으려면 중간에 컨텍스트에 getBean()을 호출해주는 역할을 맡을 오브젝트를 두면 된다. 가장 쉽게 생각해볼 수 있는 것은 바로 팩토리다. 팩토리를 이용하는 이유는 오브젝트를 요구하면서 오브젝트를 어떻게 생성하거나 가져오는지에는 신경 쓰지 않을 수 있기 때문이다. ApplicationContext를 DI 받아서 getBean()을 호출해 원하는 프로토타입 빈을 가져오는 방식으로 동작하는 팩토리를 하나 만들어서 빈으로 등록해두고, 이 팩토리 역할을 하는 빈을 DI 받아서 필요할 때 getObject()와 같은 메서드를 호출해 빈을 가져올 수 있도록 만드는 방법이 있다.

스프링이 제공하는 ObjectFactory 인터페이스와 ObjectFactory 인터페이스를 구현한 팩토리를 만들어주는 특별한 빈 클래스를 사용해보자. 스프링의 ObjectFactory 인터페이스는 타입 파라미터와 getObject()라는 간단한 팩토리 메서드를 갖고 있다.
```java
ObjectFactory<ServiceRequest> factory = ...;
ServiceRequest request = factory.getObject();
```
ObjectFactory는 비록 스프링이 제공하는 인터페이스이지만 평범하고 간단한 메서드만 갖고 있기 때문에 복잡한 ApplicationContext를 직접 사용하는 것보다 훨씬 깔끔하고 테스트하기도 편하다. ObjectFactory의 구현 클래스는 이미 스프링이 제공해주고 있다. 프로토타입처럼 컨텍스트에서 매번 빈을 가져와야 하는 구조의 팩토리를 만들 때 손쉽게 사용할 수 있도록 만들어져 있다. 클래스의 이름은 ObjectFactoryCreatingFactoryBean이다. ObjectFactory를 만들어주는 팩토리 빈이라는 뜻이다.

사용 방법은 아래와 같이 getBean()으로 가져올 빈의 이름을 넣어서 등록해주면 된다. 이 빈은 FactoryBean이기 때문에 실제 빈의 오브젝트는 ObjectFactory 타입이 된다.
```xml
<bean id="serviceRequestFactory" class= "org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">
  <property name="targetBeanName" value="serviceRequest"/>
</bean>
```
cf) 여기서 value는 팩토리 메서드에서 getBean()으로 가져올 빈의 이름을 넣는다.
```java
@Configuration
public class ObjectFactoryConfig {
  @Bean
  public ObjectFactoryCreatingFactoryBean serviceRequestFactory() {
    ObjectFactoryCreatingFactoryBean factoryBean = new ObjectFactoryCreatingFactoryBean();
    factoryBean.setTargetBeanName("serviceRequest");
    return factoryBean;
  }
}
```
이제 serviceRequestFactory 빈을 ServiceRequest를 사용할 컨트롤러에 DI 해주고 아래와 같이 사용하면 된다.
```java
@Resource // ObjectFactory 타입은 여러개 있을 수 있으므로 이름으로 빈을 지정하는 편이 낫다.
private ObjectFactory<ServiceRequest> serviceRequestFactory;

public void serviceRequestFormSubmit(HttpServletRequest request) {
  ServiceRequest serviceRequest = this.serviceRequestFactory.getObject();
  serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
  ...
}
```
ObjectFactory는 프로토타입 빈뿐 아니라 DL을 이용해 빈을 가져와야 하는 모든 경우에 적용할 수 있다.

**ServiceLocatorFactoryBean**
ObjectFactory가 단순하고 깔끔하지만 프레임워크의 인터페이스를 애플리케이션 코드에서 사용하는 것이 맘에 들지 않을 수 있다. 또는 기존에 만들어둔 팩토리 인터페이스를 활용하고 싶을지도 모르겠다. 이럴 땐 ObjectFactoryCreatingFactoryBean 대신 ServiceLocatorFactoryBean을 사용하면 된다.

ServiceLocatorFactoryBean은 ObjectFactory처럼 스프링이 미리 정의해둔 인터페이스를 사용하지 않아도 된다. DL 방식으로 가져올 빈을 리턴하는 임의의 이름을 가진 메서드가 정의된 인터페이스가 있으면 된다. 메서드 이름은 어떻게 지어도 상관없다. 
```java
public interface ServiceRequestFactory {
  ServiceRequest getServiceFactory();
}
```
이렇게 정의한 인터페이스를 이용해 스프링의 ServiceLocatorFactoryBean으로 아래와 같이 빈을 등록해주면 된다.
```xml
<bean class="org.springframework.beans.factory.config.ServiceLocatorFactoryBean">
  <property name="serviceLocatorInterface" value=".. ServiceRequestFactory" /> //팩토리 인터페이스를 지정한다. 빈의 실제 타입이 된다.
</bean>
```
범용적으로 사용하는 ObjectFactory와 달리 ServiceRequest 전용으로 만든 인터페이스가 이 빈의 타입이 되기 때문에 @Autowired를 이용해 타입으로 가져올 수 있다. 빈을 이름으로 접근할 필요가 없을 때는 위의 빈 선언처럼 id를 생략할 수도 있다. 컨트롤러에서 사용할 때는 아래와 같이 팩토리 인터페이스 타입으로 DI 받아서 사용하면 된다. 타입 파라미터를 사용해야 하는 ObjectFactory보다 코드가 한결 깔끔하다.
```java
팩토리 인터페이스를 사용하는 컨트롤러 코드

@Autowired
ServiceRequestFactory serviceRequestFactory;

public void serviceRequestFormSubmit(HttpServletRequest request) {
  ServiceRequest serviceRequest = this.serviceRequestFactory.getServiceFactory();
  serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
  ...
}
```

**메서드 주입**
ApplicationContext를 직접 이용하는 방법은 스프링 API에 의존적인 코드를 만드는 불편함이 있다. 반면에 ObjectFactory나 ServiceLocatorFactoryBean을 사용하면 코드는 깔끔해지지만 빈을 새로 추가해야 하는 번거로움이 있다. 이 두 가지 단점을 모두 극복할 수 있도록 스프링이 제공해주는 또 다른 DL 전략은 메서드 주입이다.

메서드 주입은 @Autowired를 메서드에 붙여서 메서드 파라미터에 의해 DI 되게 하는 메서드를 이용한 주입 방식과 혼동하면 안된다. 메서드 주입은 메서드를 통한 주입이 아니라 메서드 코드 자체를 주입하는 것을 말한다. 메서드 주입은 일정한 규칙을 따르는 추상 메서드를 작성해두면 ApplicationContext와 getBean() 메서드를 사용해서 새로운 프로토타입 빈을 가져오는 기능을 담당하는 메서드를 런타임 시에 추가해주는 기술이다.

컨트롤러 클래스에 아래와 같이 추상 메서드를 선언해둔다. 팩토리 역할을 하는 메서드라고 보면 된다.
그리고 이 메서드를 사용해 새로운 빈 오브젝트를 가져오도록 코드를 작성한다.
```java
abstract public ServiceRequest getServiceRequest();

public void serviceRequestFormSubmit(HttpServletRequest request) {
  ServiceRequest serviceRequest = this.getServiceFactory();
  serviceRequest.setCustomerByCustomerNo(request.getParameter("custno"));
  ...
}
```
추상 메서드를 가졌으므로 당연히 클래스도 추상 클래스로 정의돼야 한다. 이제 이 추상 클래스를 확장해서 getServiceRequest()라는 추상 메서드를 주입해주도록 스프링 빈을 다음과 같이 정의한다.
```xml
<bean id="serviceRequestController" class="...ServiceRequestController">
  <lookup-method name="getServiceRequest" bean="serviceRequest"/>
</bean>
```
`<lookup-method>`라는 태그의 name이 스프링이 구현해줄 추상 메서드의 이름이고, bean 애트리뷰트는 메서드에서 getBean()으로 가져올 빈의 이름이다. 이렇게 설정해두면 스프링은 추상 클래스를 상속해서 getServiceRequest() 메서드를 완성하고 상속한 클래스를 빈으로 등록해둔다.

메서드 주입 방식은 그 자체로 스프링 API에 의존적이 아니므로 스프링 외의 환경에 가져다 사용할 수도 있고 컨테이너의 도움 없이 단위 테스트를 할 수도 있다. 지금까지 살펴본 것중에서 가장 고급 방식이지만 불편한 점도 있다. 클래스 자체가 추상 클래스이므로 테스트에서 사용할 때 상속을 통해 추상 메서드를 오버라이드한 뒤에 사용해야 한다는 번거로움이 있다. 단위 테스트를 많이 작성할 것이라면 메서드 주입 방법은 장점보다 단점이 더 많을 수 있다.

**`Provider<T>`**
마지막으로 살펴볼 프로토타입 빈을 DL 하는 방법은 가장 최근에 소개된 것이다. @Inject와 함께 JSR-330에 추가된 표준 인터페이스인 Provider를 이용하는 것이다. Provider는 ObjectFactory와 거의 유사하게 `<T>`타입 파라미터와 get()이라는 팩토리 메서드를 가진 인터페이스다. 기본 개념과 사용 방법은 ObjectFactory와 거의 유사하지만 ObjectFactoryCreatingFactoryBean을 이용해 빈을 등록해주지 않아도 되기 때문에 사용이 편리하다. Provider 인터페이스를 @Inject, @Autowired, @Resource 중의 하나를 이용해 DI 되도록 지정해주기만 하면 스프링이 자동으로 Provider를 구현한 오브젝트를 생성해서 주입해주기 때문이다. 오브젝트 팩토리 주입이라고 생각해도 좋을 것이다. 팩토리 빈을 XML이나 @Configuration 자바 코드로 정의하지 않아도 ObjectFactory처럼 동작하기 때문에 손쉽게 사용할 수 있다. Provider를 사용할 때는 아래와 같이 타입 파라미터로 생성할 빈의 타입을 넣어주기만 하면 된다.
```java
@Inject
Provider<ServiceRequest> serviceRequestProvider;

public void serviceRequestFormSubmit(HttpServletRequest request) {
  ServiceRequest serviceRequest = this.serviceRequestProvider.get();
  serviceRequest.setCustomerByCustomerNo(request.getParameter("custno");
  ...
}
```

### 빈 등록정보 조회 유틸리티 클래스
```java
import java.util.ArrayList;
import java.util.List;
import org.springframework.context.support.GenericApplicationContext;

public class BeanDefinitionUtils {
	public static void printBeanDefinitions(GenericApplicationContext gac) {
		List<List<String>> roleBeanInfos = new ArrayList<>();
		roleBeanInfos.add(new ArrayList<>());
		roleBeanInfos.add(new ArrayList<>());
		roleBeanInfos.add(new ArrayList<>());
		
		for (String name : gac.getBeanDefinitionNames()) {
			int role = gac.getBeanDefinition(name).getRole();
			List<String> beanInfos = roleBeanInfos.get(role);
			beanInfos.add(role + "\t" + name + "\t" + gac.getBean(name).getClass().getName());
		}
		
		for (List<String> beanInfos : roleBeanInfos) {
			for (String beanInfo : beanInfos) {
				System.out.println(beanInfo);
			}
		}
	}
}
```  
컨테이너의 빈 등록 정보 확인 (vol.1 p692)
```java
	@Autowired
	DefaultListableBeanFactory bf;

	@Test
	public void beanTest() throws Exception {
		for (String s : bf.getBeanDefinitionNames()) {
			System.out.println(bf.getBean(s).getClass().getName());
		}
	}
```
