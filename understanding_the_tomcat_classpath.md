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
이 요소는 단일 톰캣 서버를 정의하고 Logger와 ContextManager 설정요소를 포함한다. 추가적으로 서버 요소는 port, shutdown, className 속성을 지원한다. 