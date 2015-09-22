**본문 : Yous님 블로그 https://yous.be/2015/05/05/using-pinpoint-with-docker/**

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
디테일한 부분은 Docker의 installation guides를 참고하자<br> https://docs.docker.com/installation/#installation

**Dockerfile 살펴보기**<br>
https://github.com/yous/pinpoint-docker <br>
여기서 Dockerfile을 볼 수 있다. 이제부터 Dockerfile을 한줄한줄 살펴보자 

```
FROM debian
```
```FROM``` 명령어는 
