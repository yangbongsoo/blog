#Apache

cf) Apache Syntax error check
```
$sudo apachectl -t
```

##httpd.conf

```xml

ServerRoot "/usr" (mac에 기본적으로 깔려있는 apache 기준)

...

#Listen 12.34.56.78:80
Listen 80

...

LoadModule jk_module /private/etc/apache2/other/mod_jk.so

#'Main' server configuration 
#이 섹션의 지시문은 <main> 서버가 사용하는 값을 설정하며, 
#<VirtualHost> 정의에 의해 처리되지 않는 요청에 응답한다.
#또한 이 값은 나중에 파일에 정의 할 수있는 <VirtualHost> 컨테이너의 기본#값을 제공한다.
#이러한 지시어는 모두 <VirtualHost> 컨테이너 안에 나타날 수 있는데, 가상 #호스트가 정의 될 때 기본 설정은 무시된다.

ServerAdmin you@example.com
ServerName www.example.com:80

#문서를 제공할 디렉토리다. 기본적으로 모든 요청은 디렉토리에서 처리되지만
#심볼릭 링크와 별칭을 사용하여 다른 위치를 가리킬 수도 있다.
DocumentRoot "/abc/def/ght"

#default는 매우 제한적인 기능으로 구성한다.
#서버의 파일 시스템 전체에 대한 액세스를 거부한다. 
#아래의 다른 <Directory> 블록에서 웹 콘텐츠 디렉토리에 대한 액세스를 명시적으로 허용해야 한다.
<Directory />
    Options FollowSymLinks
    AllowOverride none
    Order deny,allow
    Deny from all
</Directory>

...

#StartServers : 아파치 구동 시 띄울 프로세스 갯수
#MinSpareServers, MaxSpareServers :
#부하가 적어서 MinSpareServers 개수 보다 적었을 경우 최소한 이 개수 만큼 유지하려고 아파치가 노력하고 
#부하가 증가하여 프로세스 개수가 많아질 경우에 MaxSpareServers 개수 이하로 줄이려고 아파치는 노력한다.
#MaxRequestsPerChild : 프로세스가 요청 받을 수 있는 맥시멈 수 (0일 경우엔 무한)
#MaxClients : 실행 가능한 최대 프로세스 갯수
<IfModule mpm_prefork_module>
    StartServers          5
    MinSpareServers       5
    MaxSpareServers      10
    #ServerLimit         2048
    MaxClients          484
    MaxRequestsPerChild   0
</IfModule>

...

############################
# JKConnector Configuation #
############################
<IfModule mod_jk.c>
   JkMount /*.ybs tomcat
   JkMount /*.jsp tomcat
   JkMount /jkmanager/* jkstatus
   JkMountCopy All
   JkLogFile "/var/log/apache2/mod_jk.log"
   JkShmFile "/var/log/apache2/mod_jk.shm"
   JkWorkersFile /private/etc/apache2/workers.properties

   <Location /jkmanager/>
        JkMount jkstatus
        Order deny,allow
        Deny from all
        Allow from 127.0.0.1
   </Location>
</IfModule>

###############################
# Virtual Hosts Configuration #
###############################
<VirtualHost *:80>
    ServerAdmin goodbs1000@gmail.com
    DocumentRoot /Users/yangbongsoo/Documents/myProject/target/deploy
    ServerName xxx.xxx.com
    <Directory "/Users/yangbongsoo/Documents/myProject/target/deploy">
            Options FollowSymLinks
            AllowOverride None
            Order allow,deny
            Allow from all
    </Directory>

    <Directory ~ "/\.svn/*">
    Order deny,allow
    Deny from all
   </Directory>
   
   <Directory ~ "/META-INF">
    Order deny,allow
    Deny from all
   </Directory>
   
   <Directory ~ "/WEB-INF">
    Order deny,allow
    Deny from all
   </Directory>

    RewriteEngine on
    RewriteRule  ^/(projectName)*(/)*$ /projectName/Main.jsp [R]
    JkMountCopy On
    JkMount /*.ybs tomcat
    JkMount /*.jsp tomcat
</VirtualHost>
```
**ServerRoot :** 서버의 설정, 에러, 로그파일들이있는 디렉토리 트리의 맨 위

**Listen :** 아파치를 디폴트가 아닌 특정 IP 주소 나 포트에 바인드 할 수도 있다(prevent Apache from glomming onto all bound IP addresses).

**LoadModule jk_module :** 본인 pc에 디폴트로 mod_jk가 없어서 so 파일을 구해다 other 디렉토리에 넣었다.

**VirtualHost :** 위와 같이 http.conf에서 직접 작업 하지 않고 Include /private/etc/apache2/extra/httpd-vhosts.conf 한 후 그곳에서 작업해도 된다.

##workers.properties
```xml
worker.list=tomcat

worker.tomcat.type=ajp13
worker.tomcat.port=8009
worker.tomcat.host=localhost
worker.tomcat.socket_timeout=100
worker.tomcat.connection_pool_timeout=100
#worker.tomcat.lbfactor=1

worker.list=jkstatus
worker.jkstatus.type=status
```

#Tomcat
참고문헌 : 자바 고양이 톰캣 이야기 (최진식 저)
##server.xml
```xml
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">

  <Listener className="org.apache.catalina.core.JasperListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" gcDaemonProtection="false" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />

  <Service name="Catalina">

    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
                URIEncoding="UTF-8"
               redirectPort="8443" />

    <!--
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" />
    -->
    
    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="8009" protocol="AJP/1.3"
                enableLookups="false"
                acceptCount="1000" debug="0"
                connectionTimeout="180000"
                useBodyEncodingForURI="true"
                maxPostSize="4194304"
                maxParameterCount="4000"
                disableUploadTimeout="true"
                redirectPort="8443" />


    <Engine name="Catalina" defaultHost="localhost">


      <!-- my Server Setting start -->

      <Host name="localhost"  appBase="webapps"
                  unpackWARs="true" autoDeploy="true"
                  xmlValidation="false" xmlNamespaceAware="false">

        <Context docBase="/xxx/xxx/xxx/xxx/xxxx" path="/myProject" />
        <Context docBase="/xxx/xxx/xxx/xxx/xxx" path="/monitor" reloadable="false" />
        <Context path="/managerAgent" debug="0" privileged="true" docBase="managerAgent.war" />

      </Host>
      <!-- my Server Setting end -->
    </Engine>
  </Service>
</Server>
```
**Server**

최상위 Element인 `<Server>`는 `<Service>` 모음으로, Shutdown 요청 처리를 위한 address와 port 속성을 가지고 있다. 각각 Shutdown 요청을 받기 위해 listen하는 IP address와 포트를 설정하며 기본값을 localhost와 8005이다. 만약 port 속성을 -1로 설정하면 Shutdown 포트 기능을 사용하지 않는다. 한 shuStdown 속성은 Shutdown 명령어(패스워드)를 설정한다. 기본 설정 값 'SHUTDOWN'인데 보안상 변경하는 것이 좋다.

**Service**

Server 하위에 있으며 Connector 모음이다. Service 속성은 className과 name 단 2개다.

**Engine**

defaultHost 속성은 Engine 하위에 속한 Host 가운데 하나이며 어떤 Host도 처리하지 않는 요청을 처리한다.

**Host**

Host Container는 가상 호스트 기능을 제공한다. Host 이름은 name 속성을 통해 설정한다. 만약 할당된 URL이 있다면 URL로 설정한다. 상위 Engine 내에 2개 이상의 Host가 구성되어 있다면 그중 1개가 defaultHost값이 되어야한다.
appBase는 Host의 애플리케이션 디렉토리다. 기본은 webapps다. autoDeploy 속성을 통해 appBase 내 변경 사항을 주기적으로 확인할 수 있다(기본 true). 하지만 운영 환경이라면 가급적 false로 설정하는 것이 좋다. unpackWARs는 WAR 파일을 풀어서 사용할지 여부를 설정하는 속성으로 기본 true이다.

**Context**

Host 내에 배포된 애플리케이션이다. reloadable 속성은 WEB-INF/classes 및 WEB-INF/lib 디렉토리에 변경이 발생할 때 자동 반영 여부를 결정하는 속성으로 기본 false다. true로 설정하면 빈번한 Tomcat 재기동을 피할 수 있어 개발 시에는 유용하지만 운영 시에는 적지 않은 오버 헤드를 동반하므로 적용에 신중해야 한다.





