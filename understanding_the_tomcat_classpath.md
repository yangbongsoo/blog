참고 : https://www.mulesoft.com/tcat/tomcat-classpath

아파치 톰캣 유저 포럼에 많은 공통된 질문들이 있다. 바로 **'웹 애플리케이션에 필요한 Jar 파일들을 포함시키기 위해 톰캣 classpath를 어떻게 설정해야 되는가'**이다. 

어떻게 톰캣이 생성되고 classpath를 이용하는지 알아보자. 또한 classpath 관련된 이슈들을 하나하나 살펴보자. 

#Why Classpaths Cause Trouble For Tomcat Users
classpath는 JVM에게 프로그램을 돌리기 위해 필요한 클래스들과 패키지들이 어디에 있는지 말해주는 argument다. classpath는 항상 프로그램 자체 외부 소스에서 셋팅된다. 이렇게 프로그램으로부터 classpath에 대한 책임을 분리시키는 것은 자바 코드가 추상적인 방식으로 클래스들과 패키지들을 참조할 수 있게 한다. 

그런데 왜 많은 자바 개발자들이 classpath가 뭐고 어떻게 동작하는지 이해했음에도 불구하고 톰캣을 돌릴 때 문제가 생기는 걸까? 

이 질문에 대한 3가지 답이 있고 우리는 각각에 대해서 자세히 알아볼 것이다.

1. 톰캣은 다른 자바 프로그램들과 같은 방법으로 classpath를 resolve하지 않는다. 
2. 톰캣이 classpath를 reslove하는 방법은 매 주요 릴리즈마다 조용히 바뀌어왔다. 
3. 톰캣 문서와 디폴트 설정은 어떤 일을 달성하는 최고의 방법을 push한다. 만약 best way를 따르지 않으면 톰캣이 기술적으로 custom 설정을 지원해도 you're left in the dark. 그리고 outside dependencies, shared dependencies, 동일한 dependency의 여러 버전 같은 덜 공통적인 classpath 상황들에 대한 제어 방법을 제공하지 않는다.

#How Tomcat Classpath Usage Differs From Standard Usage
그럼 톰캣 classpath 사용이 표준 사용과는 어떻게 다른지 알아보자. 

아파치 톰캣은 **설정**을 표준화하는 노력과 웹 애플리케이션 배포의 효율적인 관리를 위해 가능한 self-contained하고 직관적이고 자동적인것을 목표로 한다. 반면에 보안과 namespace 이유로 다른 라이브러리 접근에 제한을 건다(먼저 설정에 대해서 자세히 알아보자).

##Tomcat Configuration
톰캣을 띄우고 서버를 돌리고 난 후 다음 step은 basic setting을 설정하는 것이다. 초기 설정 프로세스는 두개의 task(여기서 자세히 설명될것이다)로 구성되어 있다. 첫 번째는 톰캣 XML 설정파일을 편집하는 것이고, 두 번째는 적절한 환경 변수들을 정의하는 것이다. 

###XML 설정 파일
톰캣을 띄우고 돌리는데 가장 중요한 두개의 설정 파일은 `server.xml`과 `web.xml`이다. 이 파일들은 기본적으로 `TOMCAT-HOME/conf/server.xml`과 `TOMCAT-HOME/conf/web.xml`에 위치한다.
같은 설정을 두번 하지 않도록 해라. 

###Server.xml
`server.xml`은 메인 설정 파일이고 톰캣 startup 초기 설정을 명세하는 책임이 있다. `server.xml`파일의 요소들은 5가지 기본 카테고리에 속한다(Top Level Elements, Connectors, Containers, Nested Components, Global Settings). 
```
<!--  server.xml 의 root element, server의 shutdown port를 지정 함 -->
<Server port="8005" shutdown="SHUTDOWN">  
      |     <!--  server는 1개 이상의  service를 가질 수 있지만, 보통은 server.xml을 분리해서 관리-->
      +---<Service  name="Catalina">   <!-- service는 독립적인 톰캣의 서비스 이다. -->
                  |    <!-- Connector Client와 요청을 주고 응답을 받는 Interface이다. -->
                  +---<Connector port="8080" protocol="HTTP/1.1">
                  |    <!-- Connector 에는 HTTP와 AJP등이 있다. -->
                  +---<Connector port="8009" protocol="AJP/1.3" >  <!-- Apache Jserv Protocol -->
                  |    <!-- Engine은 적절한 Host로 처리를 넘기는 역할을 한다. -->
                  +---<Engine name="Catalina" defaultHost="localhost">
                              | <!-- Realm, Valve Component를 이용하면 Database연결, Single Sing On,
                              +---<Realm>              Access Log등 부가기능을 이용 할 수 있다. -->
                              |
                              +---<Valve>
                              | 
                              +---<Logger>
                              |   <!-- 가상 호스트를 정의한다. -->
                              +---<Host appBase="webapps">
                                          | <!-가상호스트에서 동작하는 하나의 웹 어플리케이션 이다. -->
                                          +---<Context path="" docBase="C:\workspace\project\wiki ">
                                          |
                                          +---<Valve>
                                          |
                                          +---<Realm>
                                          |
                                          +---<Logger>
```
###Top Level Elements
**Server**<br>
이 태그는 단일 톰캣 서버를 정의하고 Logger와 ContextManager 설정요소를 포함한다. 추가적으로 port, shutdown, className 속성을 지원한다. 

port 속성은 shutdown 명령을 위해서 사용된다. shutdown 속성은 문자열 명령어로, 특정 포트에 shutdown 할 트리거로 정의된다. className 속성은 어떤 자바 클래스 구현체가 사용될것인지 정의한다. 

**Service**<br>
Service 태그는 Server 태그 안에 있고, 같은 Engine 컴포넌트가 공유하는 하나 이상의 Connector 컴포넌트들을 포함하고 있다. 이 컴포넌트의 메인 기능은 이러한 컴포넌트들을 싱글 서비스로써 정의하는 것이다. 그리고 Service 태그의 name 속성은 로그 안에서 표현된다(ex Catalina). 

**Connectors**<br>
Service 태그에서 하나 이상의 Connector를 중첩함으로써, processing을 위한 싱글 Engine 컴포넌트에 이러한 포트로 요청 할 수 있다. 톰캣은 HTTP와 AJP Connector 둘다 지원한다. 

**HTTP Connector**<br>
HTTP/1.1 Connector이고 Catalina에 stand-alone 웹 서버 기능을 제공한다. 즉, servlet들과 JSP 외에도 Catalina는 요청을 위한 특정 TCP 포트를 사용할 수 있다는 걸 의미한다. 사용자가 정의한 각각의 Connector들은 싱글 TCP 포트 Catalina가 HTTP 요청을 들어야 한다. HTTP Connector들을 설정할 때, "minSpareThreads", "maxThreads", "acceptCount" 속성에 주의해라. 특히 maxThreads 속성은 중요하다. 이 속성은 최대 스레드(사용 가능한 스레드 수를 초과하는 요청을 제어하기 위해 생성된)개수를 제어한다. 이 속성값을 너무 낮게 잡으면 요청들이 서버 소켓 스택에 쌓이게 되고 다 차면 connection을 거부하게 된다. 

**AJP Connector**<br>
이 요소는 AJP protocol로 통신하고, 메인 역할은 아파치 설치와 톰캣 통합을 돕는 것이다. 이 기능을 쓰는 가장 공통된 이유는 톰캣 프론트의 정적 콘텐츠를 serve하기 위해 아파치를 사용할 때다. 이 기술은 동적 페이지 생성과 로드 밸런싱에 대한 더 많은 전력을 확보하기 위한 것이고, 빠른 성능이 필요하다면 고려해볼만 하다. AJP Connectors는 또한 아파치 톰캣 SSL 처리 기능을 표시 할 수 있다.

**Containers**<br>
이 요소들은 요청들이 올바르게 processing 되기 위해 Catalina에서 사용된다.

**Engine**<br>
Engine 태그는 Service 태그 안에서, 하나 이상의 Connector들과 결합할 때 사용되고 부모 service와 관련된 모든 요청들을 처리할 책임이 있다. Engine 태그는 Service 태그 안에서만 사용될 수 있으며 한 개만 허용된다.
"defaultHost" 속성에 주의해라. 

Pay close attention to the "defaultHost" attribute, which defines the Host element responsible for serving requests for host names on the server that are not configured in server.xml. This attribute must match the name of one of the Host elements nested inside the Engine element in question. Also, it's important to assign a unique, logical name to each of your Engine elements, using the "name" attribute. If a single Server element in your server.xml file includes multiple Service elements, you are required to assign a unique name to every Engine element.

**Host**<br>
This element, which is nested inside of the Engine element, is used to associate server network names with Catalina servers. This element will only function properly if the virtual host in question is registered with the managing DNS of the domain in question.
One of the most useful features of the Host element is its ability to contain nested Alias elements, which are used to define multiple network names that should resolve to the same virtual host.

**Context**<br>
This element represents a single web application, and contains path information for directing requests to the appropriate application resources. When Catalina receives a request, it attempts to match the longest URI to the context path of a given Context until it finds the correct element to serve the request. The Context element can have a maximum of one nested instance per element of the utility elements Loader, Manager, Realm, Resources, and WatchedResource. Although Tomcat allows you to define Contexts within "TOMCAT-HOME/conf/server.xml", this should generally be avoided, as these central configuration settings cannot be reloaded without restarting Tomcat, which makes editing Context attributes more invasive than necessary.