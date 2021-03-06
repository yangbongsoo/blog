## Story10 로그는 반드시 필요한 내용만 찍자

### System.out.println()의 문제점
대부분의 개발자들은 로그를 찍기 위해서 System.out.println() 메서드를 사용한 시스템 로그를 많이 사용한다.
가장 편하고, 확인하기 좋은 방법이지만 성능에 영향을 많이 주는 경우가 빈번히 발생한다.

왜 성능에 영향을 많이 줄까? 파일이나 콘솔에 로그를 남길 경우를 생각해 보자. 내용이 완전히
프린트되거나 저장될 때까지, 뒤에 프린트하려는 부분은 대기 할 수밖에 없다. 특히 콘솔에 로그를
남길 경우에는 더더욱 그렇다. 그렇게 되면 애플리케이션에서는 대기 시간이 발생한다. 이 대기 시간은
시스템의 속도에 의존적이다. 만약 디스크에 로그를 남긴다면, 서버 디스크의 RPM이 높을수록 로그의
처리 속도는 빨라질 것이다.

더 큰 문제는 System.out.println()으로 출력하는 로그가 개발할 때만 사용된다는 것이다.
운영할 때는 전혀 사용되지 않고, 볼 수도 없는 디버그용 로그를 운영 서버에서 고스란히 처리하고 있는 셈이다.

### 로그를 깔금하게 처리해주는 slf4j와 LogBack
기존의 로거들은 출력을 위해서 문자열을 더해 전달해 줘야만 했다. 하지만, slf4j는 format 문자열에
중괄호를 넣고, 그 순서대로 출력하고자 하는 데이터들을 콤마로 구분하여 전달해준다.  
```java
logger.error("message : {}", e.getMessage());
```
이렇게 전달해 주면 로그를 출력하지 않을 경우 필요 없는 문자열 더하기 연산이 발생하지 않는다.

예외처리를 할 때는 아래와 같이 예외 클래스에서 원하는 스택 정보를 가공하여 메세지 처리 하는것도 좋은 방법이다.
```java
if (logger.isErrorEnabled()) {
    StackTraceElement[] ste = exception.getStackTrace();
    StringBuffer str = new StringBuffer();
    int lastIndex = ste.length - 1;
    int count = 1;
    for (int i = lastIndex; i>lastIndex-3; i--) {
    	String className = ste[i].getClassName();
	    String methodName = ste[i].getMethodName();
	    int lineNumber = ste[i].getLineNumber();
    	String fileName = ste[i].getFileName();

	    str.append("\n").append("[" +count++ + "]")
    		.append("className :").append(className).append("\n")
	    	.append("methodName :").append(methodName).append("\n")
		    .append("fileName :").append(fileName).append("\n")
    		.append("lineNumber :").append(lineNumber).append("\n")
	    	.append("message :").append(exception.getMessage()).append("\n")
		    .append("cause :").append(exception.getCause()).append("\n");
    }
    logger.error(str.toString());
}
```
## 추가적으로 정리한 내용
### `if (logger.isErrorEnabled()) `와 같은 logging guard 필요성 이슈

slf4j는 parameterized logging이라고 불리는 advanced feature를 제공한다.
그리고 parameterized logging은 로깅 성능을 크게 향상시킨다.   

```java
logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
```
위와 같은 로그가 있을 때, 메세지 파라미터를 생성하는 비용(`i`와 `entry[i]`를 string으로 변환하고
다른 string들과 연결)이 발생한다. 이 작업은 메세지가 로깅되냐 안되냐에 관계없이 항상 발생한다.

파라미터 생성 비용을 피하는 한가지 방법은 아래와 같이 logging guard로 둘러치는 것이다.
```java
if(logger.isDebugEnabled()) {
  logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
}
``` 
이렇게 하면 로거에 대해 디버깅이 비활성화 된 경우, 파라미터 생성 비용이 들지 않는다.
하지만 로그 레벨을 DEBUG로 하게 되면, 로거 사용 여부를 평가하는 데 드는 비용이 발생한다.
한 번은 debugEnabled에 한 번, 다른 한 번은 디버그에 사용된다.

이것은 로거를 평가하는 데 실제로 문장을 기록하는 데 걸리는 시간의 1% 미만이기 때문에 
경미한 오버헤드다.

메세지 포맷에 따라 매우 편리한 대안이 있다.
entry가 객체라고 가정하면 다음과 같이 작성할 수 있다.
```java
Object entry = new SomeObject();
logger.debug("The entry is {}.", entry);
```
로그 여부를 평가한 후, 그리고 결정이 긍정적일 경우에만 로거 구현에서 메시지를 포맷하고
'{}' 쌍을 입력 문자열 값으로 바꾼다. 다시말해 이 포맷은 log 문이 disabled인 경우 파라미터 생성 비용을 발생시키지 않는다.

출처 : https://www.slf4j.org/faq.html#logging_performance

### 수반되는 메시지없이 예외를 기록 할 수 있나?
답은 No  

e가 예외 인 경우 ERROR 레벨에서 예외를 기록하려면 첨부된 메시지를 추가해야한다.
```java
logger.error("some accompanying message", e);
```

### exception/throwable이 있는 경우 로깅문을 파라미터화할 수 있나?
답은 Yes

slf4j 1.6.0 버전부터 되고 그 아래로는 안된다.
```java
String s = "Hello world";
try {
  Integer i = Integer.valueOf(s);
} catch (NumberFormatException e) {
  logger.error("Failed to format {}", s, e);
}
```
NumberFormatException stack trace가 출력될거다(1.6 이하 버전에서는 무시됨).

### jcl, slf4j, logback, log4j 관계
**jcl**은 apache.commons.logging의 줄임말이다(아파치 재단의 자카르타 프로젝트).
jcl은 엄밀히 말하면 로거가 아니다. 로깅을 하는일을 하지 않는다.
로깅 라이브러리가 아니라 로깅 추상화 라이브러리다. 로깅 라이브러리 선택권은 개발자의 것이다.
따라서 라이브러리나 프레임워크는 주로 로깅 추상화 라이브러리를 사용한다.
```java
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
```
jcl이 로깅 구현체를 찾는 방법
- 설정파일에서 찾기(그런데 설정파일 잘 안만든다)
- 애플리케이션 클래스패스에서 log4j 구현체 찾아보기(다른 구현체가 있으면 그거 쓰고 없으면 java.util.logging에 있는거 씀)
 
그런데 클래스 로더를 사용해서 클래스패스에 어떤 클래스가 있는지를 찾는 방법이 좋지 않음.

**slf4j**는 jcl과 같은 로깅 추상화 라이브러리인데 구현체를 찾는 방법이 jcl과 다르다.
컴파일 시점에 들어있는 의존성 정보로 판단해서 찾는다. 그래서 클래스 로더나 메모리 문제가
발생하지 않는다. 대신에 의존성 설정을 잘 해야 된다.

slf4j는 세 가지 모듈(api, binding, bridging)로 구성되어 있다.

api는 로깅 인터페이스를 의미한다.
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

logger.debug("The entry is {}.", entry);
```
```xml
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
</dependency>
```

binding은 api의 구현체로 연결해주는 일을 한다(어댑터 역할).
주의할점은 binding은 라이브러리나 프레임워크 개발자가 사용하는게 아니다. 애플리케이션
개발자가 사용하는거다. 그리고 **여러 binding 중 반드시 한 개만 사용해야 한다.**
```
slf4j-log4j12-{version}.jar
slf4j-jdk14-{version}.jar
slf4j-nop-{version}.jar
slf4j-jcl-{version}.jar
logback-classic-{logback-version}.jar
``` 

마지막으로 bridging모듈은 레거시를 위한 거다. slf4j가 없었을 때는 jcl을 쓰거나 log4j를 쓰거나 다른 로거를 썼을 것이다.
그런 호출을 slf4j 호출로 바꿔주는거다. 예를 들어 jcl 호출한 코드를 slf4j가 호출한것처럼 바꿔주려면 먼저 의존성에 jcl-over-slf4j.jar를 추가한다.
그러면 jcl 호출을 받아서 slf4j api를 호출한다(jcl 인터페이스를 구현하고 있음).

그리고 모든 로깅을 logback으로 하고 싶으면 아래 그림과 같이 설정하면 된다.

![](/assets/slf4j-logback.jpg)

출처 : 스프링 부트와 로깅 (백기선) https://www.youtube.com/watch?v=o2-JaRD9qQE

### logback.xml 설정

**로그레벨**<br>
① FATAL : 가장 크리티컬한 에러가 일어 났을 때 사용<br>
② ERROR : 일반 에러가 일어 났을 때 사용<br>
③ WARN : 에러는 아니지만 주의할 필요가 있을 때 사용<br>
④ INFO : 일반 정보를 나타낼 때 사용<br>
⑤ DEBUG : 일반 정보를 상세히 나타낼 때 사용<br>

cf) trace도 있는데 쓰는곳은 못봄(DEBUG 아래단계)

**Appender**<br>
로그를 출력 할 위치, 출력 형식 등을 설정
```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %-10contextName %logger{36} - %msg%n</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- daily rollover -->
            <fileNamePattern>/Users/yangbongsoo/logs/project/project.%d{yyyy-MM-dd}-${MYPID}.log</fileNamePattern>
            <!-- keep 30 days' worth of history -->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %-10contextName %logger{36} - %msg%n</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>
    
    <logger name="net.openhft.chronicle.map.TcpReplicator" level="DEBUG" >
    </logger>
    <logger name="net.openhft.chronicle" level="DEBUG" >
    </logger>    
    
    <root level="INFO">
        <appender-ref ref="CONSOLE" />        
    </root>
    ...
</configuration>
```
Logback-Core 모듈을 통해 사용할 수 있는 기본적 Appender는 3가지다 

**ConsoleAppender** : 로그를 OutputStream에 write 하여, 최종적으로 콘솔에 출력<br>
**FileAppender** : 로그의 내용을 지정된 File에 기록<br>
**RollingFileAppender** : FileAppender로 부터 상속받은 Appender로 날짜, 최대 용량 등을 설정하여 지정한 파일명 패턴에 따라 로그가 다른 파일에 기록되도록 한다. 
이를 이용하여 대량의 로그를 효과적으로 기록할 수 있다.

Logback-Core의 기본 Appender 외에도 Logback-Classic 모듈의 다양한 Appender (SSLSocketAppender, SMTPAppender, DBAppender 등)을 사용하여 로그를 원격위치에 기록 할 수도 있다.
Appender들의 하위 항목으로 출력 형식(Layout Pattern)을 지정하여 각 Appender마다 원하는 내용을 출력시킬 수 있다.
ex) %logger(Logger 이름), %thread(현재 스레드명), %level(로그 레벨), %msg(로그메시지), %n(new line) 등

**Logger** : 실제 로그 기능을 수행하는 객체로 각 Logger마다 Name을 부여하여 사용한다.
각 Logger 마다 원하는 출력 레벨값을 설정할 수 있으며, 0개 이상의 Appender를 지정할 수 있다. 
각 소스로부터 입력받은 로깅 메시지는 로그 레벨에 따라 Appender로 전달 된다.
기본적으로 최상위 로거인 Root Logger를 설정해 주어야하며, 추가로 필요한 로거에 대해 String 또는 클래스명 형식으로 Logger Name을 추가하여 사용할 수 있다.
또한 Logger의 Name은 .문자를 구분자로 사용하여 계층적으로 활용 할 수 있다.

출처 : https://thinkwarelab.wordpress.com/2016/11/18/java%EC%97%90%EC%84%9C-logback%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%A1%9C%EA%B9%85logging-%EC%82%AC%EC%9A%A9%EB%B2%95/ 