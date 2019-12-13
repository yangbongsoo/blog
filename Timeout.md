# Timeout 



## RestTemplate(HttpComponentsClientHttpRequestFactory)

HttpComponentsClientHttpRequestFactory

1. org.apache.httpcomponents.httpclient (4.5.9)
1-1. maxConnPerRoute : IP/Domain name 당 최대 커넥션 갯수
1-2. maxConnTotal : 최대 커넥션 갯수
1-3. automaticRetriesDisabled : retry 안함 설정 따로 안하면 디폴트 DefaultHttpRequestRetryHandler(디폴트 3번)
1-4. setKeepAliveStrategy
1-5. setConnectionManagerShared
1-6. evictIdleConnections

2. RequestConfig
2-1. connectionRequestTimeout : ConnectionManager(커넥션풀)로부터 꺼내올 때의 타임아웃
2-2. connectTimeout : connection 맺기(서버에 소켓 연결)까지의 시간 타임아웃
2-3. readTimeout : socketTimeout. 요청/응답간의 타임아웃.

3. bufferRequestBody : 내부적으로 request body 버퍼 만듦. default is true. POST나 PUT으로 많은 양의 데이터를 보낼 때 false로 하길 추천.
---

okhttp3.OkHttpClient
1. connectTimeout 
2. readTimeout
3. writeTimeout

---

java.net.HttpURLConnection 
1. connectTimeout : 
2. readTimeout : 

---

WebClient
reactor.netty.http.client

1. Mono.timeout :
2. ReadTimeoutHandler :
3. WriteTimeoutHandler : 


## Nginx Timeout
### client_body_timeout 

Defines a timeout for reading client request body. 
The timeout is set only for a period between two successive read operations, 
not for the transmission of the whole request body. If a client does not transmit anything within this time, 
the request is terminated with the 408 (Request Time-out) error.

client request body를 읽는 것의 timeout 시간 설정
 
디폴트 60초
### client_header_timeout

Defines a timeout for reading client request header. 
If a client does not transmit the entire header within this time, the request is terminated with the 408 (Request Time-out) error.

client request header를 읽는 것의 timeout 시간 설정
client가 지정된 시간안에 전체 헤더를 전송하지 않으면 요청은 408(Request Time-out)로 끝난다. 디폴트 60초
 
### keepalive_timeout 
### send_timeout 
### proxy_connect_timeout 
### proxy_read_timeout 
### proxy_send_timeout 
