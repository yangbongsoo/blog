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

아파치 톰캣은 **설정**을 표준화하는 노력과 웹 애플리케이션 배포의 효율적인 관리를 위해 가능한 self-contained하고 직관적이고 자동적인것을 목표로 한다. 반면에 보안과 namespace 이유로 다른 라이브러리 접근에 제한을 건다.

**톰캣 start 스크립트는 "system" 클래스로더를 만들 때 자바 classpath 환경변수를 무시하고 자신만의 classpath를 실행시킨다. 자바 classpath 환경변수(의존성 레파지토리들을 선언하는 전통적인 장소)를 사용하지 않는다.** 다시 말해, 시스템 환경변수에 추카적인 레파지토리를 선언했을 때 톰캣이 boot 될때마다 자신만의 환경변수로 그것을 덮어써버리게 된다. 

톰캣이 어떻게 classpath를 reslove하는지 이해하기 위해 startup process를 살펴보자. 

1. JVM bootstrap loader가 코어 자바 라이브러리들을 로드한다(JVM은 JAVA_HOME 변수를 사용하여 코어 라이브러리들을 찾는다).

2. Startup.sh는 "start" 파라미터와 함께 Catalina.sh를 호출해서 system classpath를 overwrites하고 bootstrap.jar와 tomcat-juli.jar를 로드한다. 이러한 리소스들은 톰캣에서만 볼 수 있다.

3. Class loader들은 각각 deployed Context로 만들어진다. deployed Context는 각 web 애플리케이션의 `WEB-INF/classes` 와 `WEB-INF/lib`에 있는 모든 클래스들과 JAR 파일들을 순서대로 로드한다. 이러한 리소스들은 그것들을 로드한 웹 애플리케이션에서만 볼 수 있다. 

4. The Common class loader는 `$CATALINA_HOME/lib`에 있는 모든 클래스들과 JAR 파일들을 로드한다. 이러한 리소스들은 톰캣과 모든 애플리케이션에서 볼 수 있다.

There you have it. Rather than resolving one classpath configured in one attribute in the standard location for a Java application, Tomcat resolves multiple classpaths configured using 4 or more attributes, only one of which is configured in the standard location.

Ultimately, this is all designed to save you hassle, but if you're deviating from the standard use, it's easy to see how confusion can creep into the equation.

Next, let's look at another source of this confusion: changes to classpath resolution from version to version.