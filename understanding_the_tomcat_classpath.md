#Understanding The Tomcat Classpath 

원문 : https://www.mulesoft.com/tcat/tomcat-classpath

아파치 톰캣 유저 포럼에 많은 공통된 질문들이 있다. 바로 **웹 애플리케이션에 필요한 Jar 파일들을 포함시키기 위해 톰캣 classpath를 어떻게 설정해야 되는가**이다. 

어떻게 톰캣이 생성되고 classpath를 이용하는지 알아보자. 또한 classpath와 관련된 이슈들을 하나씩 살펴보자. 

##Why Classpaths Cause Trouble For Tomcat Users
classpath는 JVM에게 프로그램을 돌리기 위해 필요한 클래스들과 패키지들이 어디에 있는지 말해주는 argument다. classpath는 항상 프로그램 외부 소스에서 셋팅된다. 이렇게 프로그램으로부터 classpath에 대한 책임을 분리시키는 것은 자바 코드가 추상적인 방식으로 클래스들과 패키지들을 참조할 수 있게 한다. 

그런데 왜 많은 자바 개발자들이 classpath가 뭐고 어떻게 동작하는지 이해했음에도 불구하고 톰캣을 돌릴 때 문제가 생기는 걸까? 

이 질문에 대한 3가지 답이 있고 우리는 각각에 대해서 자세히 알아볼 것이다.

1. 톰캣은 다른 자바 프로그램들과 같은 방법으로 classpath를 resolve하지 않는다. 
2. 톰캣이 classpath를 reslove하는 방법은 모든 주요 릴리즈마다 조용히 바뀌어왔다. 
3. 톰캣 문서와 디폴트 설정은 어떤 일을 달성하는 best way을 강제한다. 만약 best way를 따르지 않으면 톰캣이 기술적으로 custom 설정을 지원해도 you're left in the dark. 그리고 outside dependencies, shared dependencies, 동일한 dependency의 여러 버전 같은 덜 공통적인 classpath 상황들에 대한 제어 방법을 제공하지 않는다.

##How Tomcat Classpath UsageDiffers From Standard Usage
그럼 톰캣 classpath 사용이 표준 사용과는 어떻게 다른지 알아보자. 

아파치 톰캣은 설정을 표준화하는 노력과 웹 애플리케이션 배포의 효율적인 관리를 위해 가능한 self-contained하고 직관적이고 자동적인것을 목표로 한다. 반면에 보안과 namespace 이유로 다른 라이브러리 접근에 제한을 건다.

**톰캣 start 스크립트는 "system" class loader를 만들 때 자바 classpath 환경변수를 무시하고 자신만의 classpath를 실행시킨다. 자바 classpath 환경변수(의존성 레파지토리들을 선언하는 전통적인 장소)를 사용하지 않는다.** 다시 말해, 아무리 시스템 환경변수에 추가적인 레파지토리를 선언해도 톰캣이 boot 될때마다 자신만의 환경변수로 그것을 덮어쓰게 된다. 

그렇다면 톰캣이 어떻게 classpath를 reslove하는지 이해하기 위해 startup process를 살펴보자. 

1. JVM bootstrap loader가 코어 자바 라이브러리들을 로드한다(JVM은 JAVA_HOME 변수를 사용하여 코어 라이브러리들을 찾는다).

2. Startup.sh는 "start" 파라미터와 함께 Catalina.sh를 호출해서 system classpath를 overwrites하고 bootstrap.jar와 tomcat-juli.jar를 로드한다. 이러한 리소스들은 톰캣에서만 볼 수 있다.

3. Class loader들은 각각 deployed Context로 만들어진다. deployed Context는 각 web 애플리케이션의 `WEB-INF/classes` 와 `WEB-INF/lib`에 있는 모든 클래스들과 JAR 파일들을 순서대로 로드한다. 이러한 리소스들은 그것들을 로드한 웹 애플리케이션에서만 볼 수 있다. 

4. The Common class loader는 `$CATALINA_HOME/lib`에 있는 모든 클래스들과 JAR 파일들을 로드한다. 이러한 리소스들은 톰캣과 모든 애플리케이션에서 볼 수 있다.

자바 애플리케이션의 standard location에서 하나의 속성이 설정된 한 classpath를 reslove 하는 것보다, 톰캣은 4개 이상의 속성들이 설정된 다수의 classpath들을 resolve한다(그중 단 하나만이 standard location에서 설정된 것이다). 

다음은 classpath resolution을 한 버전에서 다른 버전으로 바꿀 때 생기는 혼란에 대해서 살펴보자. 

##How Tomcat Class Loading Has Changed From Version To Version

이전 톰캣 버전에서 classloader hierachy는 약간씩 다르게 동작했다.

Tomcat 4.x와 그 이전에서 "server" loader는 Catalina 클래스들을 로딩하는 책임이 있었다. 지금은 commons loader가 제어한다. 

Tomcat 5.x에서 "shared" loader는 `$CATALINA_HOME/shared/lib` 디렉토리에 위치해서, 애플리케이션들 간의 공유되는 클래스들을 로딩하는 책임이 있었다. 하지만 공유되는 의존성들을 dependent Contexts에서 간단하게 복제하는 쪽으로 유저들을 이끌면서 Tomcat 6에서 버려졌다. 결국 이 loader 또한 Common loader로 대체됐다. 추가적으로 Tomcat 5.x는 모든 Catalina 컴포넌트들을 로드하는 Catalina loader를 포함했었고 지금은 다시 Common loader가 제어한다.

##When You Can't Do Things The "Best" Way
Tomcat을 documentation이 추천하는 대로만 사용하면 classpath와 관련된 문제는 발생하지 않는다. 

WAR들은 모든 라이브러리, 패키지들의 중복된 버전을 갖게 되고 standard Tomcat distribution에 포함되지 않은 여러 애플리케이션들 간에 JAR를 공유할 필요가 없다. 또한 외부 리소스를 호출할 필요도 없고, 웹 애플리케이션을 돌리기 위해 필요한 single JAR파일의 multiple 버전 같은 복잡한 상황도 발생하지 않는다. 

하지만 그렇게만 사용되지 않는게 현실이다. 이런 상황에 놓인 유저들에게는 `catalina.properties` 파일이 모든 문제의 답이다. 

##Configuring Tomcat Classpath Handling Via catalina.properties

다행히 default class loading methods 사용을 원치 않는 유저라면 톰캣 classpath option들을 하드코딩하지 않아도 된다. Catalina central properties 파일인 `$CATALINA_HOME/conf/catalina.properties`에서 읽어온다.

이 파일은 JVM이 제어하는 bootstrap loader 이외의 모든 loader들에 대한 설정을 포함하고 있고, 또 JVM이 제어하는 system loader는 톰캣 startup 스크립트에 의해 re-written된다. 

이 파일을 살펴보면 몇가지 알 수 있다. 
1. 서버와 Shared loader들은 그것 자체로 제거되지 않는다. 만약 속성들이 정의되지 않았다면 Commons loader가 제어한다.
2. 다양한 loader들에 의해 로드된 클래스들과 JAR파일들은 자동적으로 로드되지 않고, simple wildcard syntax를 그룹으로 간단히 지정된다. 
3. 이 파일에 외부 레파지토리를 명세할 수 없다.

server loader는 혼자 남지만 shared loader는 여전히 많은 유용한 애플리케이션들을 갖고 있다. (Note : shared loader는 Commons loader가 클래스 로딩을 끝낸 후에 start-up process에서 클래스들을 마지막으로 로드할 것이다.)

이제 톰캣 classpath에 대한 공통적인 문제를 어떻게 고쳐야 되는지 살펴보자.

##Problems, Solutions, and Best Practices

Problem: My application relies on an external repository, and I can't import it.

To make Tomcat aware of an external repository, declare the file in catalina.properties under the shared loader, using the correct syntax. Syntax will vary based on the type of file or repository you are attempting to configure: