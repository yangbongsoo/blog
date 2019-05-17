# Tshark

참고 : 티샤크를 활용한 네트워크 트래픽 분석\(와이어샤크의 커맨드라인 버전 TShark\)

와이어샤크 패키지 설치

```text
# yum install wireshark
```

root권한으로 실행시키지 않고 setcap으로 필요한 기능만을 부여

```text
# cd /usr/sbin
# sudo ./groupadd tshark
# sudo ./usermod -a -G tshark yangbongsoo
# sudo chgrp tshark /usr/sbin/dumpcap
# sudo chmod 750 /usr/sbin/dumpcap
# sudo ./setcap cap_net_raw,cap_net_admin=eip /usr/sbin/dumpcap
# sudo ./getcap /usr/sbin/dumpcap
/usr/sbin/dumpcap = cap_net_admin,cap_net_raw+eip
```

yangbongsoo 계정으로 다시 들어와서 트래픽을 캡쳐할 권한이 있는지 확인

```text
$ ./tshark -i eth0 -c 1 -q
Capturing on eth0
1 packet captured
```

와이어샤크와 마찬가지로 티샤크는 덤프캡을 이용해서 데이터를 수집한다. 덤프캡에는 기본적인 패킷 캡처 기능만 구현돼있으므로, 매우 복잡해서 취약점이 존재할 확률이 높은 와이어샤크나 티샤크 대신 덤프캡에 루트 권한을 주는게 훨씬 더 안전하다.

```text
$ ./tshark -V tcp port 80 -R "http.request || http.response" &
$ pstree -pa 'yangbongsoo'
nginx,155263
  ├─nginx,155264
  ├─nginx,155265
  ├─nginx,155266
  └─nginx,155267

bash,109967
  ├─pstree,110571 -pa yangbongsoo
  └─tshark,110535 -V tcp port 80 -R http.request\040||\040http.response
      └─dumpcap,110538 -n -i eth0 -f tcp\040port\04080 -Z none
```

위의 pstree 결과로 티샤크가 데이터를 캡처하기 위해 덤프캡을 자식 프로세스로 생성하는 것을 볼 수 있다.

```text
$ ./tshark -D
1. eth0
2. nflog (Linux netfilter log (NFLOG) interface)
3. nfqueue (Linux netfilter queue (NFQUEUE) interface)
4. any (Pseudo-device that captures on all interfaces)
5. lo
```

옵션 -D를 이용하면 시스템에서 이용 가능한 네트워크 인터페이스를 나열할 수 있고 -i를 이용하면 트래픽을 캡처할 리스닝 인터페이스를 지정할 수 있다. 티샤크는 수신된 패킷마다 기본 요약 정보를 출력한다.

```text
$ ./tshark -i eth0 -c 2
Capturing on eth0
0.000000000 10.xxx.xxx.xxx -> 10.xxx.xxx.xxx TCP 1012 45850 > biimenu [PSH, ACK] Seq=1 Ack=1 Win=501 Len=946 TSval=1734909355 TSecr=856327483
0.002694435 10.xxx.xxx.xxx -> 10.xxx.xxx.xxx TCP 66 biimenu > 45850 [ACK] Seq=1 Ack=947 Win=501 Len=0 TSval=856417489 TSecr=1734909355
2 packets captured
```

네크워크 카드에 패킷이 도달하고, 수신 데이터는 커널에 정의된 메모리 블록으로 복사된다. 패킷 필터는 사용자가 지정한 패킷만 필터링해서 버퍼에 저장한다. 저장된 패킷은 사용자 공간에서 실행 중인 덤프캡으로 전송되며 덤프캡은 이 패킷을 libpcap 파일 형식으로 기록한다. 끝으로 티샤크는 덤프캡이 작성한 캡처 파일을 읽고 처리한다.

커널은 수신된 패킷을 반드시 커널 공간에서 사용자 공간으로 복사해야 한다는 사실을 알아두자. 이와 같은 컨텍스트 스위칭은 CPU 시간을 소모하므로 네트워크 카드를 통과하는 모든 데이터 흐름을 캡처하면 시스템 전체의 성능이 저하될 수 있다. 바로 이때문에 캡처 필터가 필요하다. 캡처 필터를 사용하면 커널 공간에서 불필요한 패킷은 제외시키고 사용자가 관심 있는 패킷만 허용함으로써 성능 저하를 최소화할 수 있다.

필터는 -f 옵션을 사용해서 지정할 수 있다.

```text
$ ./tshark -f "tcp port 80" -i eth0
```

캡처 필터를 티샤크의 핵심인 디스플레이\(또는 리드\) 필터와 혼동하면 안된다. 디스플레이 필터는 이미 캡처된 패킷을 필터링하는 데 사용된다. 이 필터를 사용하면 프로토콜의 각 필드를 디코딩 및 해석하는 디섹터의 활용도를 극대화할 수 있다.

디스플레이 필터는 -R 옵션으로 지정할 수 있다.

```text
$ ./tshark  -f "tcp port 80" -i eth0 -R "http.request || http.response" -V
```

-V 옵션은 add output of packet tree\(Packet Details\)

```text
$ ./tshark  -f "tcp port 80" -i eth0 -R "http.request || http.response" -V | grep "Hypertext Transfer Protocol" -A 21
```

grep 해서 필요한부분만 추출할수도 있다.

```text
--
Hypertext Transfer Protocol
    GET /api/static/real.js HTTP/1.1\r\n
        [Expert Info (Chat/Sequence): GET /api/static/real.js HTTP/1.1\r\n]
            [Message: GET /api/static/real.js HTTP/1.1\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /api/static/real.js
        Request Version: HTTP/1.1
    CHECK: check\r\n
    Host: 10.xxx.xxx.xxx\r\n
    Connection: close\r\n
    Pragma: no-cache\r\n
    Cache-Control: no-cache\r\n
    Upgrade-Insecure-Requests: 1\r\n
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36\r\n
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8\r\n
    Accept-Encoding: gzip, deflate\r\n
    Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7\r\n
    \r\n
    [Full request URI: http://10.xxx.xxx.xxx/api/static/real.js]

--
Hypertext Transfer Protocol
    HTTP/1.1 200 OK\r\n
        [Expert Info (Chat/Sequence): HTTP/1.1 200 OK\r\n]
            [Message: HTTP/1.1 200 OK\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Version: HTTP/1.1
        Status Code: 200
        Response Phrase: OK
    Server: nginx\r\n
    Date: Thu, 11 Jan 2018 06:23:28 GMT\r\n
    Content-Type: application/javascript; charset=UTF-8\r\n
    Transfer-Encoding: chunked\r\n
    Connection: close\r\n
    X-Powered-By: Express\r\n
    Access-Control-Allow-Origin: http://node.test.com\r\n
    Vary: Origin\r\n
    Access-Control-Allow-Credentials: true\r\n
    Cache-Control: public, max-age=0\r\n
    Last-Modified: Thu, 21 Dec 2017 07:03:54 GMT\r\n
    ETag: W/"14856f-16077e2b57a"\r\n
    Content-Encoding: gzip\r\n
--
```

tshark의 리드필터\(-R 옵션\)로 지정할 수 있는 HTTP 프로토콜의 필드 목록을 확인하는 방법은 아래와 같다.

```text
$ ./tshark -G | cut -f3 | grep "^http\."
http.notification
http.response
http.request
http.authbasic
http.request.method
http.request.uri
http.request.version
http.request.full_uri
http.response.code
http.response.phrase
http.authorization
http.proxy_authenticate
http.proxy_authorization
http.proxy_connect_host
http.proxy_connect_port
http.www_authenticate
http.content_type
http.content_length_header
http.content_length
http.content_encoding
http.transfer_encoding
http.upgrade
http.user_agent
http.host
http.connection
http.cookie
http.accept
http.referer
http.accept_language
http.accept_encoding
http.date
http.cache_control
http.server
http.location
http.sec_websocket_accept
http.sec_websocket_extensions
http.sec_websocket_key
http.sec_websocket_protocol
http.sec_websocket_version
http.set_cookie
http.last_modified
http.x_forwarded_for
```

cf\) http2 필드는 1.12.0버전부터 가능하다.

전체 IP 통신의 목록 구하기

```text
$ ./tshark -r ~/tshark-log/temp.pcap -q -z "conv,ip,ip.addr==10.xxx.xxx.xxx" -w ~/tshark-log/write.pcap
================================================================================
IPv4 Conversations
Filter:ip.addr==10.xxx.xxx.xxx
                                               |       <-      | |       ->      | |     Total     |   Rel. Start   |   Duration   |
                                               | Frames  Bytes | | Frames  Bytes | | Frames  Bytes |                |              |
10.xxx.xxx.xxx        <-> 10.10.10.10             207     14144      80    371276     287    385420     4.065868087         0.0994
10.xxx.xxx.xxx        <-> 10.10.10.10             44    382591      39      3071      83    385662     4.069769413         0.0772
10.10.10.10       <-> 10.xxx.xxx.xxx              8     11380      10       668      18     12048    11.738211339         0.0106
10.xxx.xxx.xxx        <-> 10.10.10.10               7       595       6     11248      13     11843    11.731682788         0.0066
10.10.10.10         <-> 10.xxx.xxx.xxx              1      1012       1        66       2      1078     4.220985502         0.0028
10.xxx.xxx.xxx        <-> 10.10.10.10                1        90       1        90       2       180     4.051242000         0.0008
10.xxx.xxx.xxx        <-> 10.10.10.10                1       106       0         0       1       106    15.104299993         0.0000
================================================================================
```

위에서 사용된 옵션들

```text
-q : be more quiet on stdout (e.g. when using statistics)
-z <statistics>          various statistics, see the man page for details
-r <infile>              set the filename to read from (no stdin!)
-w <outfile|->           write packets to a pcap-format file named "outfile"
                          (or to the standard output for "-")
```

저장된 write.pcap 파일 읽기

```text
$ ./capinfos ~/tshark-log/write.pcap
File name:           /home/yangbongsoo/tshark-log/temp.pcap
File type:           Wireshark - pcapng
File encapsulation:  Ethernet
Packet size limit:   file hdr: (not set)
Number of packets:   467
File size:           817148 bytes
Data size:           801043 bytes
Capture duration:    15 seconds
Start time:          Thu Jan 11 16:10:34 2018
End time:            Thu Jan 11 16:10:49 2018
Data byte rate:      53034.10 bytes/sec
Data bit rate:       424272.82 bits/sec
Average packet size: 1715.30 bytes
Average packet rate: 30.92 packets/sec
SHA1:                df7f85751dd2d56d8a1b64a28b93b30553df6a08
RIPEMD160:           k37d0ccd2b20s81s6d9da4b44d3d168sc98s1aca
MD5:                 1q908e9a36cn382l768s95w4k6c8w6kb
Strict time order:   True
```

허용된 포트\(HTTP, HTTPS\)를 제외한 포트로의 외부 연결 파악하기

```text
$ ./tshark -o column.format:'" Source","%s","Destination","%d", "dstport", "%uD", "Protocol", "%p"' -r ~/tshark-log/temp.pcap -R "ip.src == 10.xxx.xxx.xxx && ! dns && tcp.dstport != 80 && tcp.dstport != 443" | sort -u

10.xxx.xxx.xxx -> 10.10.10.10 9973 TCP
10.xxx.xxx.xxx -> 10.10.10.10 18000 TCP
10.xxx.xxx.xxx -> 10.10.10.10 59048 HTTP
10.xxx.xxx.xxx -> 10.10.10.10 59048 TCP
```

-o 옵션을 통해서 티샤크의 옵션을 변경할 수 있다. `-o " <name>:<value> ... override preference setting`

