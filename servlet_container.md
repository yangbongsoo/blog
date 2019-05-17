# 웹 프로그래머를 위한 서블릿 컨테이너의 이해
WAS는 클라이언트와 서버 간의 소켓 통신에 필요한 TCP/IP 연결 관리와 HTTP 프로토콜 해석 등의 네트워크 기반 작업을 추상화해 일종의 실행 환경을 제공한다. 따라서 웹 프로그래머는 TCP/IP 연결을 직접 생성하고 HTTP 프로토콜을 해석하는 과정을 생략해 웹을 쉽게 구현할 수 있다. 결국 관련 기술을 깊이 알지 못한 채로 WAS가 제공하는 추상화된 API로 네트워크에 접근한다. 

WAS가 "복잡한 실제 작업을 적절히 추상화해 사용하기 편하게 한다" 개념을 성공적으로 충족하면서, 이를 사용하는 웹 기반 프로그래밍은 오히려 별것 아니다는 평판을 얻었다. 하지만 웹 기반으로 제공되는 서비스의 영역은 지속적으로 넓어지고 있으며, 규모는 거대해지고 있다. WAS의 내부구조와 동작 원리를 이해하지 못하면 고성능, 고가용성에 대한 요구를 충족시킬 수 없다.

## 서블릿의 이해
원칙적으로 `javax.servlet.Servlet` 인터페이스를 구현한 것이 서블릿이다. 서블릿은 서블릿 컨테이너 내에 등록된 후 서블릿 컨테이너에 의해 생성, 호출, 소멸이 이뤄진다. 다시 말해, **서블릿은 독립적으로 실행되지 않고 main 메서드가 필요하지 않으며 서블릿 컨테이너에 의해 서블릿의 상태가 바뀐다.**

때때로 서블릿은 자신의 상태 변경 시점을 알아내 적절한 리소스 획득/반환 등의 처리를 해야 하므로 Servlet 인터페이스에 init/destroy 메서드가 정의된다.

![](/assets/servlet_container1.jpg)

### GenericServlet
```java
public abstract class GenericServlet 
    implements Servlet, ServletConfig, java.io.Serializable {

}
```
서블릿 인터페이스만 구현하면 서블릿 컨테이너가 생성, 소멸 등의 생명주기 관리작업을 수행할 수 있다. 그런데 서블릿 컨테이너가 서블릿 관리를 위해 필요한 기능은 서블릿 스펙에 모두 정의돼 있으므로 서블릿 명세는 서블릿에 필요한 구현을 미리 작성해 GenericServlet이란 이름으로 제공한다.

![](/assets/servlet_container2.jpg)

이 GenericServlet 클래스는 추상 클래스지만, service 메서드를 제외하고는 모두 구현된 일종의 서블릿을 위한 어댑터 역할을 제공한다. 따라서 서블릿을 작성하는 프로그래머들은 반복적으로 동일한(서블릿 컨테이너의 관리를 받으려고) 서블릿 인터페이스를 구현하는 대신 GenericServlet을 상속해서 사용한다.

### HttpServlet
```java
public abstract class HttpServlet extends GenericServlet {
}

```
일반적으로 서블릿이라 하면 거의 대부분 이 HttpServlet을 상속받은 서블릿을 의미한다. HttpServlet은 GenericServlet을 상속받으며, GenericServlet의 유일한 추상 메서드인 service를 HTTP 프로토콜 요청 메서드에 적합하게 재구현한 것이다.

![](/assets/servlet_container3.jpg)

서블릿 컨테이너는 받은 요청에 대해 서블릿을 선택한 후 Servlet 인터페이스에 정의된 service(ServletRequest, ServletResponse)를 호출한다. 그러면 클래스 상속 위계에 따라 그 처리가 부모 클래스인 GenericServlet에서 자식 클래스인 HttpServlet으로 넘어온다. HttpServlet의 service 메서드 내에서는 HTTP 요청 메서드에 의해 여러 doXXX 메서드로 분기돼 처리된다.

```java
    protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                if (ifModifiedSince < lastModified) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
            
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);
            
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
            
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
            
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
            
        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
            
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
```
### Apache Tomcat
Apache Tomcat은 오픈소스 서블릿 컨테이너다. 

cf) 자바고양이 톰캣이야기(최진식 저)
초창기 Tomcat은 Apache Jakarta 프로젝트 하위에 있었기 때문에 Jakarta Tomcat으로 불렸다(Jakarta 라는 이름은 과거 Sun Microsystems의 회의실 명에서 유래했다는 설이 있다). Catalina는 과거 Tomcat 코드네임이었고 지금은 Tomcat 코어 엔진을 뜻하는 단어다. Tomcat에서 가장 중요한 환경 변수 또한 `$CATALINA_HOME`과 `$CATALINA_BASE`다.

catalina.sh
```
# Add on extra jar files to CLASSPATH
if [ ! -z "$CLASSPATH" ] ; then
  CLASSPATH="$CLASSPATH":
fi
CLASSPATH="$CLASSPATH""$CATALINA_HOME"/bin/bootstrap.jar

if [ -z "$CATALINA_OUT" ] ; then
  CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out
fi

if [ -z "$CATALINA_TMPDIR" ] ; then
  # Define the java.io.tmpdir to use for Catalina
  CATALINA_TMPDIR="$CATALINA_BASE"/temp
fi

# Add tomcat-juli.jar to classpath
# tomcat-juli.jar can be over-ridden per instance
if [ -r "$CATALINA_BASE/bin/tomcat-juli.jar" ] ; then
  CLASSPATH=$CLASSPATH:$CATALINA_BASE/bin/tomcat-juli.jar
else
  CLASSPATH=$CLASSPATH:$CATALINA_HOME/bin/tomcat-juli.jar
fi
```

톰캣이 시작될 때 jar 파일 두 개만 클래스패스에 지정돼 있으므로 우리가 만든 XXXServlet을 찾을 방법이 필요하며 그것을 배치(deploy)라 한다. 다시말해 배치(deploy)는 서블릿을 포함한 웹 애플리케이션을 서블릿 컨테이너에 알려주는 과정이다.

톰캣 서버는 시작할 때 webapps 디렉토리 아래에서 새로운 웹 애플리케이션을 탐색한다. 

cf)
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <webappDirectory>deploy</webappDirectory>
    </configuration>
</plugin>
```

![](/assets/servlet_container4.png)

위와 같이 webapps 디렉토리 안에 deploy 디렉토리를 넣어주면 톰캣 서버는 디렉토리 안에 있는 웹 애플리케이션 구조를 자동으로 인식하므로 톰캣이 구동될 때 톰캣 서버에 배치(deploy)된다.

웹 애플리케이션은 서블릿 스펙에 정의된 특정한 구조를 가진 디렉토리 혹은 디렉토리 압축 파일이므로 서블릿 컨테이너는 해당 웹 애플리케이션이 가진 서블릿의 위치를 알 수 있다.

## 서블릿 컨테이너
### 부팅 과정에서 벌어지는 일들 
톰캣 서블릿 컨테이너 시작 클래스인 Bootstrap의 main 메서드이다.
```java
    /**
     * Main method and entry point when starting Tomcat via the provided
     * scripts.
     *
     * @param args Command line arguments to be processed
     */
    public static void main(String args[]) {

        if (daemon == null) {
            // Don't set daemon until init() has completed
            Bootstrap bootstrap = new Bootstrap();
            try {
                bootstrap.init();
            } catch (Throwable t) {
                handleThrowable(t);
                t.printStackTrace();
                return;
            }
            daemon = bootstrap;
        } else {
            // When running as a service the call to stop will be on a new
            // thread so make sure the correct class loader is used to prevent
            // a range of class not found exceptions.
            Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
        }

        try {
            String command = "start";
            if (args.length > 0) {
                command = args[args.length - 1];
            }

            if (command.equals("startd")) {
                args[args.length - 1] = "start";
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stopd")) {
                args[args.length - 1] = "stop";
                daemon.stop();
            } else if (command.equals("start")) {
                daemon.setAwait(true);
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stop")) {
                daemon.stopServer(args);
            } else if (command.equals("configtest")) {
                daemon.load(args);
                if (null==daemon.getServer()) {
                    System.exit(1);
                }
                System.exit(0);
            } else {
                log.warn("Bootstrap: command \"" + command + "\" does not exist.");
            }
        } catch (Throwable t) {
            // Unwrap the Exception for clearer error reporting
            if (t instanceof InvocationTargetException &&
                    t.getCause() != null) {
                t = t.getCause();
            }
            handleThrowable(t);
            t.printStackTrace();
            System.exit(1);
        }

    }
```
Bootstrap 클래스를 생성하고 init 메서드를 호출해 초기화한다. 멤버 변수인 daemon에 방금 생성한 Bootstrap 인스턴스를 할당한다. 이후 main 메서드의 args 배열로 전달받은 명령행 인자 값에 따라 Bootstrap 클래스의 인스턴스인 daemon의 load, start, stop 등의 메서드가 호출된다.

만약 예외가 발생하지 않았다면 main 메서드는 종료된다. 따라서 프로그램은 종료돼야 한다. 하지만 수많은 웹 사이트가 톰캣 서블릿 컨테이너로 서비스한다. 지금까지 지나쳐 온 여러 메서드(init, load, start 등)에 서블릿 컨테이너를 종료하지 않고 HTTP 요청을 받아들일 수 있게 소켓을 열고, 서블릿을 초기화하며 스레드를 관리하는 등 웹 서비스를 동작하게 하는 기능이 있으리라 짐작할 수 있을 것이다. 먼저 init 메서드를 알아보자.
```java
    /**
     * Initialize daemon.
     * @throws Exception Fatal initialization error
     */
    public void init() throws Exception {

        initClassLoaders();

        Thread.currentThread().setContextClassLoader(catalinaLoader);

        SecurityClassLoad.securityClassLoad(catalinaLoader);

        // Load our startup class and call its process() method
        if (log.isDebugEnabled())
            log.debug("Loading startup class");
        Class<?> startupClass =
            catalinaLoader.loadClass
            ("org.apache.catalina.startup.Catalina");
        Object startupInstance = startupClass.newInstance();

        // Set the shared extensions class loader
        if (log.isDebugEnabled())
            log.debug("Setting startup class properties");
        String methodName = "setParentClassLoader";
        Class<?> paramTypes[] = new Class[1];
        paramTypes[0] = Class.forName("java.lang.ClassLoader");
        Object paramValues[] = new Object[1];
        paramValues[0] = sharedLoader;
        Method method =
            startupInstance.getClass().getMethod(methodName, paramTypes);
        method.invoke(startupInstance, paramValues);

        catalinaDaemon = startupInstance;

    }
```
주목해야 하는 곳은 initClassLoaders 메서드다.
```java
    private void initClassLoaders() {
        try {
            commonLoader = createClassLoader("common", null);
            if( commonLoader == null ) {
                // no config file, default to this loader - we might be in a 'single' env.
                commonLoader=this.getClass().getClassLoader();
            }
            catalinaLoader = createClassLoader("server", commonLoader);
            sharedLoader = createClassLoader("shared", commonLoader);
        } catch (Throwable t) {
            handleThrowable(t);
            log.error("Class loader creation threw exception", t);
            System.exit(1);
        }
    }
```
이 메서드는 모두 세 개의 클래스로더를 생성한다. 생성하는 클래스로더는 common, server, shared다. common 클래스로더는 루트 클래스로더로, 부모가 없는 최상위 클래스로더다. server와 shared 클래스로더는 common 클래스로더를 부모 클래스로더로 가진다(shared나 server 클래스로더에 로딩된 클래스는 상위 클래스로더인 common 클래스로더에 로딩된 클래스에 접근할 수 있다). common, shared, server 클래스로더는 catalina.properties 파일에 정의된 디렉토리의 jar 파일을 읽어들여, 파일 안에 포함된 클래스를 로딩한다.
```
common.loader=${catalina.base}/lib,${catalina.base}/lib/*.jar,${catalina.home}/lib,${catalina.home}/lib/*.jar
...
server.loader=
...
shared.loader=
...
```
기본으로 제공되는 catalina.properties에는 common 클래스로더만 설치 디렉토리 아래 lib 디렉토리로 지정되어 있고, shared, server 클래스로더는 위치가 지정되어 있지 않다. 따라서 설정을 변경하지 않으면 shared나 server 클래스로더에 아무런 영향이 없다.

`Thread.currentThread().setContextClassLoader(catalinaLoader)`는 지금까지 설정한 클래스로더를 현재 클래스로더로 바꾼다. 이부분은 특별히 강조할 만한 가치가 있다. 앞서 살펴본 톰캣 구동 스크립트를 다시 한번 살펴보면 클래스 패스에 jar 파일 두 개만 설정됐었다(catalina.sh에서 bootstrap.jar와 tomcat-juli.jar).

그렇다면 jar 파일 이외의 라이브러리는 어떻게 사용할 수 있을까? 그보다 먼저 서블릿 인터페이스는 lib 디렉토리 아래 servlet-api.jar에 포함되어 있다. 클래스패스에 포함되어 있지 않다면 서블릿 컨테이너는 서블릿 인터페이스를 어떻게 알고 사용할 수 있을까?

웹 애플리케이션 하나가 서블릿 컨테이너 안으로 배치(deploy)되면 서블릿 컨테이너는 먼저 해당 웹 애플리케이션 전용 클래스로더를 생성한다. 그후 WEB-INF/classes 디렉토리와 WEB-INF/lib 디렉토리 내에 있는 클래스파일과 .jar 아카이브 파일들을 해당 클래스로더를 사용해 로딩한다. 그리고 Bootstrap 클래스의 initClassLoaders() 메서드를 통해 catalina.properties 파일에 정의된 디렉토리의 jar 파일들을 읽어들여 common 클래스로더를 로딩한다(여기에 servlet-api.jar가 있다). 그리고 웹 애플리케이션을 호출하는 HTTP 요청이 들어왔을 때 WEB-INF/web.xml에 정의된 서블릿 매핑에 따라서 웹 애플리케이션 클래스로더에서 서블릿을 찾아 요청을 전달(해당 인스턴스의 service 메서드 호출)한다.

cf) Understanding The Tomcat Classpath

https://www.mulesoft.com/tcat/tomcat-classpath
톰캣이 어떻게 classpath를 reslove하는지 이해하기 위해 startup process를 살펴보자.

1. JVM bootstrap loader가 코어 자바 라이브러리들을 로드한다(JVM은 JAVA_HOME 변수를 사용하여 코어 라이브러리들을 찾는다).

2. Startup.sh는 "start" 파라미터와 함께 Catalina.sh를 호출해서 system classpath를 overwrites하고 bootstrap.jar와 tomcat-juli.jar를 로드한다. 이러한 리소스들은 톰캣에서만 볼 수 있다.

3. Class loader들은 각각 deployed Context로 만들어진다. deployed Context는 각 web 애플리케이션의 WEB-INF/classes 와 WEB-INF/lib에 있는 모든 클래스들과 JAR 파일들을 순서대로 로드한다. 이러한 리소스들은 그것들을 로드한 웹 애플리케이션에서만 볼 수 있다.

4. The Common class loader는 $CATALINA_HOME/lib에 있는 모든 클래스들과 JAR 파일들을 로드한다. 이러한 리소스들은 톰캣과 모든 애플리케이션에서 볼 수 있다.

cf) 컴파일 때는 직접 의존성을 참조하고, 런타임 때는 톰캣에 있는 servlet-api.jar를 참조하도록 provided 스코프를 지정한다.
```xml
<dependency>
     <groupId>javax.servlet</groupId>
     <artifactId>servlet-api</artifactId>
     <version>3.0.1</version>
     <scope>provided</scope>
</dependency>
```
이제 실질적인 톰캣 서버 클래스인 카탈리나 클래스를 찾아 로딩을 시작한다.
```java
public void init() throws Exception {
 
// 생략 

Class<?> startupClass =
    catalinaLoader.loadClass
    ("org.apache.catalina.startup.Catalina”);
```
이후 Bootstrap 클래스의 load, start, stop 메서드가 불리지만 내부적으로는 catalinaDaemon, 즉 Catalina 클래스에 있는 같은 이름의 메서드가 호출된다. 카탈리나 클래스를 클래스로더에서 로딩한 후 startup 인스턴스를 만든다.
```java
Class<?> startupClass =
    catalinaLoader.loadClass
    ("org.apache.catalina.startup.Catalina");
Object startupInstance = startupClass.newInstance();
```

```java
public class Catalina { 
/**
 * Start a new server instance.
 */
public void start() {

    if (getServer() == null) {
        load();
    }

    if (getServer() == null) {
        log.fatal("Cannot start server. Server instance is not configured.");
        return;
    }

    long t1 = System.nanoTime();

    // Start the new server
    try {
        getServer().start();
    } catch (LifecycleException e) {
        log.fatal(sm.getString("catalina.serverStartFail"), e);
        try {
            getServer().destroy();
        } catch (LifecycleException e1) {
            log.debug("destroy() failed for failed Server ", e1);
        }
        return;
    }
```
이렇게 부팅 과정을 관리하는 클래스(Bootstrap)의 서버 역할을 수행하는 클래스(Catalina)를 분리해 놓으면 추후 서버 클래스 개선이 필요할 때 Bootstrap 클래스의 변경을 최소화하면서 서버 역할 수행 클래스를 교체할 수 있다.

### 생명주기 관리
톰캣 서블릿 컨테이너를 사용한 경험이 있는 웹 프로그래머라면 한 번쯤 config 디렉토리 안에 있는 XML 파일이 어떤 의미가 있으며, 어떤 시점에 어떻게 로딩되는지 의문을 가졌을 것이라 생각한다. 이런 설정 값을 읽어들이는 과정은 바로 Catalina 클래스의 load 메서드에서 찾아볼 수 있다.
```java
/**
 * Start a new server instance.
 */
public void load() {

    long t1 = System.nanoTime();

    initDirs();

    // Before digester - it may be needed
    initNaming();

    // Create and execute our Digester
    Digester digester = createStartDigester(); // 설정 파일을 개체화해 접근할 수 있도록 한다.

    InputSource inputSource = null;
    InputStream inputStream = null;
    File file = null;
    try {
        try {
            file = configFile(); //conf/server.xml
            inputStream = new FileInputStream(file);
            inputSource = new InputSource(file.toURI().toURL().toString());
        } catch (Exception e) {
	… 이하 생략
```
톰캣 서블릿 컨테이너는 Digester를 사용해 부팅 시 설정 파일을 객체화해 접근하는 방식을 지원한다. 그러므로 server.xml 파일이 로딩되면 XML 태크게 해당하는 객체가 생성된다. 톰캣 서블릿 컨테이너는 설정 파일에 지정된 객체들을 시동 시, 자동으로 생성될 뿐만 아니라 유지, 관리, 소멸 등의 생명주기를 관리함으로써 동작 중인 서버를 재시동하지 않고서 기능 변경을 하는 방법을 제공한다.

정리하면 사용자가 설정 파일에 정의한 각 XML element는 Digester에 의해 서버 객체로 변경돼 로딩되며, 이런 객체는 Lifecycle 인터페이스를 구현했으므로 프로그래밍적으로 초기화, 시작, 종료, 소멸 등을 컨트롤할 수 있다는 의미가 된다.
 
