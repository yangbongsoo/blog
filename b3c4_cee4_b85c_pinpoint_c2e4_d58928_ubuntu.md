**원본 : Yous님 블로그 https://yous.be/2015/05/05/using-pinpoint-with-docker/**

**사전준비**<br>
Quickstart 스크립트로 샘플 Pinpoint instance를 실행시키는 것이 이번 포스팅의 목표다. <br>

pinpoint : https://github.com/naver/pinpoint/tree/master/quickstart<br>
도커 : https://www.docker.com/

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



