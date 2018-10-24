# Connection reset 

에러로그에서 exception 나오는 부분의 코드를 보면, socketRead 하다가
ConnectionResetException이 발생했다. socketRead 메서드 안의 socketRead0 메서드는
native라서 자바코드로는 더이상 디버깅할 수 없다. 정리해보면 예외가 발생했고 resetState가 CONNECTION_RESET_PENDING이 되고
곧 CONNECTION_RESET이 된다. 그리고 최종적으로 `throw new SocketException("Connection reset");`이 수행된다.
```java
class SocketInputStream extends FileInputStream {
    ...
    
    int read(byte b[], int off, int length, int timeout) throws IOException {
        
        ...
        
        boolean gotReset = false;
        
        // acquire file descriptor and do the read
        FileDescriptor fd = impl.acquireFD();
        try {
            n = socketRead(fd, b, off, length, timeout);
            if (n > 0) {
                return n;
            }
        } catch (ConnectionResetException rstExc) {
            gotReset = true;
        } finally {
            impl.releaseFD();
        }
        
        /*
         * We receive a "connection reset" but there may be bytes still
         * buffered on the socket
         */
        if (gotReset) {
            impl.setConnectionResetPending();
            impl.acquireFD();
            try {
                n = socketRead(fd, b, off, length, timeout);
                if (n > 0) {
                    return n;
                }
            } catch (ConnectionResetException rstExc) {
            } finally {
                impl.releaseFD();
            }
        }      
        
        /*
         * If we get here we are at EOF, the socket has been closed,
         * or the connection has been reset.
         */
        if (impl.isClosedOrPending()) {
            throw new SocketException("Socket closed");
        }
        if (impl.isConnectionResetPending()) {
            impl.setConnectionReset();
        }
        if (impl.isConnectionReset()) {
            throw new SocketException("Connection reset");
        }
    }
    ...
}
```

## RST flag
RST : The Reset flag indicates that the connection should be rest, and must be sent if a segment is
received which is apparently not for the current connection. On receipt of a segment with the
RST bit set, the receiving station will immediately aobrt the connection. Where a connection is aborted,
all data in transit is considered lost, and all buffers allocated to that connection are released. 

Reset flag는 connection이 reset되야 함을 나타내며, segment가 보기에 현재 connection을 위한게 아닌걸 받았을 때
보내져야 한다. RST 비트가 설정된 segment를 수신하면 즉시 연결을 중단한다. 연결이 중단된 경우, 전송 중인 모든 데이터는 손실된 것으로 간주되며 
해당 연결에 할당된 모든 버퍼가 해제된다.

![](/assets/tcphandshake.png)

일반적인 3 way handshake를 통해 커넥션을 맺고 4 way handshake를 통해 커넥션을 끊는다.

### RST 재설정 세그먼트 사용 예제
 
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

### TCP/IP ILLustrated 재설정 세그먼트(reset segment)
일반적으로 재설정은 참조 연결에 대해서 정확하지 않은 세그먼트가 도착할 때 TCP에 의해 보내진다.
참조 연결(reference connection)이라는 용어는 목적지 IP 주소와 포트 번호, 송신측 IP 주소와 포트 번호가 정의된 연결을 의미한다.
RFC 793은 이것을 소켓(socket)이라고 부른다. 재설정은 정상적으로 TCP 연결의 빠른 해제의 결과다. 재설정 세그먼트 사용의 예를
보기 위해 시나리오를 구성해보자.

**존재하지 않는 포트에 대한 연결 요구** <br>
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


cf) Connection reset은 read 시 상대방 socket이 close 된 경우이고 
Connection reset by peer: socket write error는 write 시 상대방 socket이 close된 경우다.



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
