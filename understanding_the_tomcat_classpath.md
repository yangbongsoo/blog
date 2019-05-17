# Understanding The Tomcat Classpath

원문 : [https://www.mulesoft.com/tcat/tomcat-classpath](https://www.mulesoft.com/tcat/tomcat-classpath)

아파치 톰캣 유저 포럼에 공통된 질문들이 있다. 바로 **웹 애플리케이션에 필요한 Jar 파일들을 포함시키기 위해 톰캣 classpath를 어떻게 설정해야 되는가**이다. 먼저 톰캣이 어떻게 생성되고 classpath를 이용하는지 알아보자. 그리고 classpath와 관련된 이슈들을 하나씩 살펴보자.

## Why Classpaths Cause Trouble For Tomcat Users

classpath는 JVM에게 프로그램을 돌리기 위해 필요한 클래스들과 패키지들이 어디에 있는지 말해주는 argument다. classpath는 항상 프로그램 외부 소스에서 셋팅된다. 이렇게 프로그램으로부터 classpath에 대한 책임을 분리시키는 것은 자바 코드가 추상적인 방식으로 클래스들과 패키지들을 참조할 수 있게 한다.

그런데 왜 많은 자바 개발자들이 classpath가 뭐고 어떻게 동작하는지 이해했음에도 불구하고 톰캣을 돌릴 때 문제가 생기는 걸까?

이 질문에 대한 3가지 답이 있고 우리는 각각에 대해서 자세히 알아볼 것이다.

1. 톰캣은 다른 자바 프로그램들과 같은 방법으로 classpath를 resolve하지 않는다. 
2. 톰캣이 classpath를 reslove하는 방법은 모든 주요 릴리즈마다 조용히 바뀌어왔다. 
3. 톰캣 문서와 디폴트 설정은 어떤 일을 달성하는 best way을 강제한다. 만약 best way를 따르지 않으면 톰캣이 기술적으로 custom 설정을 지원해도 you're left in the dark. 그리고 outside dependencies, shared dependencies, 동일한 dependency의 여러 버전 같은 덜 공통적인 classpath 상황들에 대한 제어 방법을 제공하지 않는다.

## How Tomcat Classpath UsageDiffers From Standard Usage

그럼 톰캣 classpath 사용이 표준 사용과는 어떻게 다른지 알아보자.

아파치 톰캣은 설정을 표준화하는 노력과 웹 애플리케이션 배포의 효율적인 관리를 위해 가능한 self-contained하고 직관적이고 자동적인것을 목표로 한다. 반면에 보안과 namespace 이유로 다른 라이브러리 접근에 제한을 건다.

**톰캣 start 스크립트는 "system" classloader를 만들 때 자바 classpath 환경변수를 무시하고 자신만의 classpath를 실행시킨다. 자바 classpath 환경변수\(의존성 레파지토리들을 선언하는 전통적인 장소\)를 사용하지 않는다.** 다시 말해, 아무리 시스템 환경변수에 추가적인 레파지토리를 선언해도 톰캣이 boot 될때마다 자신만의 환경변수로 그것을 덮어쓰게 된다.

그렇다면 톰캣이 어떻게 classpath를 reslove하는지 이해하기 위해 startup process를 살펴보자.

1. JVM bootstrap loader가 코어 자바 라이브러리들을 로드한다\(JVM은 JAVA\_HOME 변수를 사용하여 코어 라이브러리들을 찾는다\).
2. Startup.sh는 "start" 파라미터와 함께 Catalina.sh를 호출해서 system classpath를 overwrites하고 bootstrap.jar와 tomcat-juli.jar를 로드한다. 이러한 리소스들은 톰캣에서만 볼 수 있다.
3. Class loader들은 각각 deployed Context로 만들어진다. deployed Context는 각 web 애플리케이션의 `WEB-INF/classes` 와 `WEB-INF/lib`에 있는 모든 클래스들과 JAR 파일들을 순서대로 로드한다. 이러한 리소스들은 그것들을 로드한 웹 애플리케이션에서만 볼 수 있다.
4. The Common class loader는 `$CATALINA_HOME/lib`에 있는 모든 클래스들과 JAR 파일들을 로드한다. 이러한 리소스들은 톰캣과 모든 애플리케이션에서 볼 수 있다.

자바 애플리케이션의 standard location에서 하나의 속성이 설정된 한 classpath를 reslove 하는 것보다, 톰캣은 4개 이상의 속성들이 설정된 다수의 classpath들을 resolve한다\(그중 단 하나만이 standard location에서 설정된 것이다\).

다음은 classpath resolution을 한 버전에서 다른 버전으로 바꿀 때 생기는 혼란에 대해서 살펴보자.

## How Tomcat Class Loading Has Changed From Version To Version

이전 톰캣 버전에서 classloader hierachy는 약간씩 다르게 동작했다.

Tomcat 4.x와 그 이전에서 "server" loader는 Catalina 클래스들을 로딩하는 책임이 있었다. 지금은 commons loader가 제어한다.

Tomcat 5.x에서 "shared" loader는 `$CATALINA_HOME/shared/lib` 디렉토리에 위치해서, 애플리케이션들 간의 공유되는 클래스들을 로딩하는 책임이 있었다. 하지만 공유되는 의존성들을 dependent Contexts에서 간단하게 복제하는 쪽으로 유저들을 이끌면서 Tomcat 6에서 버려졌다. 결국 이 loader 또한 Common loader로 대체됐다. 추가적으로 Tomcat 5.x는 모든 Catalina 컴포넌트들을 로드하는 Catalina loader를 포함했었고 지금은 다시 Common loader가 제어한다.

## When You Can't Do Things The "Best" Way

Tomcat을 documentation이 추천하는 대로만 사용하면 classpath와 관련된 문제는 발생하지 않는다.

WAR들은 모든 라이브러리, 패키지들의 중복된 버전을 갖게 되고 standard Tomcat distribution에 포함되지 않은 여러 애플리케이션들 간에 JAR를 공유할 필요가 없다. 또한 외부 리소스를 호출할 필요도 없고, 웹 애플리케이션을 돌리기 위해 필요한 single JAR파일의 multiple 버전 같은 복잡한 상황도 발생하지 않는다.

하지만 그렇게만 사용되지 않는게 현실이다. 이런 상황에 놓인 유저들에게는 `catalina.properties` 파일이 모든 문제의 답이다.

## Configuring Tomcat Classpath Handling Via catalina.properties

다행히 default class loading methods 사용을 원치 않는 유저라면 톰캣 classpath option들을 하드코딩하지 않아도 된다. Catalina central properties 파일인 `$CATALINA_HOME/conf/catalina.properties`에서 읽어온다.

이 파일은 JVM이 제어하는 bootstrap loader 이외의 모든 loader들에 대한 설정을 포함하고 있고, 또 JVM이 제어하는 system loader는 톰캣 startup 스크립트에 의해 re-written된다.

이 파일을 살펴보면 다음과 같은 사실을 알 수 있다. 1. 서버와 Shared loader들은 그것 자체로 제거되지 않는다. 만약 속성들이 정의되지 않았다면 Commons loader가 제어한다. 2. 다양한 loader들에 의해 로드된 클래스들과 JAR파일들은 자동적으로 로드되지 않고, simple wildcard syntax를 그룹으로 간단히 지정된다. 3. 이 파일에 외부 레파지토리를 명세할 수 없다.

server loader는 혼자 남지만 shared loader는 여전히 많은 유용한 애플리케이션들을 갖고 있다. \(Note : shared loader는 Commons loader가 클래스 로딩을 끝낸 후에 start-up process에서 클래스들을 마지막으로 로드할 것이다.\)

이제 톰캣 classpath에 대한 공통적인 문제를 어떻게 고쳐야 되는지 살펴보자.

## Problems, Solutions, and Best Practices

**문제 : 애플리케이션이 외부 레파지토리를 의존하고 있는데 그걸 import 할 수가 없다.**

톰캣이 외부 레파지토리를 인식하려면 shared loader 아래의 `catalina.properties`에 syntax 맞게 선언해라.

* 클래스 레파지토리로서 폴더를 추가하려면 `path/to/foldername` 
* 클래스 레파지토리로서 폴더안에 JAR 파일들을 추가하려면  `path/to/foldername/*.jar`
* 클래스 레파지토리로서 단일 JAR 파일을 추가하려면 `file:/path/to/foldername/jarname.jar`
* 환경변수를 호출하려면 `${}`를 사용해라 ex\) `${VARIABLE_NAME}`
* 여러 리소스들을 선언하려면 각각의 entry를 콤마로 구분지어라. 
* 모든 경로들은 상대 경로로 `CATALINA_BASE` or `CATALINA_HOME`를 이용할 수 있고 아예 절대 경로를 사용할 수도 있다.

**문제 : 다수의 애플리케이션이 하나의 JAR 파일을 공유하길 원한다. 그리고 그 JAR 파일은 톰캣 안에 있길 원한다.**

`$CATALINA_HOME/lib`안의 JDBC 드라이버들과 같은, 공통적인 서드파티 라이브러리들 말고는 추가적인 라이브러리들을 포함하지 않는게 best다. 대신에 Tomcat 5.x에서 사용되는 `/shared/lib`과 `/shared/classes` 디렉토리를 만들고 catalina.properties에서 shared.loader 속성을 설정해라. `"shared/classes,shared/lib/*.jar"`

**문제 : 애플리케이션에 또다른 프레임워크와 함께 내장 톰캣 서버를 사용하고 있는데 애플리케이션에서 프레임워크 컴포넌트들을 접근하려고 할 때마다 classpath 에러가 발생한다.**

이 문제는 이번 주제 범위에서 다소 벗어나있지만 공통적인 classpath와 관련된 질문이다. 다음은 에러 원인에 대한 간략한 개요다.

Spring이나 Wicket 같은 프레임워크를 추가하는 애플리케이션에 내장된 톰캣은 프레임워크를 시작할 때, 애플리케이션의 `WEB-INF/lib` 디렉토리 안에 것을 로드하지 않고 System classloader를 사용해 코어 클래스를 로드한다.

이것은 톰캣이 독립형 애플리케이션 컨테이너로써 돌아가는 기본 동작이다. 그러나 내장형일 때는 리소스를 웹 애플리케이션에서 이용할 수 없게 되는 결과를 낳는다.

Java class loading is lazy. 즉, 어떤 클래스를 요청하는 첫 classloader는 그 라이프사이클의 나머지 클래스를 소유하고 있다. 만약 System classloader\(System classloader의 클래스들은 웹 애플리케이션을 볼 수 없다\)가 프레임워크 클래스를 처음으로 로드했다면 JVM은 classpath 에러를 발생시키는 원인이 되는 추가적인 클래스 인스턴스들을 막는다.

이 문제는 애플리케이션에 custom bootstrap classloader를 추가함으로써 해결할 수 있다. 웹 애플리케이션을 대신해서 적절한 라이브러리들을 로드하기 위해 custom bootstrap classloader를 설정해라. 그리고 나머지는 정상적으로 start-up 시키면 애플리케이션의 모든 classloader 충돌을 해결할 수 있다.

**문제 : WAR의 일부분으로 모든 의존성 패키지를 포함하는 standard 애플리케이션을 사용하고 있지만 여전히 클래스 definition error가 발생한다.**

이 문제는 잘못 구현된 빌드나 배포 프로세스를 포함해 여러가지에 의해 발생할 수 있지만 대부분 웹 애플리케이션의 디렉토리 구조 때문에 발생한다.

Java naming convention은 클래스 이름들이, 자신들이 저장되는 디렉터리 구조를 반영하도록 한다. 예를 들어 `com.mycompany.mygreat.class` 클래스는 `WEB-INF/classes/com/mycompany/` 디렉토리에 저장될 필요가 있다.

종종 코드에서 누락된 기간\(missing period\)은 classpath와 관련된거 같은 에러를 유발한다. 톰캣을 비난하기 전에 항상 가장 간단한 해결책부터 체크해라!

**문제 : 웹 애플리케이션의 다른 섹션들은 같은 JAR 라이브러리의 서로 다른 두개 버전을 사용해야 한다.**

이러한 상황은 단일 애플리케이션에서, 다른 버전의 라이브러리를 의존하는 다수의 웹 프레임워크를 사용할때 자주 발생한다.

이 문제는 serious pain이고 3가지 해결책이 있지만 그 중 어느것도 classpath 자체적으로 해결될순 없다.

우리가 이 문제를 여기에 포함시킨 이유는 몇몇 유저들이 프레임워크 JAR안에 포함된 Manifest 파일에서, 필요한 의존성에 대한 다른 버전의 classpath들을 대안으로 명세하는 것을 시도했기 때문이다.

이 feature가 Weblogic\(일부 유저가 톰캣을 사용하는 이유가 될 수 있는\) 같은 웹 애플리케이션 서버에 의해 지원되는 동안에, Manifest 파일을 이용해 classpath를 명세하는 것은 톰캣에서 지원되지 않으며 서블릿 스펙도 아니다.

이 문제를 해결하기 위해 4가지 접근방법이 있다. 하지만 그 중 어떤것도 단순히 classpath를 고치는 것으로 해결될 수 없고 그 중 어떤것도 고통에서 자유로울 순 없다.

첫째, 프레임워크 버전을 업데이트 해라.

둘째, 두개 이상의 custom classloader를 만들어라\(각 JAR당 하나씩\). 그리고 필요로하는 버전으로 클래스의 두개의 인스턴스를 만들기 위해 애플리케이션의 `WEB-INF/context.xml` 파일에 설정해라.

셋째, 프레임워크와 단일 JAR 파일에서 의존성을 패키징하기 위해 jarjar 유틸리티를 사용해라. 그러면 같이 함께 로드될 것이다. 이 방법은 이상적이진 않지만 동작은 할 것이다.

마지막으로, 이런 문제를 하루걸러 계속 다루게 된다면 OSGi 프레임워크를 구현하는것을 고려해라. 이 프레임워크는 특별할 상황을 위해 설계되었고 하나의 클래스의 여러 버전이 하나의 JVM에서 실행되어야만 한다.

