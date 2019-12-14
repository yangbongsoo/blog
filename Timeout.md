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
### client_header_timeout

Defines a timeout for reading client request header.
If a client does not transmit the entire header within this time, the request is terminated with the 408 (Request Time-out) error.

client request header를 읽는 것의 timeout 시간 설정
client가 nginx client_header_timeout에 지정한 시간안에 전체 헤더를 전송하지 않으면 요청은 408(Request Time-out)로 끝난다. 디폴트 60초

### client_body_timeout 

Defines a timeout for reading client request body. 
The timeout is set only for a period between two successive read operations, 
not for the transmission of the whole request body. If a client does not transmit anything within this time, 
the request is terminated with the 408 (Request Time-out) error.

client request body를 읽는 것의 timeout 시간 설정
두개의 연속적인 읽기 작업 사이의 timeout 시간이다.
request body 전체 전송 timeout 시간이 아니다.
client가 nginx client_body_timeout에 지정한 시간안에 아무것도 전송하지 않으면 요청은 408(Request Time-out)로 끝난다. 디폴트 60초

```
// ngx_http_request_body.c Line : 245
static void
ngx_http_read_client_request_body_handler(ngx_http_request_t *r)
{
    ngx_int_t  rc;

    // 타임아웃이면 NGX_HTTP_REQUEST_TIME_OUT(408)로 ngx_http_finalize_request 호출
    if (r->connection->read->timedout) {
        r->connection->timedout = 1;
        ngx_http_finalize_request(r, NGX_HTTP_REQUEST_TIME_OUT);
        return;
    }

    rc = ngx_http_do_read_client_request_body(r);

    if (rc >= NGX_HTTP_SPECIAL_RESPONSE) {
        ngx_http_finalize_request(r, rc);
    }
}
```


```
// ngx_http_request_body.c Line : 264
static ngx_int_t
ngx_http_do_read_client_request_body(ngx_http_request_t *r)
{
    ...

        if (!c->read->ready) {

            if (r->request_body_no_buffering
                && rb->buf->pos != rb->buf->last)
            {
                /* pass buffer to request body filter chain */

                out.buf = rb->buf;
                out.next = NULL;

                rc = ngx_http_request_body_filter(r, &out);

                if (rc != NGX_OK) {
                    return rc;
                }
            }

            clcf = ngx_http_get_module_loc_conf(r, ngx_http_core_module);
            ngx_add_timer(c->read, clcf->client_body_timeout);

            if (ngx_handle_read_event(c->read, 0) != NGX_OK) {
                return NGX_HTTP_INTERNAL_SERVER_ERROR;
            }


            // ngx_http_finalize_request does not close the connection whenever rc ==
            // NGX_AGAIN. Instead ngx_http_read_client_request_body sets read handler
            // to ngx_http_read_client_request_body_handler. Therefore the read handler
            // will be recalled
            // https://www.ruby-forum.com/t/ngx-again-and-post-data/150217/2
            return NGX_AGAIN;
        }
    ...

```

```
// ngx_http_request.c Line 2413
void
ngx_http_finalize_request(ngx_http_request_t *r, ngx_int_t rc)
{
    ...

    if (rc == NGX_ERROR
        || rc == NGX_HTTP_REQUEST_TIME_OUT
        || rc == NGX_HTTP_CLIENT_CLOSED_REQUEST
        || c->error)
    {
        if (ngx_http_post_action(r) == NGX_OK) {
            return;
        }

        ngx_http_terminate_request(r, rc);
        return;
    }

    ...
}
```

 
디폴트 60초

### keepalive_timeout

```
keepalive_timeout timeout [header_timeout];
```

The first parameter sets a timeout during which a keep-alive client connection will stay open on the server side.
The zero value disables keep-alive client connections.
The optional second parameter sets a value in the “Keep-Alive: timeout=time” response header field.
Two parameters may differ.

첫번째 파라미터는 서버와 커넥션이 열려, 유지하는 시간의 timeout 설정이다(디폴트 75초).
0는 keep-alive client connections을 사용안하는 설정이다.
두번째 파라미터(옵션)는 응답 헤더 필드에 timeout 시간 설정이다. “Keep-Alive: timeout=time”
첫번째와 두번째 파라미터는 다르다.

The “Keep-Alive: timeout=time” header field is recognized by Mozilla and Konqueror.
MSIE closes keep-alive connections by itself in about 60 seconds.

“Keep-Alive: timeout=time” 헤더 필드는 Mozilla and Konqueror 에서 인식되고
MSIE는 약 60초 후 자체적으로 keep-alive connections을 닫는다.

### send_timeout

Sets a timeout for transmitting a response to the client.
The timeout is set only between two successive write operations, not for the transmission of the whole response.
If the client does not receive anything within this time, the connection is closed.

client로 응답을 전송하는데 timeout 시간인데
두개의 연속적인 쓰기 작업 사이의 timeout 시간이다.
전체 response 전송 timeout 시간이 아니다.
client가 nginx send_timeout에 지정한 시간안에 아무것도 받지 못하면 connection은 닫힌다. 디폴트 60초


### proxy_connect_timeout

Defines a timeout for establishing a connection with a proxied server.
It should be noted that this timeout cannot usually exceed 75 seconds.

proxied server 와 연결을 맺는데(establishing) timeout 시간 설정.
75초를 초과 할 수 없다. 디폴트 60초

### proxy_read_timeout

Defines a timeout for reading a response from the proxied server.
The timeout is set only between two successive read operations, not for the transmission of the whole response.
If the proxied server does not transmit anything within this time, the connection is closed.

proxied server 로부터 response를 읽는데 timeout 시간 설정
두개의 연속적인 읽기 작업 사이의 timeout 시간이다.
전체 response 전송 timeout 시간이 아니다.
proxied server가 nginx proxy_read_timeout에 지정한 시간안에 아무것도 전송하지 않으면 connection은 닫힌다.

디폴트 60초

### proxy_send_timeout 

Sets a timeout for transmitting a request to the proxied server.
The timeout is set only between two successive write operations,
not for the transmission of the whole request.
If the proxied server does not receive anything within this time, the connection is closed.

proxied server로 요청을 전송하는데 timeout 시간 설정
두개의 연속적인 쓰기 작업 사이의 timeout 시간이다.
전체 request 전송 timeout 시간이 아니다.
proxied server가 nginx proxy_send_timeout에 지정한 시간안에 아무것도 받지 못하면 connection은 닫힌다.

