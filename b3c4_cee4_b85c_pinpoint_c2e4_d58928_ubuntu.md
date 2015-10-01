**원본 : Yous님 블로그 https://yous.be/2015/05/05/using-pinpoint-with-docker/**

**사전준비**<br>
Quickstart 스크립트로 샘플 Pinpoint instance를 실행시키는 것이 이번 포스팅의 목표다. <br>

pinpoint : https://github.com/naver/pinpoint/tree/master/quickstart<br>
도커 : https://www.docker.com/

###요약(빠른설치)<br>
AWS EC2를 생성하고<br>
**cf) 주의!! AWS 1년 무료 t2.micro(메모리 1G)는 스펙이 딸려서 Pinpoint감당 못함 최소 t2.small로 해야함.**


```
sudo apt-get update
sudo apt-get install -y git wget curl procps net-tools
sudo wget -qO- https://get.docker.com/ | sh
sudo docker pull yous/pinpoint
sudo docker run -i -t -p 28080:28080 -p 28081:28081 -p 28082:28082 \
  --cap-add SYS_PTRACE --security-opt apparmor:unconfined yous/pinpoint bash
```
도커 안에서 아래 스크립트를 실행시킨다.

cf) 1.0.4버전으로도 설치 가능하다 
```
docker pull yous/pinpoint:1.0.4
docker run -it --rm -p 28080:28080 -p 28081:28081 -p 28082:28082 \
--cap-add SYS_PTRACE --security-opt apparmor:unconfined yous/pinpoint:1.0.4 bash
```

주의) 도커에서는 새롭게 터미널창을 켜서 web.sh , testapp.sh 실행이 안된다. attach해서 도커안으로 들어가도 동기화되버린다. 그래서 백그라운드로 실행시키는게 좋은듯.
```
quickstart/bin/start-hbase.sh
quickstart/bin/init-hbase.sh

quickstart/bin/start-collector.sh &
quickstart/bin/start-web.sh &
quickstart/bin/start-testapp.sh &
```
그럼 Web UI : http://address:28080 TetsAPP : http://address:28081를 통해 확인할 수 있다. 



---

###디테일한 설명 <br>

**요구사항**<br>
첫번째로 Docker를 설치한다. 
```
wget -qO- https://get.docker.com/ | sh
```
Docker가 정확하게 설치됐는지 확인해보자.
```
sudo docker run hello-world
```
디테일한 부분은 Docker의 installation guides를 참고하자.<br> https://docs.docker.com/installation/#installation

**Dockerfile 살펴보기**<br>
https://github.com/yous/pinpoint-docker <br>
여기서 Dockerfile을 볼 수 있다. 이제부터 Dockerfile을 한줄한줄 살펴보자. 

```
FROM debian
```
```FROM``` 명령어는 Docker의 기본 이미지를 세팅한다. 우리는 최신 debian 이미지를 사용한다. 

```
RUN echo 'deb http://http.debian.net/debian/ wheezy contrib' >> /etc/apt/sources.list
RUN apt-get update

```
```RUN``` 명령어는 Docker에서 Commands를 실행한다. 우리는 ```java-package```패키지를 위해 ```/etc/apt/sources.list```에  ```http://http.debian.net/debian/ wheezy contrib'```를 추가하고 ```apt-get update```를 실행한다. 

```
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y git wget curl procps net-tools
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y java-package fakeroot
```
```git```, ```wget```, ```curl```같은 기본 툴을 설치한다. 또한 뒤에  QuickStart script에서 사용되는 ps, netstat를 위해 ```procps```, ```net-tools```도 설치한다. 

```RUN DEBIAN_FRONTEND=noninteractive```은 Docker 이미지를 만들때 ```apt-get install```만을 위한 것임을 주의해라. 이것은 ```apt-get install``` 진행할 때 나오는 경고 메시지들을 막는다. 또한 `DEBIAN_FRONTED`을 아래와 같이 설정 할 수 있다.

```
ENV DEBIAN_FRONTED noninteractive
```
하지만 우리는 `ENV`를 설정(환경 변수 설정을 의미)하면 빌드가 끝난 뒤에도 이 설정이 남아있게 되므로 추가하지 않았다. 우리가 Docker 이미지를 `docker run -i -t ... bash`
로 실행 시킬 때 Docker는 interactive하기 때문에 `DEBIAN_FRONTED`설정은 틀리게 된다. 그러므로 항상 inline으로 설정해야한다. 이부분에 대한 더 자세한 내용은 아래 주소를 참고하자. <br>
https://github.com/docker/docker/issues/4032<br>

```
RUN useradd pinpoint -m
```
pinpoint의 설치 가이드를 보면 JDK6과 JDK7+를 설치해야한다. Java를 설치하기 위해 우리는 non-root user가 필요하다. 그래서 user `pinpoint`를 추가했고 `-m`으로  홈 디렉토리를 만든다. 추가된 user로 Java를 설치한다. 

cf) debian 계열(우분투)의 경우 useradd / adduser 모두 사용할 수 있지만 차이가 있다.<br>
useradd : 순수 계정만 생성해주고, 기본 셸인 sh가 할당된다.(홈 디렉토리 / 패스워드 등을 따로 설정해줘야함)<br>
adduser : 계정생성 및 비밀번호와 사용자 정보를 입력받아 계정을 생성하고, 사용자가 설정한 기본 셸을 사용자의 셸로 지정해주고 홈 디렉토리도 만들어 준다. <br>
-m 옵션 : 홈 디렉토리를 지정할 때 사용(-d 옵션과 쓰임)

```
WORKDIR /home/pinpoint
```
`WORKDIR`명령어는 명령이 실행될 디렉토리이다. 위의 명령어 후부터 모든 `RUN`명령어는 `/home/pinpoint`에서 실행한다. 

```
RUN wget --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" \
  http://download.oracle.com/otn-pub/java/jdk/6u45-b06/jdk-6u45-linux-x64.bin
RUN chown pinpoint jdk-6u45-linux-x64.bin
RUN su pinpoint -c 'yes | fakeroot make-jpkg jdk-6u45-linux-x64.bin'
RUN rm jdk-6u45-linux-x64.bin
RUN dpkg -i oracle-j2sdk1.6_1.6.0+update45_amd64.deb
RUN rm oracle-j2sdk1.6_1.6.0+update45_amd64.deb

RUN wget --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" \
  http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.tar.gz
RUN chown pinpoint jdk-7u79-linux-x64.tar.gz
RUN su pinpoint -c 'yes | fakeroot make-jpkg jdk-7u79-linux-x64.tar.gz'
RUN rm jdk-7u79-linux-x64.tar.gz
RUN dpkg -i oracle-j2sdk1.7_1.7.0+update79_amd64.deb
RUN rm oracle-j2sdk1.7_1.7.0+update79_amd64.deb
```
지금 Java SE 6 과 Java SE 7을 Docker에서 설치했다.`wget` 스크립트는 http://stackoverflow.com/questions/10268583/downloading-java-jdk-on-linux-via-wget-is-shown-license-page-instead/10959815#10959815 여기를 참고하자. 
Running `fakeroot make-jpkg ...`와 `dpkg -i ...`로 Java를 설치한다. 

```
ENV JAVA_6_HOME /usr/lib/jvm/j2sdk1.6-oracle
ENV JAVA_7_HOME /usr/lib/jvm/j2sdk1.7-oracle
ENV JAVA_HOME /usr/lib/jvm/j2sdk1.7-oracle
```
다음과 같이 환경변수를 추가한다. 

```
WORKDIR /usr/local/apache-maven
```
이제 Maven을 설치해야한다. 

```
ADD http://mirror.apache-kr.org/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz ./
ADD http://www.apache.org/dist//maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz.md5 ./
ADD http://www.apache.org/dist//maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz.asc ./
```
`ADD`명령어는 명시된 경로 URL로부터 새로운 파일이나 디렉토리나 원격 파일을 복사한다. 위의 라인은 Apache mirror를 통해 Maven 파일을 다운받는 것이다. 

```
RUN [ $(md5sum apache-maven-3.2.5-bin.tar.gz | grep --only-matching -m 1 '^[0-9a-f]*') = $(cat apache-maven-3.2.5-bin.tar.gz.md5) ]
```
`apache-maven-3.2.5-bin.tar.gz`과 `apache-maven-3.2.5-bin-tar.gz.md5`의 MD5 checksum을 매치시킨다. <br>
cf) MD5 checksum이란 다운이 이상없이 됐는지 확인하는 용도다. 자세한 설명 : http://mytory.net/archives/96/

```
RUN gpg --keyserver pgp.mit.edu --recv-key BB617866
RUN gpg --verify apache-maven-3.2.5-bin.tar.gz.asc apache-maven-3.2.5-bin.tar.gz
```
이것은 `apache-maven-3.2.5-bin.tar.gz`과 `apache-maven-3.2.5-bin.tar.gz.asc`의 signature를 확인한다.

```
RUN tar -zxf apache-maven-3.2.5-bin.tar.gz
ENV PATH $PATH:/usr/local/apache-maven/apache-maven-3.2.5/bin
RUN rm apache-maven-3.2.5-bin.tar.gz apache-maven-3.2.5-bin.tar.gz.md5 apache-maven-3.2.5-bin.tar.gz.asc
```
이제 Docker에 Maven 3.2.5를 설치하고 `PATH`에 Maven의 경로를 추가했다. 

```
RUN git clone https://github.com/naver/pinpoint.git /pinpoint
WORKDIR /pinpoint
RUN git checkout tags/1.0.5
RUN mvn install -Dmaven.test.skip=true
```
Pinpoint clone을 내려받고 버전을 1.0.5로 바꿔준다. 그리고 설치한다.
https://github.com/naver/pinpoint/releases/tag/1.1.1

```
WORKDIR quickstart/hbase
ADD http://archive.apache.org/dist/hbase/hbase-0.94.25/hbase-0.94.25.tar.gz ./
RUN tar -zxf hbase-0.94.25.tar.gz
RUN rm hbase-0.94.25.tar.gz
RUN ln -s hbase-0.94.25 hbase
RUN cp ../conf/hbase/hbase-site.xml hbase-0.94.25/conf/
RUN chmod +x hbase-0.94.25/bin/start-hbase.sh
```
Pinpoint 설치 후 HBase를 설치한다. 

```
VOLUME [/pinpoint]
```
`VOLUME`은 디렉토리의 내용을 컨테이너에 저장하지 않고 호스트에 저장하도록 설정한다. 

**Docker 실행하기**<br>
이제 Docker 이미지를 pull할 수 있다.
```
docker pull yous/pinpoint
```

그리고 이미지를 돌린다.
```
docker run -i -t yous/pinpoint bash
```

만약 Docker exit후에 컨테이너를 지우고 싶으면 아래와 같이 입력한다. 
```
docker run -i -t --rm yous/pinpoint bash
```
위의 명령어는 `which java`같은 몇가지 요구사항을 체크할 수 있게 하지만 성공적으로 QuickStart 스크립트를 돌릴 수 없다. 스크립트가 `netstat` 결과로 프로그램 이름을 체크 하기 때문에 몇가지 옵션을 붙여야 한다. 디테일한 자료 : https://github.com/docker/docker/issues/7276

```
docker run -i -t --cap-add SYS_PTRACE --security-opt apparmor:unconfined \
  yous/pinpoint bash
```
또한 QuickStart에서 28080,28081,28082 포트를 사용하므로 container 포트로 바인딩해주는게 필요하다. `-p`가 그 역할을 수행한다. 디테일한 자료 : https://docs.docker.com/userguide/dockerlinks/

```
docker run -i -t -p 28080:28080 -p 28081:28081 -p 28082:28082 \
  --cap-add SYS_PTRACE --security-opt apparmor:unconfined yous/pinpoint bash
```

**QuickStart**<br>
지금까지 Pinpoint를 위한 Docker 이미지를 만들었다. 이제 QuickStart 스크립트를 돌릴 수 있다. Pinpoint 가이드에서 언급한대로 몇개의 추가적인 스크립트를 돌려야한다. 

**Install & Start HBase**<br>
Download & Start : `quickstart/bin/start-hbase.sh`<br>
Initialize Tables : `quickstart/bin/init-hbase.sh`<br>

**Start Pinpoint Daemons**<br>
Collector : `quickstart/bin/start-collector.sh`<br>
Web UI : `quickstart/bin/start-web.sh`<br>
TestAPP : `quickstart/bin/start-testapp.sh`<br>

HBase와 3개의 데몬들을 돌리면 아래의 주소로 Pinpoint TestApp에 대한 APM을 확인할 수 있다.

Web UI : http://address:28080<br>
TetsAPP : http://address:28081<br>

**Stopping**<br>
HBase : `quickstart/bin/stop-hbase.sh`<br>
Collector : `quickstart/bin/stop-collector.sh`<br>
Web UI : `quickstart/bin/stop-web.sh`<br>
TestAPP : `quickstart/bin/stop-testapp.sh`<br>
