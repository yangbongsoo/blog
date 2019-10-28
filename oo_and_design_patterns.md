# 개발자가 반드시 정복해야할 객체지향과 디자인패턴
## 서비스 로케이터
로버트 C 마틴은 소프트웨어를 두 개의 영역으로 구분해서 설명하고 있는데, 한 영역은 고수준 정책 및 저수준 구현을 포함한 애플리케이션 영역이고 또 다른 영역은 애플리케이션이 동작하도록 각 객체들을 연결해 주는 메인 영역이다. 본 장에서는 애플리케이션 영역과 메인 영역에 대해 살펴보고, 메인 영역에서 객체를 연결하기 위해 사용되는 방법인 DI와 서비스 로케이터에 대해 알아보자.

![](/assets/servicelocator1.jpg)

JobQueue와 Transcoder는 변화되는 부분을 추상화한 인터페이스로서, 다른 코드에 영향을 주지 않으면서 확장할 수 있는 구조를 갖고 있다.(OCP) 따라서 Worker 클래스는 이들 콘크리트 클래스에 의존하지 않는다.

Worker 클래스는 JobQueue에 저장된 객체로부터 JobData를 가져와 Transcoder를 이용해서 작업을 실행하는 책임이 있다. 
```java
public class Worker {
    public void run(){
        JobQueue jobQueue = ...;// JobQueue를 구한다.
        Transcoder transcoder = ...; // Transcoder를 구한다.
        boolean someRunningCondition = true;

        while(someRunningCondition){
            JobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}
```
Worker가 제대로 동작하려면 JobQueue나 Transcoder를 구현한 클래스의 객체가 필요하다. 비슷하게 JobCLI 클래스도 JobQueue에 작업 데이터를 넣어야 하는데, 이를 수행하려면 JobQueue를 구현한 객체를 구해야 한다.

이를 위해 Locator라는 객체를 사용하기로 했다고 해보자. 
```java
public class Locator {
    private static Locator instance;
    public static Locator getInstance(){
        return instance;
    }

    public static void init(Locator locator){
        instance = locator;
    }

    private JobQueue jobQueue;
    private Transcoder transcoder;

    public Locator(JobQueue jobQueue, Transcoder transcoder){
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }

    public JobQueue getJobQueue(){
        return jobQueue;
    }

    public Transcoder getTranscoder(){
        return transcoder;
    }
}
```
```java
public class Worker {
    public void run(){
        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        Transcoder transcoder = Locator.getInstance().getTranscoder();

        boolean someRunningCondition = true;

        while(someRunningCondition){
            JobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}

public class JobCLI {
    public void interact(){
        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        File source = new File("test.txt");
        File target = new File("test2.txt");

        jobQueue.addJob(new JobData(source,target));
    }
}
```
**여기서 질문이 발생한다. 그렇다면 과연 누가 Locator 객체를 초기화 해줄 것인가? 그리고 JobCLI 객체와 Worker 객체를 생성하고 실행해 주는 건 누구인가?**

드디어 메인 영역이 출현할 차례가 되었다. 메인 영역은 다음 작업을 수행한다. 
1. 애플리케이션 영역에서 사용될 객체를 생성한다.
2. 각 객체 간의 의존 관계를 설정한다.
3. 애플리케이션을 실행한다.

![](/assets/servicelocator2.jpg)

```java
public class MainTest {
    public static void main(String[] args) {
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();

        Locator locator = new Locator(jobQueue, transcoder);
        Locator.init(locator);

        final Worker worker = new Worker();
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                worker.run();
            }
        });

        JobCLI cli = new JobCLI();
        cli.interact();
    }
}
```
**사용할 객체를 제공하는 책임을 갖는 객체를 서비스 로케이터라고 부른다.**

프로그램 개발 환경이나 사용하는 프레임워크의 제약으로 인해 DI 패턴을 적용할 수 없는 경우가 있다. 예를 들어 안드로이드 플랫폼의 경우는 화면을 생성할 때 Activity 클래스를 상속받도록 하고 있는데, 이 때 안드로이드 실행환경은 정해진 메서드만을 호출할 뿐, 안드로이드 프레임워크가 DI 처리를 위한 방법을 제공하지는 않는다. 따라서 의존 객체를 찾는 다른 방법인 서비스 로케이터를 살펴보자.

서비스 로케이터를 구현하는 방법은 다양하게 존재할 수 있는데, 본 장에서는 객체 등록 방식의 구현 방법과 상속을 통한 구현 방법에 대해 알아볼 것이다.

**객체 등록 방식의 구현**
```java
// 생성자를 이용해서 객체를 등록 받는 서비스 로케이터 구현
public class ServiceLocator {
    private JobQueue jobQueue;
    private Transcoder transcoder;

    public ServiceLocator(JobQueue jobQueue, Transcoder transcoder){
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }
    public JobQueue getJobQueue(){
        return jobQueue;
    }
    public Transcoder getTranscoder(){
        return transcoder;
    }
    //서비스 로케이터 접근 위한 static 메서드
    private static ServiceLocator instance;
    public static void load(ServiceLocator locator){
        ServiceLocator.instance = locator;
    }
    public static ServiceLocator getInstance(){
        return instance;
    }
}
```
```java
// 메인 영역에서 서비스 로케이터에 객체 등록
public static void main(String[] args){
  //의존 객체 생성
  FileJobQueue jobQueue = new FileJobQueue();
  FfmpegTranscoder transcoder = new FfmpegTranscoder();
  
  //서비스 로케이터 초기화
  ServiceLocator locator = new ServiceLocator(jobQueue, transcoder);
  ServiceLocator.load(locator);
  
  //애플리케이션 코드 실행
  Worker worker = new Worker();
  JobCLI jobCli = new JobCLI();
  jobCli.interact();
}
```
```java
public class Worker {
    public void run(){
        //서비스 로케이터 이용해서 의존 객체 구함
        ServiceLocator locator = ServiceLocator.getInstance();
        JobQueue jobQueue = locator.getJobQueue();
        Transcoder transcoder = locator.getTranscoder();

        boolean someRunningCondition = true;

        while(someRunningCondition){
            JobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}
```
그런데 서비스 로케이터가 제공할 객체 종류가 많을 경우, 서비스 로케이터 객체를 생성할 때 한번에 모든 객체를 전달하는 것은 코드 가독성을 떨어뜨릴 수 있다. 이런 경우에는 각 객체마다 별도의 등록 메서드를 제공하는 방식을 취해서 서비스 로케이터 초기화 부분의 가독성을 높여줄 수 있다.
```java
// 객체마다 등록 메서드를 따로 제공하는 방식
public class ServiceLocator {
    private JobQueue jobQueue;
    private Transcoder transcoder;

    public void setTranscoder(Transcoder transcoder) {
        this.transcoder = transcoder;
    }
    public void setJobQueue(JobQueue jobQueue) {
        this.jobQueue = jobQueue;
    }
    public JobQueue getJobQueue(){
        return jobQueue;
    }
    public Transcoder getTranscoder(){
        return transcoder;
    }
    //서비스 로케이터 접근 위한 static 메서드
    private static ServiceLocator instance;
    public static void load(ServiceLocator locator){
        ServiceLocator.instance = locator;
    }
    public static ServiceLocator getInstance(){
        return instance;
    }
}
```
객체를 등록하는 방식의 장점은 서비스 로케이터 구현이 쉽다는 점이다. 하지만 서비스 로케이터에 객체를 등록하는 인터페이스가 노출되어 있기 때문에 애플리케이션 영역에서 얼마든지 의존 객체를 바꿀 수 있다.
```java
public class Worker{
  public void run(){
    //고수준 모듈에서 저수준 모듈에 직접 접근하는 걸 유도할 수 있음
    ServiceLocator oldLocator = ServiceLocator.getInstance();
    ServiceLocator newLocator = new ServiceLocator(
      //DIP 위반
      new DbJobQueue(), oldLocator.getTranscoder());
  }
}
```
**상속을 통한 구현**
객체를 구하는 추상 메서드를 제공하는 상위 타입 구현
상위 타입을 상속받은 하위 타입에서 사용할 객체 설정
```java
// 상속 방식 서비스 로케이터 구현의 상위 타입
public abstract class ServiceLocator {
    public abstract JobQueue getJobQueue();
    public abstract Transcoder getTranscoder();

    protected ServiceLocator(){
        if(instance != null)
            throw new IllegalStateException("이미 있어");
        ServiceLocator.instance = this;
    }

    private static ServiceLocator instance;
    public static ServiceLocator getInstance(){
        return instance;
    }
}
```
ServiceLocator가 추상 클래스라는 것은 이 클래스를 상속받아 추상 메서드의 구현을 제공하는 클래스가 필요하다는 뜻이다. 
```java
public class MyServiceLocator extends ServiceLocator{
    private FileJobQueue jobQueue;
    private FfmpegTranscoder transcoder;

    public MyServiceLocator() {
        super();
        this.jobQueue = new FileJobQueue();
        this.transcoder = new FfmpegTranscoder();
    }

    @Override
    public JobQueue getJobQueue() {
        return jobQueue;
    }

    @Override
    public Transcoder getTranscoder() {
        return transcoder;
    }
}
```
**서비스 로케이터의 단점은 인터페이스 분리 원칙을 위반한다는 점이다. 예를 들어 JobCLI 클래스가 사용하는 타입은 JobQueue 뿐인데, ServiceLocator를 사용함으로써 Transcoder 타입에 대한 의존이 함께 발생하게 된다.**
```java
public class JobCLI {
    public void interact(){
        ...
        //ServiceLocator의 인터페이스 변경 시 영향을 받을 수 있음
        JobQueue jobQueue = ServiceLocator.getInstance().getJobQueue();
  }
}
```
서비스 로케이터를 사용하는 코드가 많아질수록 이런 문제가 배로 발생하게 된다. 이 문제를 해결하려면 의존 객체마다 서비스 로케이터를 작성해 주어야 한다. 이 방법은 의존 객체 별로 서비스 로케이터 인터페이스가 분리되는 효과는 얻을 수 있지만, 다음 코드 처럼 동일한 구조의 서비스 로케이터 클래스를 중복해서 만드는 문제를 야기할 수 있다.
```java
// 타입만 다르고 구조가 완전히 같은 Locator 클래스들
public class JobQueueLocator{
  private JobQueue jobQueue;
  public void setJobQueue(JobQueue jobQueue){
    this.jobQueue = jobQueue;
  }
  public JobQueue getJobQueue(){
    return jobQueue;
  }
  
  private static JobQueueLocator instance;
  public static void load(JobQueueLocator locator){
    JobQueueLocator.instance = locator;
  }
  public static JobQueueLocator getInstance(){
    return instance;
  }
}

// TranscoderLocator는 JobQueueLocator와 동일하다
public class TranscoderLocator{
  private Transcoder transcoder;
  public void setTranscoder(Transcoder transcoder){
    this.transcoder = transcoder;
  }
  public Transcoder getTranscoder(){
    return transcoder;
  }
  // 동일 패턴 코드
}
```
이런 중복된 코드를 무조건 피해야 하는데, **제네릭을 사용한 서비스 로케이터는 중복을 피하면서 인터페이스를 분리한 것과 같은 효과를 낼 수 있다.**
```java
public class ServiceLocator {
    private static Map<Class<?>, Object> objectMap =
            new HashMap<>();

    public static <T> T get(Class<T> klass){
        return (T) objectMap.get(klass);
    }
    public static void regist(Class<?> klass, Object obj){
        objectMap.put(klass,obj);
    }
}
```
메인 영역에서는 ServiceLocator.regist() 메서드를 이용해서 객체를 등록해 준다.
```java
ServiceLocator.regist(JobQueue.class, new FileJobQueue());
ServiceLocator.regist(Transcoder.class, new FfmpegTranscoder());
```
ServiceLocator를 사용하는 코드는 다음과 같이 코드를 작성하게 된다.
```java
// JobQueue에만 의존
JobQueue jobQueue = ServiceLocator.get(JobQueue.class);
```

**서비스 로케이터의 가장 큰 단점은 동일 타입의 객체가 다수 필요할 경우, 각 객체 별로 제공 메서드를 만들어 주어야 한다는 점이다.** 예를 들어, FileJobQueue 객체와 DbJobQueue 객체가 서로 다른 부분에 함께 사용되어야 한다면, 이 경우 ServiceLocator는 다음과 같이 두 개의 메서드를 제공해야 한다.
```java
public class ServiceLocator{
  public JobQueue getJobQueue1() { ... }
  public JobQueue getJobQueue2() { ... }
}
```
1,2를 붙이는게 맘에 안들지만 그렇다고 메서드 이름에 File이나 Db라는 단어를 붙이는 것도 안된다. 콘크리트 클래스에 직접 의존하는 것과 동일한 효과를 발생시키기 때문이다. 결론은 부득이한 상황이 아니라면 서비스 로케이터보다는 DI를 사용하자.
## 옵저버(Observer) 패턴

![](/assets/observerpattern2.jpg)

StatusChecker는 시스템의 상태가 불안정해지면 이 사실을 SmsSender, MessageSender, EmailSender 객체에게 알려주는데, 여기서 **핵심은 상태가 변경될 때 정해지지 않은 임의의 객체에게 변경 사실을 알려준다는 점이다.** 이렇게 한 객체의 상태 변화를 정해지지 않은 여러 다른 객체에 통지하고 싶을 때 사용되는 패턴이 옵저버 패턴이다.

옵저버 패턴에는 크게 주제(subject) 객체와 옵저버 객체가 등장하는데, 주제 객체는 다음의 두 가지 책임을 갖는다.
1. 옵저버 목록을 관리하고, 옵저버를 등록하고 제거할 수 있는 메서드를 제공한다.
2. 상태의 변경이 발생하면 등록된 옵저버에 변경 내역을 알린다. notifyStatus() 메서드가, 등록된 옵저버 객체의 onAbnormalStatus() 메서드를 호출한다.

```java
public class StatusSubject {
    private List<StatusObserver> observers = new ArrayList<>();

    public void add(StatusObserver observer){
        observers.add(observer);
    }

    public void remove(StatusObserver observer){
        observers.remove(observer);
    }

    public void notifyStatus(Status status){
        for(StatusObserver observer : observers){
            observer.onAbnormalStatus(status);
        }
    }
}
```
Status의 상태 변경을 알려야 하는 StatusChecker 클래스는 StatusSubject 클래스를 상속받아 구현한다.
```java
// 옵저버에게 통지가 필요한 콘크리트 클래스의 구현
public class StatusChecker extends StatusSubject{

    public void check(){
        Status status = loadStatus();

        if(status.isNotNormal()) {
            super.notifyStatus(status);
        }
    }

    private Status loadStatus(){
        Status status = new Status();
        return status;
    }
}
```
StatusChecker 클래스는 비정상 상태가 감지되면 상위 클래스의 notifyStatus() 메서드를 호출해서 등록된 옵저버 객체들에 상태 값을 전달한다.

주제 객체의 상태에 변화가 생길 때 그 내용을 통지받도록 하려면 옵저버 객체를 주제 객체에 등록해 주어야 한다.
```java
public class ObserverMain {
    public static void main(String[] args) {
        StatusChecker statusChecker = new StatusChecker();
        statusChecker.add(new StatusEmailSender());
        statusChecker.add(new StatusMessageSender());
        statusChecker.add(new StatusSmsSender());
        statusChecker.check();
    }
}
```
**옵저버 패턴을 적용할 때의 장점은 주제 클래스 변경 없이 상태 변경을 통지 받을 옵저버를 추가할 수 있다는 점이다.**

옵저버 패턴이 가장 많이 사용되는 영역을 꼽으라면 GUI 프로그래밍 영역일 것이다. 버튼이 눌릴 때 로그인 기능을 호출한다고 할 때, 버튼이 주제 객체가 되고 로그인 모듈을 호출하는 객체가 옵저버가 된다.

예를 들어 안드로이드에서는 다음과 같이 OnClickListener 타입의 객체를 Button 객체에 등록하는데, 이때 OnClickListener 인터페이스가 옵저버 인터페이스가 된다.
```java
public class MyActivity extends Activity implements View.OnClickListener{
	public void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		…
		Button loginButton = getViewById(R.id.main_loginbtn);
		loginButton.setOnClickListener(this);
	}

	@Override
	public void onClick(View v){
		login(id, password);
	}
}
```
한 개의 옵저버 객체를 여러 주체 객체에 등록할 수도 있을 것이다. GUI 프로그래밍을 하면 이런 상황이 흔하게 발생한다.
```java
public class MyActivity extends Activity implements View.OnClickListener{
	public void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		…
		// 두 개의 버튼에 동일한 OnClickListener 객체 등록
		Button loginButton = (Button) findViewById(R.id.main_loginbtn);
		loginButton.setOnClickListener(this);
		Button logoutButton = (Button) findViewById(R.id.main_logoutbtn);
		logoutButton.setOnClickListener(this);
	}

	@Override
	public void onClick(View v){
		//주제 객체를 구분할 수 있는 방법 필요
		if(v.getId() == R.id.main_loginbtn)
			login(id, password);
		else if(v.getId() == R.id.main_logoutbtn)
			logout();
	}
}
```
앞서 StatusChecker  예제나 안드로이드 예제는 모두 주제 객체를 위한 추상 타입을 제공하고 있다. 예를 들어, StatusChecker는 상위 타입인 StatusSubject 추상 클래스가 존재하고 안드로이드의 Button 클래스의 상위 타입은 View가 존재한다. 둘 모두 옵저버 객체를 관리하기 위한 기능을 제공한다는 공통점이 있다.
```java
// StatusChecker 클래스
public void add(StatusObserver observer) { … }

// View 클래스 
public void setOnClickListener(OnClickListener o) { … }
```

## 단일 책임 원칙
클래스는 단 한 개의 책임을 가져야 한다. 즉, 클래스를 변경하는 이유는 단 한 개여야 한다.
그런데 단일 책임 원칙은 가장 어려운 원칙이기도 하다. 한개의 책임에 대한 정의가 명확하지 않고, 책임을 도출하기 위해서는 다양한 경허이 필요하기 때문이다.

```java
public class DataViewer {
    public void display() {
        String data = loadHtml();
        updateGui(data);
    }

    public String loadHtml() {
        HttpClient client = new HttpClient();
        client.connect(url);
        return client.getResponse();
    }

    private void updateGui(String data) {
        GuiData guiModel = parseDataToGuiData(data);
        tableUI.changeData(guiModel);
    }

    private GuiData parseDataToGuiData(String data) {
        // ... 파싱 처리 코드
    }

    // ... 기타 필드 등 다른 코드
}
```
display 메서드는 loadHtml 메서드에서 읽어 온 HTML 응답 문자열을 updateGui 메서드에 보낸다.
updateGui 메서드는 parseDataToGuiData 메서드를 이용해서 HTML 응답 메세지를 GUI에 보여주기 위한
GuiData 객체로 변환한 뒤에 실제 tableUI를 이용해서 데이터를 보여주고 있다.

여기서 데이터를 제공하는 서버가 HTTP 프로토콜에서 소켓 기반의 프로토콜로 변경되었다. 이 프로토콜은 응답 데이터로 byte 배열을 제공한다.
그러면 아래와 같은 변화가 연쇄적으로 발생할 것이다.

```java
    public void display() {
        // String data = loadHtml();
        byte[] data = loadHtml();
        updateGui(data);
    }

    public byte[] loadHtml() {
        // HttpClient client = new HttpClient();
        // client.connect(url);
        // return client.getResponse();
        SocketClient client = new SocketClient();
        client.connect(server, port);
        return client.read();
    }

    private void updateGui(byte[] data) {
        GuiData guiModel = parseDataToGuiData(data);
        tableUI.changeData(guiModel);
    }

    private GuiData parseDataToGuiData(byte[] data) {
        // ... 파싱 처리 코드
    }

    // ... 기타 필드 등 다른 코드

```
이러한 연쇄적인 코드 수정은 두 개의 책임(데이터를 읽는 책임과 화면에 보여주는 책임)이 한 클래스에 아주 밀접하게 결합되어 있어서 발생한 증상이다.

![](/assets/srp1.png)

위와 같이 데이터 읽기와 데이터를 화면에 보여주는 책임을 두 개의 클래스로 분리하고 둘 간에 주고받는 데이터를 저수준의 String이 아닌
알맞게 추상화된 타입을 사용하면, 데이터를 읽어 오는 부분의 변경 때문에 화면을 보여주는 부분의 코드가 변경되는 상황을 막을 수 있다.

단일 책임 원칙을 어길 때 발생하는 또 다른 문제점은 재사용을 어렵게 한다는 것이다. DataViewer 클래스가 HTTP 연동을 위해서
HttpClient 패키지를 사용하고 화면에 데이터를 보여주기 위해 GuiComp 패키지를 사용한다면 이들 간의 관계는 아래와 같을 것이다
(HttpClient 패키지와 GuiComp 패키지가 각각 별도의 jar 파일로 제공).

![](/assets/srp2.png)

이때 데이터를 읽어 오는 기능이 필요한 DataRequiredClient 클래스를 만들어야 한다면 구현하기 위해 필요한 것은 DataViewer 클래스와
HttpClient jar 파일이다. 하지만 실제로는 DataViewer가 GuiComp를 필요로 하므로 GuiComp jar 파일까지 필요하다.
즉 실제 사용하지 않는 기능이 의존하는 jar 파일까지 필요한 것이다. 그러므로 단일 책임 원칙에 따라 아래와 같이 책임을 분리시켜야 한다.

![](/assets/srp3.png)


단일 책임 원칙을 지키기 위한 방법은 메서드를 실행하는 것이 누구인지 확인해 보는 것이다. 아래 그림에서 DataViewer 클래스는 display 메서드와
loadData 메서드를 제공하는데, GUIApplication은 display 메서드를 사용하고 DataProcessor는 loadData()를 사용한다고 해보자.

![](/assets/srp4.png)

GUIApplication이 화면에 표시되는 방식을 변경해야 할 경우, 변경되는 메서드는 DataViewer 클래스의 display 메서드이다. 반면에
DataProcessor가 읽어 오는 데이터를 String이 아닌 다른 타입으로 변경해야 할 경우, DataViewer의 loadData 메서드는 String이
아닌 DataProcessor가 요구하는 타입으로 변경될 가능성이 높다. 이렇게 클래스의 사용자들이 서로 다른 메서드들을 사용한다면 그들 메서드는
각각 다른 책임에 속할 가능성이 높고 따라서 책임 분리 후보가 될 수 있다.

## 개방 폐쇄 원칙
확장에는 열려 있고 변경에는 닫혀 있어야한다.

![](/assets/ocp2.png)

메모리에서 byte를 읽어 오는 기능을 추가해야 할 경우, ByteSource 인터페이스를 상속받은 MemoryByteSource 클래스를 구현함으로써 기능 추가가 가능하다.
그리고 새로운 기능이 추가되었지만, 이 새로운 기능을 사용할 FlowController 클래스의 코드는 변경되지 않는다. 즉 기능을 확장 하면서도 기능을 사용하는 기존 코드는
변경되지 않는 것이다.

OCP를 구현하는 또 다른 방법은 상속을 이용하는 것이다.

```java
public class ResponseSender {
    private Data data;
    public ResponseSender(Data data) {
        this.data = data;
    }
    
    public Data getData() {
        return data;
    }

    public void send() {
        sendHeader();
        sendBody();
    }

    protected void sendHeader() { 
        // 헤더 데이터 전송
    }
    
    protected void sendBody() {
        // 텍스트로 데이터 전송
    }

}
```

하위 클래스에서 sendHeader, sendBody 메서드를 오버라이딩 함으로써 기능 확장이 가능하다.

```java
public ZippedResponseSender extends ResponseSender {
    public ZippedResponseSender(Data data) {
        super(data);
    }
    
    @Override
    protected void sendBody() {
        // 데이터 압축 처리
    }
}
```

ZippedResponseSender 클래스는 기존 기능에 압축 기능을 추가해 주는데, 이 기능을 추가하기 위해 ResponseSender 클래스의 코드는 바뀌지 않았다.

### 개방 폐쇄 원칙이 깨질 때의 주요 증상

![](/assets/ocp3.png)

예를 들어 슈팅 게임을 개발하는 경우 다음과 같은 구조가 있다고 하자.
그런데 화면에 이들 캐릭터를 표시해주는 코드가 다음과 같다면 어떨까?

```java
public void drawCharacter(Character character) {    
    if (character instanceof Missile) {
        Missile missile = (Missile) character; // 타입 다운 캐스팅
    } else {   
        character.draw();
    }   
}
```
위 코드는 character 파라미터의 타입이 Missile인 경우 별도 처리를 하고 있다. 만약 위와 같이 특정 타입인 경우에 별도 처리를 하도록 drawCharacter 메서드를
구현한다면 drawCharacter 메서드는 Character가 확장될때 함께 수정된다. 즉 변경에 닫혀 있지 않은것이다. instanceof 와 같은 타입 확인 연산자가 사용된다면
해당 코드는 개방 폐쇄 원칙을 지키지 않을 가능성이 높다. 

## 리스코프 치환 원칙
리스코프 치환 원칙은 OCP을 받쳐 주는 다형성에 관한 원칙을 제공한다. 리스코프 치환 원칙은 다음과 같다. **상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.**
```java
public void someMethod(SuperClass sc){
  sc.someMethod();
}
```
someMethod()는 상위 타입인 SuperClass 타입의 객체를 사용하고 있는데, 이 메서드에 다음과 같이 하위 타입의 객체를 전달해도 someMethod()가 정상적으로 동작해야 한다는 것이 리스코프 치환 원칙이다.
`someMethod(new SubClass());`

**리스코프 치환 원칙을 지키지 않을 때의 문제점**
리스코프 치환 원칙을 설명할 때 자주 사용되는 대표적인 예가 직사각형 - 정사각형 문제이다. 
```java
public class Rectangle{
  private int width;
  private int height;
  
  public void setWidth(int width){
    this.width = width;
  }
  public void setHeight(int height){
    this.height = hight;
  }
  public int getWidth(){
    return width;
  }
  public int getHeight(){
    return height;
  }
}
```
정사각형을 직사각형의 특수한 경우로 보고 정사각형이 직사각형을 상속받도록 구현했다고 하자. 정사각형은 가로, 세로 길이가 모두 동일해야 하기 때문에 setWidth() 메서드와 setHeight() 메서드를 재정의해서 가로, 세로 값이 일치되도록 구현하였다.
```java
public class Square extends Rectangle{
  @Override
  public void setWidth(int width){
    super.setWidth(width);
    super.setHeight(width);
  }
  
  @Override
  public void setHeight(int height){
    super.setWidth(height);
    super.setHeight(height);
  }
}
```
이제 Rectangle 클래스를 사용하는 코드를 살펴보자. 이 코드는 높이와 폭을 비교해서 높이를 더 길게 만들어 주는 기능을 제공한다고 해보자. 
```java
public void increaseHeight(Rectangle rec){
  if(rec.getHeight() <= rec.getWidth()){
    rec.setHeight(rec.getWidth() + 10);
  }
}
```
increaseHeight() 메서드를 사용하는 코드는 메서드 실행 후에 width 보다 height의 값이 더 크다고 가정할 것이다. 그런데 increaseHeight() 메서드의 rec 파라미터로 Square 객체가 전달되면, 이 가정은 깨진다. Square의 setHeight() 메서드는 높이와 폭을 모두 같은 값으로 만들어 버리기 때문에 increaseHeight() 메서드를 실행하더라도 높이가 폭보다 길어지지 않게 된다.
```java
public void increaseHeight(Rectangle rec){
  if(rec instanceof Square)
      throw new CantSupportSquareException();
  
  if(rec.getHeight() <= rec.getWidth())
    rec.setHeight(rec.getWidth() + 10);
  
}
```
이 문제를 해소하기 위해 rec 파라미터의 실제 타입이 Square일 경우를 막는 instanceof 연산자를 사용할 수 있을 것이다. 하지만 instanceof 연산자를 사용한다는 것 자체가 리스코프 치환 원칙 위반이 되고 이는 increaseHeight() 메서드가 Rectangle의 확장에 열려 있지 않다는 것을 뜻한다.

리스코프 치환 원칙을 어기는 또 다른 흔한 예는 상위 타입에서 지정한 리턴 값의 범위에 해당되지 않는 값을 리턴하는 것이다. 예를 들어, 입력 스트림으로부터 데이터를 읽어와 출력 스트림에 복사해 주는 복사 기능은 다음과 같이 구현될 것이다.
```java
public class CopyUtil {
  public static void copy(InputStream is, OutputStream out){
    byte[] data = new byte[512];
    int len = -1;
    
    //InputStream.read() 메서드는 스트림의 끝에 도달하면 -1을 리턴
    while((len = is.read(data)) != -1){
      out.write(data,0,len);
    }
  }
}
```
InputStream의 read() 메서드는 스트림의 끝에 도달해서 더 이상 데이터를 읽어올 수 없을 경우 -1을 리턴한다고 정의되어 있고, CopyUtil.copy() 메서드는 이 규칙에 따라 is.read()의 리턴 값이 -1이 아닐 때까지 반복해서 데이터를읽어와 out에 쓴다. 그런데 만약 InputStream을 상속한 하위 타입에서 read() 메서드를 아래와 같이 구현하면 어떻게 될까?
```java
public class SatanInputStream implements InputStream{
  public int read(byte[] data){
    ...
    return 0; // 데이터가 없을 때 0을 리턴하도록 구현
  }
}
```
SatanInputStream의 read() 메서드는 데이터가 없을 때 0을 리턴하도록 구현했다. SatanInputStream 클래스의 사용자는 SatanInputStream 객체로부터 데이터를 읽어 와서 파일에 저장하기 위해 다음과 같이 CopyUtil.copy() 메서드를 사용할 수 있을 것이다. 
```java
InputStream is = new SatanInputStream(someData);
OutputStream out = new FileOutputStream(filePath);
CopyUtil.copy(is,out);
```
이렇게 되면 CopyUtil.copy() 메서드는 무한루프에 빠지게 된다. 왜냐하면 SatanInputStream의 read() 메서드는 데이터가 없더라도 -1을 리턴하지 않기 때문이다. 
```java
public class CopyUtil{
  public static void copy(InputStream is, OutputStream out){
    ...
    // is가 SatanInputStream인 경우 read() 메서드는 -1을 리턴하지 않으므로, 아래 코드는 무한루프가 된다.
    while((len = is.read(data)) != -1){
      out.write(data,0,len);
    }
  }
}
```
정리하면 위와 같은 문제가 발생하는 이유는 SatanInputStream 타입의 객체가 상위 타입인 InputStream을 올바르게 대체하지 않기 때문이다. 즉, 리스코프 치환 원칙을 지키지 않았기 때문에 문제가 발생한 것이다.

리스코프 치환 원칙은 확장에 대한 것이다. 리스코프 치환 원칙을 어기면 OCP를 어길 가능성이 높아진다. 상품에 쿠폰을 적용해서 할인되는 액수 구하는 기능 예를 살펴보자.
```java
public class Coupon{
  public int calculateDiscountAmount(Item item){
    return item.getPrice() * discountRate;
  }
}
```
이 코드에서 Coupon 클래스의 calculateDiscountAmount() 메서드는 Item 클래스의  getPrice() 메서드를 이용해서 할인될 값을 구하고 있다. 그런데 특수 Item은 무조건 할인을 해주지 않는 정책이 추가되어, 이를 위해 Item 클래스를 상속받는 SpecialItem 클래스를 추가했다고 하자.
```java
public class Coupon{
  public int calculateDiscountAmount(Item item){
    if(item instanceof SpecialItem)
      return 0;
      
    return item.getPrice() * discountRate;
  }
}
```
Item 타입을 사용하는 코드(위 예제에서는 calculateDiscountAmount 메서드)는 SpecialItem 타입이 존재하는지 알 필요 없이 오직 Item 타입만 사용해야 한다. 그런데 instanceof 연산자를 통해 SpecialItem 타입인지의 여부를 확인하고있다. 즉, 하위타입인 SpecialItem이 상위 타입인 Item을 완벽하게 대체하지 못하는 상황이 발생하고 있는 것이다.

**타입을 확인하는 기능을 사용한다는 것은 클라이언트가 상위 타입만을 사용해서 프로그래밍 할 수 없다는 것을 뜻하며, 이는 하위 타입이 상위 타입을 대체할 수 없다는 것을 의미한다. 즉, instanceof 연산자를 사용한다는 것 자체가 리스코프 치환 원칙 위반이 된다.**

위의 예제 같은 경우는 Item에 대한 추상화가 덜 되었기 때문에 리스코프 치환 원칙을 어기게 됐다. 따라서 상품의 가격 할인 가능 여부가 Item 및 그 하위 타입에서 변화되는 부분이며, 변화되는 부분을 Item 클래스에 추가함으로써 리스코프 치환 원칙을 지킬 수 있게 된다.
```java
public class Item{
  //변화되는 기능을 상위 타입에 추가 
  public boolean isDiscountAvailable(){
    return true;
  }
}

public class SpecialItem extends Item{
  @Override
  public boolean isDiscountAvailable(){
    return false;
  }
}
```
이렇게 변화되는 부분을 상위 타입에 추가함으로써, instanceof 연산자를 사용하던 코드를 Item 클래스만 사용하도록 구현할 수 있게 되었다.
```java
public class Coupon{
  public int calculateDiscountAmount(Item item){
    if(!item.isDiscountAvailable())
      return 0;
      
    return item.getPrice() * discountRate;
  }
}
```
## Null 객체 패턴
장기 고객 할인이라든가 신규 고객 할인과 같이 고객의 상태에 따라 특별 할인을 해준다고 가정해 보자. 사용 요금 명세서를 생성하는 기능은 아래 코드와 같이 명세서 상세 내역에 특별 할인 기능을 추가할 수 있을 것이다.
```java
public Bill createBill(Customer customer) {
  Bill bill = new Bill();
  //... 사용 내역 추가
  bill.addItem(new Item("기본 사용요금", price));
  bill.addItem(new Item("할부금", somePrice));
  
  // 특별 할인 내역 추가
  SpecialDiscount specialDiscount = specialDiscountFactory.create(customer);
  if (specialDiscount != null) { // 특별 할인 대상인 경우만 처리
    specialDiscount.addDetailTo(bill);
  }
}
```
고객에 따라 특별 할인이 없는 경우도 있기 때문에, 위 코드에서는 specialDiscount가 null이 아닌 경우에만 특별 할인 내역을 추가하도록 했다. null 검사 코드를 사용할 때의 단점은 개발자가 null 검사 코드를 빼 먹기 쉽다는 점이다.

**Null 객체 패턴은 null을 리턴하지 않고 null을 대신할 객체를 리턴함으로써 null 검사 코드를 없앨 수 있도록 한다.**
```java
public class NullSpecialDiscount extends SpecialDiscount {
  @Override
  public void addDetailTo(Bill bill) {
    // 아무 것도 하지 않음
  }
}
```
Null 객체 패턴은 위와 같이 null 대신 사용될 클래스를 구현한다. 이 클래스는 상위 타입을 상속받으며, 아무 기능도 수행하지 않는다.
```java
public class SpecialDiscountFactory {
  public SpecialDiscount create(Customer customer) {
    if (checkNewCustomer(customer))
      return new NewCustomerSpecialDiscount();
      
    //특별 할인 혜택이 없을 때, null 대신 NullSpecialDiscount 객체 리턴
    return new NullSpecialDiscount();
  }
}
```
그리고 위와 같이 null을 대체할 클래스의 객체를 리턴한다.
```java
public Bill createBill(Customer customer) {
  Bill bill = new Bill();
  //... 사용 내역 추가
  bill.addItem(new Item("기본 사용요금", price));
  bill.addItem(new Item("할부금", somePrice));
  
  // 특별 할인 내역 추가
  SpecialDiscount specialDiscount = specialDiscountFactory.create(customer);
  specialDiscount.addDetailTo(bill); // null 검사 불필요
}
```
이제 SpecialDiscountFactory.create() 메서드를 이용해서 특별 할인 내역을 처리하는 코드는 더 이상 null 검사를 할 필요가 없어진다.

Null 객체 패턴의 장점은 null 검사 코드를 사용할 필요가 없기 때문에 코드가 간결해진다는 점이다. 코드가 간결해진다는 것은 그 만큼 코드 가독성을 높여 주므로, 향후에 코드 수정을 보다 쉽게 만들어 준다.
