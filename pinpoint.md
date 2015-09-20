#Pinpoint 

**pinpoint 기능**

1. 서버 맵;대상 서버와 연결된 다른 서버와의 관계 다이어그램
2. 스캐터;요청별 응답시간에 따른 그래프
3. request detail; 스캐터에서 선택된 요청의 스택트레이스 뷰 
4. 에러 발생 요청 표시; 에러가 발생한 요청 빨간색으로 표시
5. 서버의 jar목록 표시; 서버에 접속하지 않아도 관련된 jar 목록 확인 가능 

**구성**

![](pinpoint-architecture.png)
1. DB; HBase(하둡 분산 데이터베이스 기반) 
2. Web UI로 view적으로 보여줌 
3. Collector; Web UI를 보여주는것을 쌓아두는거 , 대상이 되는 타겟 서버의 정보를 HBase가 저장함 
4. Agent; 각각의 대상 서버에 pinpoint agent를 줘서 각각의 pinpoint agent가 collector의 데이터를 udp/tcp + thrift를 통해서 
보내주고 그걸로 디비에 저장하고 그걸 web ui가 보여줌 