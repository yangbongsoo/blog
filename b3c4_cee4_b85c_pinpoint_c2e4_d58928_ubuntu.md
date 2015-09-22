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
여기서 Dockerfile을 볼 수 있다. 이제부터 Dockerfile을 한줄한줄 살펴보자 

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

```RUN DEBIAN_FRONTEND=noninteractive```은 