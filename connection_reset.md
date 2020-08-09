# Connection reset 

## RST flag
RST : The Reset flag indicates that the connection should be rest, and must be sent if a segment is
received which is apparently not for the current connection. On receipt of a segment with the
RST bit set, the receiving station will immediately aobrt the connection. Where a connection is aborted,
all data in transit is considered lost, and all buffers allocated to that connection are released.

Reset flag는 connection이 reset되야 함을 나타내며, segment가 보기에 현재 connection을 위한게 아닌걸 받았을 때
보내져야 한다. RST 비트가 설정된 segment를 수신하면 즉시 연결을 중단한다. 연결이 중단된 경우, 전송 중인 모든 데이터는 손실된 것으로 간주되며
해당 연결에 할당된 모든 버퍼가 해제된다.

## Connection reset일 때 retry 로직 구현
apache HttpClients 인자로 retryHandler를 직접 등록하는 방법도 있고
```java
    httpClient = HttpClients.custom()
		.setRetryHandler(retryHandler())
        .setConnectionManager(connectionManager)
        .setDefaultConnectionConfig(connectionConfig)
        .setDefaultRequestConfig(requestConfig)
        .build();
```
```java
private static HttpRequestRetryHandler retryHandler() {
		return (exception, executionCount, context) -> {
			if (executionCount > 2) {
				return false;
			}

			if (exception instanceof SocketException && exception.getMessage().equals("Connection reset")) {
				return true;
			}

			return false;
		};
	}
```

retry 설정을 disable하지 않으면 기본적으로 DefaultHttpRequestRetryHandler가 등록된다(디폴트 retryCount는 3).
```java
    // Add request retry executor, if not disabled
    if (!automaticRetriesDisabled) {
        HttpRequestRetryHandler retryHandlerCopy = this.retryHandler;
        if (retryHandlerCopy == null) {
            retryHandlerCopy = DefaultHttpRequestRetryHandler.INSTANCE;
        }
        execChain = new RetryExec(execChain, retryHandlerCopy);
    }
```

## RST 재설정 세그먼트 발생 시나리오
 
![](/assets/rstflagex1.jpg)

**중복으로 SYN을 보냈을 때** <br> 
TCP A가 시퀸스 번호 200으로 SYN을 보냈는데 세그먼트 전송이 지연된다.<br>
TCP B는 지연된 SYN 세그먼트(시퀀스 번호 90)를 받았다(line 3). <br>
TCP B는 처음 수신된 SYN이므로 ACK를 전송하고, 초기 시퀀스 번호 500을 사용할 것임을 나타낸다.<br> 
그러나 TCP A는 TCP B에서 보낸 ACK 필드가 올바르지 않다는 것을 감지하고 SYN 세그먼트가 목적지에 도달하지 못했다고 생각해서 
Reset을 전송하여 세그먼트를 거부한다(line 5). <br>
TCP A는 세그먼트를 믿을 수있게 만들기 위해이 재설정 세그먼트에서 91이라는 시퀀스 번호를 사용한다. <br>
TCP B는 Reset 플래그가 설정된 세그먼트를 받고 LISTEN 상태로 다시 들어간다.<br>
TCP B는 자신의 새 시퀀스 번호를 사용하여 정상적인 방식으로 ACK로 응답한다.<br>
TCP A는 ISN(초기 시퀀스 번호)과 연결이 설정되었음을 확인한다.<br>

Connection Establishment를 논의할 때, 두개의 프로세스간 정상적으로 communication할 때와 한쪽이 crash날 때 둘다
봐야한다.

![](/assets/rstflagex2.jpg)

**한쪽이 crash나고 다시 가동될 때, 에러 복구 메커니즘** <br>
TCP A 와 TCP B가 정상적으로 동작하다가 TCP B가 segment를 보냈는데 TCP A에서 crash가 발생했다. <br>
TCP A는 close하고 다시 three way handshake를 위한 SYN을 보낸다. <br> 
TCP B는 자신이 synchronized 되어있다고 생각한다(잘 연결되어 있다고 생각). <br>
따라서 SYN 세그먼트를 수신하면 시퀀스 번호를 확인하고 문제가 생겼다는걸 알 수 있다.(line3) <br>
TCP B는 시퀀스 번호 150을 기대한다는 ACK를 다시 전송한다(line4). <br>
TCP A는 수신 된 세그먼트가 자신이 보낸거에 맞지 않다고 생각하고(자신은 three way handshake 첫 과정인 SYN을 보냈음) <br>
Reset 세그먼트를 보낸다(line5). <br> 
TCP B는 중단되고 close된다. <br>
TCP A는 이제 다시 three way handshake(line 7)로 연결을 시도 할 수 있다. <br>

또 다른 두가지 가능한 시나리오가 있다. 
 
![](/assets/rstflagex3.jpg)

TCP A가 crash났을 때 TCP A의 이벤트를 인식하지 못한 TCP B는 데이터를 포함하는 세그먼트를 전송한다. <br>
이 세그먼트를 수신하면 TCP A는 Reset을 전송한다(그러한 연결이 존재하는지 모르기 때문에). <br>

![](/assets/rstflagex4.jpg)

위 케이스는 둘 다 LISTEN 상태에서 시작한다. 이 때 위에서 봤던 중복 SYN 문제가 발생하고, TCP A는 Reset을 보낸다. <br>
TCP B는 다시 LISTEN 상태로 돌아간다. <br>

## TCP/IP Illustrated Vol 1 Second Edition 재설정 세그먼트(reset segment)
일반적으로 재설정은 참조 연결에 대해서 정확하지 않은 세그먼트가 도착할 때 TCP에 의해 보내진다.
참조 연결(reference connection)이라는 용어는 목적지 IP 주소와 포트 번호, 송신측 IP 주소와 포트 번호가 정의된 연결을 의미한다.
RFC 793은 이것을 소켓(socket)이라고 부른다. 재설정은 정상적으로 TCP 연결의 빠른 해제의 결과다. 재설정 세그먼트 사용의 예를
보기 위해 시나리오를 구성해보자.

### 존재하지 않는 포트에 대한 연결 요구
재설정 세그먼트가 생성되는 일반적 경우는, 연결 요구가 도착할 때 목적지 포트상에 프로세스가 대기하고 있지 않을 때다.
이것은 TCP에서 종종 발생한다. UDP의 경우, 사용되지 않고 있는 목적지 포트에 데이터그램이 도착하면 ICMP 목적지 접근 불가(포트 접근불가)가 생성된다.
TCP는 대신에 재설정 세그먼트(reset segment)를 사용한다.

이런 예를 발생시키는 것은 간단하다. 여기서는 텔넷 클라이언트를 사용해 목적지에서 사용되지 않고 있는 포트 번호를 지정한다.

```
$ telnet localhost 9999
Trying 127.0.0.1...
telnet: connect to address 127.0.0.1: Connection refused
telnet: Unable to connect to remote host: Connection refused
```
이 오류 메세지는 텔넷 클라이언트에 의해 즉시 출력된다. 아래는 이 명령에 대한 패킷 교환을 나타내고 있다.
```
1 22:15:16.348064 127.0.0.1.32803 > 127.0.0.1.9999:
    S [tcp sum ok] 3357881819:3357881819(0) win 32767

2 22:15:16.348105 127.0.0.1.9999 > 127.0.0.1.32803:
    R [tcp sum ok] 0:0(0) ack 3357881820 win 0
```
도착한 세그먼트의 ACK 비트는 설정돼 있지 않기 때문에 재설정의 순서 번호는 0으로 설정되고, 확인 응답 번호는 수신 ISN에
세그먼트의 데이터 바이트 수를 더한 값으로 설정돼 있다. TCP에 의해 받아들여진 재설정 세그먼트를 위해 ACK 비트 필드는 반드시
설정돼야 하고 ACK 번호 필드는 유효한 윈도우 내에 있어야 한다.

### 연결 중단

![](/assets/tcphandshake.png)

위의 그림처럼 연결을 종료하는 일반적인 방법은 상대편에 FIN 신호를 보내는 것이다. FIN은 큐에서 대기하고 있는 데이터를 모두
전송한 후에 보내지고, 데이터 손실도 전혀 없기 때문에 정규 해제(orderly release)라고 부른다. 그러나 FIN 대신 RST를 보냄으로써
연결을 중단하는 경우도 있다. 이것을 중단 해제(aboritive release)라고 부른다.

연결 중단에서는 애플리케이션에 두 가지 기능을 제공한다. (1) 대기 중인 모든 데이터를 폐기하고 즉시 재설정 신호를 전송한다.
(2) RST의 수신 측은 상대방에게 일반적인 연결 폐쇄가 아닌 중단을 했다고 알릴 수 있다. 애플리케이션에 의해 사용되는 API는
정상 폐쇄 대신 중단하는 방법을 제공해야만 한다.

소켓 API는 이 기능을 링거(linger)값 0을 가진 'linger on close'라는 소켓 옵션(SO_LINGER)을 사용해 제공한다.
링거 시간을 0으로 설정해 -L 옵션을 지정함으로써 연결이 종료될 때 일반적인 FIN 대신 중단이 보내진다.

다음 예에서는 원격에서 대용량의 출력을 발생시키는 명령이 들어오면 사용자에 의해 취소되는 것을 볼 수 있다.

```
$ ssh linux cat /user/share/dict/words
Aarhus
Aaron
Abada
Aback
Abaft
... continus ...
^C
Killed by signal 2.
```

여기서 사용자는 이 명령의 출력 중단을 결정해야 한다. 사용자가 인터럽트 문자를 입력하면 시스템은 프로세스(여기서는 ssh)가 신호 번호 2에
의해 종료됐음을 표시한다. 이 신호는 SIGINT로 부르며, 보통 이 신호가 전송되면 프로그램을 종료한다.

```
$ tcpdump -vvv -s 1500 tcp

1 22:33:06.386747 192.168.10.140.2788 > 192.168.10.14.ssh:
    S [tcp sum ok] 1520364313:1520364313(0) win 65535
    <mss 1460, nop, nop, sackOK>
    (DF) (ttl 128, id 43922, len 48)

2 22:33:06.386855 192.168.10.14.ssh > 192.168.10.140.2788:
    S [tcp sum ok] 181637276:181637276(0) ack 1520364314 win 5840
    <mss 1460, nop, nop, sackOK>
    (DF) (ttl 64, id 0, len 48)

3 22:33:06.387676 192.168.10.140.2788 > 192.168.10.14.ssh:
    . [tcp sum ok] 1:1(0) ack 1 win 65535
    (DF) (ttl 128, id 43923, len 40)

(... ssh 인증 교환을 암호화하고 벌크 데이터를 전송한다 ...)

4. 22:33:13.648247 192.168.10.140.2788 > 192.168.10.14.ssh:
    R [tcp sum ok] 1343:1343(0) ack 132929 win 0
    (DF) (ttl 128, id 44004, len 40)
```
세그먼트 1~3번은 일반적인 연결 확립을 나타내고 있다. 인터럽트 문자가 입력되면 연결은 중단된다. 재설정 세그먼트는
순서 번호와 확인 응답 번호를 포함하고 있다. 여기서 주의할 점은 재설정 세그먼트가 다른 종단으로부터 어떠한 응답도 얻지 못한다는 점이다.
이것은 전혀 확인 응답이 아니다. 재설정의 수신기는 연결을 중단하고 애플리케이션에게 연결이 재설정됐다는 것을 통보한다. 이것은 종종
Connection reset by peer 오류 표시나 유사한 메세지를 유발한다.

Connection reset by peer 는 remote server 에서 RST 패킷을 보낸것이고
일반적인 handshake 와 달리 즉시 connection 을 끊어버린다. 이것은 일반적인 half-closed 상태 전환을 우회한다.

Connection reset by peer 은 상대방과 통화 중 상대방이 전화기를 쾅 내려놓는거와 동일하다.
아무런 응답없이 계속 hang 걸린채로 내버려두는것보다는 예의바르다. 그러나 TCP/IP converseur 에서 기대되는 진정한 예의바른 FIN-ACK 가 아니다.

cf) https://stackoverflow.com/questions/1434451/what-does-connection-reset-by-peer-mean

### 절반 개방(half-open) 연결
TCP 연결에서 한쪽 종단이 상대방의 확인 없이 자신의 연결만을 폐쇄 또는 중단할 때 절반 개방(half-open)이라고 한다.
이것은 두 호스트 중 하나가 붕괴됐을 때 발생한다. 절반 개방 연결은 데이터 전송이 행해지지 않는 동안에는 가동되고 있는 한쪽 종단이
다른 쪽 종단의 붕괴 상태를 감지할 수 없다.

또 다른 일반적인 절반 개방이 발생하는 경우에서는 클라이언트 호스트가 애플리케이션을 종료하고 나서 클라이언트 호스트를 종료하지 않고
전력 공급을 중단한 경우다. 예를 들면 PC가 텔넷 클라이언트 상태로 동작 중일 때 사용자가 종료 시간이 돼서 PC의 전원을 꺼버린
경우를 생각 할 수 있다. 이 경우에 데이터의 전송이 없다면 서버는 클라이언트가 없어진 사실을 전혀 모르게 될 것이다.
새로운 텔넷 클라이언트를 실행하면 서버 호스트에는 새로운 서버가 동작된다. 이렇게 해서 서버 호스트상에 많은 절반 개방 TCP 연결이
생기게 된다.

쉽게 절반 개방 연결을 만들 수 있다. 이 경우에는 서버보다 클라이언트상에서 수행한다. 10.0.0.1 상에서 텔넷 클라이언트를 실행하고,
그것을 10.0.0.7 상의 sun rpc 서버에 접속한다. 그러고 나서 한 줄의 문자열을 입력한 후 그것을 tcpdump를 통해 살펴보고,
그런 다음 서버의 호스트에 있는 이더넷 케이블을 끊고, 서버 호스트를 재가동한다. 이에 따라 서버 호스트는 충돌하게 된다(서버를 재가동
하기 전에 이더넷 케이블을 끊는 것은 TCP가 보통 시스템을 종료할 때 행하는 개방 연결에 FIN을 전송하는 것을 막기 위해서다).

서버가 재가동된 후 케이블을 다시 접속하고, 클라이언트에서 서버로 또 다른 문자열을 전송해본다. 이때 서버의 TCP도 재실행되고,
재실행되기 전의 연결 정보는 모두 메모리에서 손실된 상태이기 때문에 데이터 세그먼트가 참조하는 연결에 대해 어떠한 정보도 알 수가 없게 된다.
TCP의 규칙은 수신자가 재설정으로 응답하는 것이다.
```
$ telnet 10.0.0.7 sunrpc
Trying 10.0.0.7...
Connected to 10.0.0.7.
Escape character is '^]'.
foo
(이더넷 케이블이 절단되고 서버가 재부트된다)
bar
Connection closed by remote host
```

### 시간_대기 감소(TWA)
TIME_WAIT 상태는 폐기된 폐쇄 연결로부터 남겨진 어떤 데이터그램을 허용하기 위한 것이다. 이 기간 동안 대기 TCP는 통상적으로 별로 할 일이 없다.
이것은 단지 2MSL 타이머가 종료될 때까지 상태를 유지한다. 따라서 이 기간 동안 연결로부터 확실한 세그먼트나 좀 더 특별한 RST 세그먼트를 수신하면 비동기화 될 수 있다.
이것을 TIME_WAIT 감소 TWA이라 부른다.

![](/assets/connection_reset_twa_image.png)

위 그림에서 보여준 예에서 서버는 연결에서 자신의 역할을 완료했고 어떤 상태를 클리어했다. 클라이언트는 TIME_WAIT 상태에 남아 있다. FIN 교환이 완료되면 클라이언트의 다음 순서 번호는 K이고
서버의 순서 번호는 L이다. 늦게 도착한 세그먼트는 서버로부터 클라이언트로 순서 번호 L-100을 사용하고 ACK 번호 K-200을 포함한다. 클라이언트가 이 세그먼트를 수신하면 이것은 순서 번호와
ACK 값 모두가 오래된 것이라고 결정한다. 이런 오래된 세그먼트를 수신하면 TCP는 가장 현재 순서 번호와 ACK 값(K와 L)을 가진 ACK를 전송함으로써 응답한다. 그러나 서버가 이 세그먼트를 수신하면
이것은 연결과 관련된 어떠한 정보도 갖고 있지 않기 때문에 RST 세그먼트로 응답한다.

이것은 서버를 위해서는 문제가 없지만, 클라이언트를 TIME_WAIT에서 CLOSED로 미리 천이(Transit)하게 만든다. 대부분의 시스템에서는 단순하게 TIME_WAIT 상태에서는 재설정 세그먼트에
동작하지 않음으로써 이 문제를 회피한다.

## TCP Dump
```
# cd /usr/sbin
# sudo ./tcpdump -i eth0 -vv -w ./dump.log  tcp port 80
```
option
```
# tcpdump -w tcpdump.log => 결과를 파일로 저장, txt 가 아닌 bin 형식으로 저장됨
# tcpdump -r tcpdump.log => 저장한 파일을 읽음
# tcpdump -i eth0 src 192.168.0.1 and tcp port 80	=> source ip 가 이것이면서 tcp port 80 인 패킷
```

client ip : 1.1.1.1 <br>
port : 3001 <br> 
```
// three way handshaking
23:34:31.893798 IP (tos 0x0, ttl 64, id 60919, offset 0, flags [DF], proto TCP (6), length 60) 
    bong.server.52986 > 1.1.1.1.3001: Flags [S], cksum 0xcaaa (correct), seq 1479370850, win 14600, options [mss 1460,sackOK,TS val 3970205077 ecr 0,nop,wscale 7], length 0
23:34:31.896765 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    1.1.1.1.3001 > bong.server.52986: Flags [S.], cksum 0x766b (correct), seq 3196372313, ack 1479370851, win 14480, options [mss 1460,sackOK,TS val 3970163747 ecr 3970205077,nop,wscale 7], length 0
23:34:31.896776 IP (tos 0x0, ttl 64, id 60920, offset 0, flags [DF], proto TCP (6), length 52)
    bong.server.52986 > 1.1.1.1.3001: Flags [.], cksum 0xdd51 (correct), seq 1, ack 1, win 115, options [nop,nop,TS val 3970205080 ecr 3970163747], length 0
    
23:34:31.896802 IP (tos 0x0, ttl 64, id 60921, offset 0, flags [DF], proto TCP (6), length 4396)
    bong.server.52986 > 1.1.1.1.3001: Flags [.], cksum 0x47ee (incorrect -> 0x1689), seq 1:4345, ack 1, win 115, options [nop,nop,TS val 3970205080 ecr 3970163747], length 4344
23:34:31.896810 IP (tos 0x0, ttl 64, id 60924, offset 0, flags [DF], proto TCP (6), length 1500)
    bong.server.52986 > 1.1.1.1.3001: Flags [.], cksum 0x3c9e (incorrect -> 0x2522), seq 4345:5793, ack 1, win 115, options [nop,nop,TS val 3970205080 ecr 3970163747], length 1448
23:34:31.896813 IP (tos 0x0, ttl 64, id 60925, offset 0, flags [DF], proto TCP (6), length 167)
    bong.server.52986 > 1.1.1.1.3001: Flags [P.], cksum 0x3769 (incorrect -> 0x5e2e), seq 5793:5908, ack 1, win 115, options [nop,nop,TS val 3970205080 ecr 3970163747], length 115
23:34:31.899658 IP (tos 0x0, ttl 54, id 8572, offset 0, flags [DF], proto TCP (6), length 52)
    1.1.1.1.3001 > bong.server.52986: Flags [.], cksum 0xc626 (correct), seq 1, ack 5908, win 136, options [nop,nop,TS val 3970163750 ecr 3970205080], length 0
23:34:31.901838 IP (tos 0x0, ttl 54, id 8573, offset 0, flags [DF], proto TCP (6), length 7292)
    1.1.1.1.3001 > bong.server.52986: Flags [.], cksum 0x533e (incorrect -> 0x145a), seq 1:7241, ack 5908, win 136, options [nop,nop,TS val 3970163752 ecr 3970205080], length 7240
23:34:31.901845 IP (tos 0x0, ttl 64, id 60926, offset 0, flags [DF], proto TCP (6), length 52)
    bong.server.52986 > 1.1.1.1.3001: Flags [.], cksum 0xa9d6 (correct), seq 5908, ack 7241, win 137, options [nop,nop,TS val 3970205085 ecr 3970163752], length 0
23:34:31.901850 IP (tos 0x0, ttl 54, id 8578, offset 0, flags [DF], proto TCP (6), length 1500)
    1.1.1.1.3001 > bong.server.52986: Flags [P.], cksum 0x04d4 (correct), seq 7241:8689, ack 5908, win 136, options [nop,nop,TS val 3970163752 ecr 3970205080], length 1448
23:34:31.901854 IP (tos 0x0, ttl 64, id 60927, offset 0, flags [DF], proto TCP (6), length 52)
    bong.server.52986 > 1.1.1.1.3001: Flags [.], cksum 0xa417 (correct), seq 5908, ack 8689, win 160, options [nop,nop,TS val 3970205085 ecr 3970163752], length 0
23:34:31.901856 IP (tos 0x0, ttl 54, id 8579, offset 0, flags [DF], proto TCP (6), length 1057)
    1.1.1.1.3001 > bong.server.52986: Flags [P.], cksum 0x6c59 (correct), seq 8689:9694, ack 5908, win 136, options [nop,nop,TS val 3970163752 ecr 3970205080], length 1005
23:34:31.901858 IP (tos 0x0, ttl 64, id 60928, offset 0, flags [DF], proto TCP (6), length 52)
    bong.server.52986 > 1.1.1.1.3001: Flags [.], cksum 0xa014 (correct), seq 5908, ack 9694, win 182, options [nop,nop,TS val 3970205085 ecr 3970163752], length 0
23:34:31.901934 IP (tos 0x0, ttl 64, id 60929, offset 0, flags [DF], proto TCP (6), length 52)
    bong.server.52986 > 1.1.1.1.3001: Flags [F.], cksum 0xa013 (correct), seq 5908, ack 9694, win 182, options [nop,nop,TS val 3970205085 ecr 3970163752], length 0
23:34:31.902096 IP (tos 0x0, ttl 54, id 8580, offset 0, flags [DF], proto TCP (6), length 52)
    1.1.1.1.3001 > bong.server.52986: Flags [F.], cksum 0xa046 (correct), seq 9694, ack 5908, win 136, options [nop,nop,TS val 3970163752 ecr 3970205080], length 0
23:34:31.902100 IP (tos 0x0, ttl 64, id 60930, offset 0, flags [DF], proto TCP (6), length 52)
    bong.server.52986 > 1.1.1.1.3001: Flags [.], cksum 0xa012 (correct), seq 5909, ack 9695, win 182, options [nop,nop,TS val 3970205085 ecr 3970163752], length 0
23:34:31.904735 IP (tos 0x0, ttl 54, id 8581, offset 0, flags [DF], proto TCP (6), length 52)
    1.1.1.1.3001 > bong.server.52986: Flags [.], cksum 0xa03d (correct), seq 9695, ack 5909, win 136, options [nop,nop,TS val 3970163755 ecr 3970205085], length 0
```
```
[S] - SYN (Start Connection)
[.] - No Flag Set
[P] - PSH (Push Data)
[F] - FIN (Finish Connection)
[R] - RST (Reset Connection)
[.]은 ACK를 뜻하며 [F.] 은 FIN+ACK 을 가리키는 싱글 패킷
```

출처 : TCP/IP The Ultimate Protocol Guide<br>
출처 : effective tcp/ip programming <br>
출처 : TCP/IP ILLustrated, Volume1 Second Edition
출처 : http://tech.kakao.com/2016/04/21/closewait-timewait/ <br>
출처 : http://multifrontgarden.tistory.com/46 <br>
