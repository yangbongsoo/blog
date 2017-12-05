# 1. Hello World 찍기

NGINX 모듈 제작하기 : http://d2.naver.com/helloworld/192785
github 주소 : https://github.com/yangbongsoo/ngx\_http\_hello\_world

cf) nginx 1.9.11버전부터 apache처럼 동적 링크가 가능하다. 
https://www.nginx.com/resources/wiki/extending/converting/#compiling-dynamic
그건 나중에 살펴보자. 

먼저 nginx 바이너리에 모듈을 추가해 빌드하려면 두개의 파일이 필요하다.

##config
```
ngx_addon_name=ngx_http_hello_world_module

if test -n "$ngx_module_link"; then
    ngx_module_type=HTTP
    ngx_module_name=$ngx_addon_name
    ngx_module_incs=
    ngx_module_deps=
    ngx_module_srcs="$ngx_addon_dir/ngx_http_hello_world_module.c"
    ngx_module_libs=
   . auto/module
else
    HTTP_MODULES="$HTTP_MODULES ngx_http_hello_world_module"
    NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_hello_world_module.c"
fi
```

##`ngx_http_(your module)_module.c`

```c
/*
 * Copyright (C) 2013 Tatsuhiko Kubo <cubicdaiya@gmail.com>
 */

#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>

#define NGX_HTTP_HELLO_WORLD "Hello, World!\n"

static char *ngx_http_hello_world(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
static ngx_int_t ngx_http_hello_world_handler(ngx_http_request_t *r);

static ngx_command_t ngx_http_hello_world_commands[] = {
    { 
        ngx_string("hello_world"),
        NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS,
        ngx_http_hello_world,
        0,
        0,
        NULL
    },
    ngx_null_command
};


static ngx_http_module_t ngx_http_hello_world_module_ctx = {
    NULL,                              /* preconfiguration */
    NULL,                              /* postconfiguration */
    
    NULL,                              /* create main configuration */
    NULL,                              /* init main configuration */

    NULL,                              /* create server configuration */
    NULL,                              /* merge server configuration */

    NULL,                              /* create location configuration */
    NULL                               /* merge location configuration */
};

ngx_module_t ngx_http_hello_world_module = {
    NGX_MODULE_V1,
    &ngx_http_hello_world_module_ctx, /* module context */
    ngx_http_hello_world_commands,    /* module directives */
    NGX_HTTP_MODULE,                  /* module type */
    NULL,                             /* init master */
    NULL,                             /* init module */
    NULL,                             /* init process */
    NULL,                             /* init thread */
    NULL,                             /* exit thread */
    NULL,                             /* exit process */
    NULL,                             /* exit master */
    NGX_MODULE_V1_PADDING
};

static ngx_int_t ngx_http_hello_world_handler(ngx_http_request_t *r)
{
    ngx_int_t                    rc;
    ngx_chain_t                  out;
    ngx_buf_t                   *b;
    ngx_str_t                    body = ngx_string(NGX_HTTP_HELLO_WORLD);

    if (r->method != NGX_HTTP_GET && r->method != NGX_HTTP_HEAD) {
        return NGX_HTTP_NOT_ALLOWED;
    }

    if (r->headers_in.if_modified_since) {
        return NGX_HTTP_NOT_MODIFIED;
    }

    b = ngx_pcalloc(r->pool, sizeof(ngx_buf_t));
    if (b == NULL) {
        return NGX_HTTP_INTERNAL_SERVER_ERROR;
    }
    b->pos      = body.data;
    b->last     = b->pos + body.len;
    b->memory   = 1;
    b->last_buf = 1;
    out.buf     = b;
    out.next    = NULL;

    ngx_str_set(&r->headers_out.content_type, "text/plain");
    r->headers_out.status            = NGX_HTTP_OK;
    r->headers_out.content_length_n  = body.len;

    rc = ngx_http_send_header(r);
    if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
        return rc;
    }

    return ngx_http_output_filter(r, &out);
}
 
static char *ngx_http_hello_world(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    ngx_http_core_loc_conf_t  *clcf;

    clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
    clcf->handler = ngx_http_hello_world_handler;

    return NGX_CONF_OK;
}
```
한번에 모든 코드르 보면 복잡하니 하나씩 확인해보자.

### 모듈지시어(Module Directives)
```c
static ngx_command_t ngx_http_hello_world_commands[] = {
    { 
        ngx_string("hello_world"),
        NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS,
        ngx_http_hello_world,
        0,
        0,
        NULL
    },
    ngx_null_command
};
```
모듈 지시어는 ngx_command_t 구조체로 선언된다. Nginx에 내장된 ngx_command_t 구조체를 이용하면 설정 파일의 값과 설정 구조체 변수를 연결할 수 있다. ngx_command_t 구조체의 타입 선언은 다음과 같다.
```c
typedef struct ngx_command_s {  
    ngx_str_t name;
    ngx_uint_t type;
    char *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
    ngx_uint_t conf;
    ngx_uint_t offset;
    void *post;
} ngx_command_t;
```
name 필드는 nginx.conf에 입력된 키워드 이름을 지정한다. ex) `ngx_string("hello_world")`

type 필드는 키워드가 설정된 위치와 키워드 뒤에 입력된 인수의 개수를 지정한다. ex) `NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS`

cf)
```
NGX_HTTP_MAIN_CONF: 메인 설정(main config)에서 유효함
NGX_HTTP_SRV_CONF: 서버 설정(server (host) config) 유효함
NGX_HTTP_LOC_CONF: 로케이션 설정(location config)에서 유효함
NGX_HTTP_UPS_CONF: 업스트림 설정(upstream config)에서 유효함
NGX_CONF_NOARGS: 지시어(directive)가 0개의 인자를 가짐
NGX_CONF_TAKE1: 지시어(directive)가 1개의 인자를 가짐
NGX_CONF_TAKE2: 지시어(directive)가 2개의 인자를 가짐
……
NGX_CONF_TAKE7: 지시어(directive)가 7개의 인자를 가짐
NGX_CONF_FLAG: 지시어(directive) 가 부울 인자("on" or "off")를 가짐
NGX_CONF_1MORE: 지시어(directive)가 1개 이상의 인자를 가짐
NGX_CONF_2MORE: 지시어(directive)가 2개 이상의 인자를 가짐
```

set 필드는 지정한 키워드의 값을 가져올 때 사용할 함수를 설정한다.
```c
static char *ngx_http_hello_world(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    ngx_http_core_loc_conf_t  *clcf;

    clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
    clcf->handler = ngx_http_hello_world_handler;

    return NGX_CONF_OK;
}
```
코어(core) 모듈의 핸들러 콜백 함수 포인터에 핸들러 함수를 등록한다.

```c
static ngx_int_t ngx_http_hello_world_handler(ngx_http_request_t *r)
{
    ngx_int_t                    rc;
    ngx_chain_t                  out;
    ngx_buf_t                   *b;
    ngx_str_t                    body = ngx_string(NGX_HTTP_HELLO_WORLD);

    if (r->method != NGX_HTTP_GET && r->method != NGX_HTTP_HEAD) {
        return NGX_HTTP_NOT_ALLOWED;
    }

    if (r->headers_in.if_modified_since) {
        return NGX_HTTP_NOT_MODIFIED;
    }

    b = ngx_pcalloc(r->pool, sizeof(ngx_buf_t));
    if (b == NULL) {
        return NGX_HTTP_INTERNAL_SERVER_ERROR;
    }
    b->pos      = body.data;
    b->last     = b->pos + body.len;
    b->memory   = 1;
    b->last_buf = 1;
    out.buf     = b;
    out.next    = NULL;

    ngx_str_set(&r->headers_out.content_type, "text/plain");
    r->headers_out.status            = NGX_HTTP_OK;
    r->headers_out.content_length_n  = body.len;

    rc = ngx_http_send_header(r);
    if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
        return rc;
    }

    return ngx_http_output_filter(r, &out);
}
```
핸들러 모듈 구현은 다음의 네 가지 단계로 나눌 수 있다.
1. 로케이션(location)설정 가져오기
2. HTTP 요청에 대한 HTTP 응답 생성
3. HTTP 응답 헤더 전송
4. HTTP 응답 본체 전송

cf) 위의 코드에서는 로케이션 설정 가져오기는 없다. 예를 들면 아래와 같다.
```c
static ngx_int_t ngx_http_mymodule_handler(ngx_http_request_t *r)  
{
    ngx_http_mymodule_loc_conf_t *my_config = NULL;
    myconfig = ngx_http_get_module_loc_conf(r, ngx_http_mymodule_module);
    ...
}
```
그다음, 핸들러는 파라미터로 ngx_http_request_t 구조체 타입의 HTTP 요청을 받는다.
```c
typedef struct ngx_http_request_s {  
    ...
    ngx_str_t uri;
    ngx_str_t args;
    ngx_http_headers_in_t headers_in;
    ...
} ngx_http_request_t;
```
uri: 요청한 페이지의 경로(예: /query.cgi)
args: 요청할 때 "?" 뒤에 오는 파라미터
headers_in: 요청 기타 인수(accept-range, cookie 등)가 저장된 구조체

다음은 응답 헤더 전송이다.
```c
ngx_str_set(&r->headers_out.content_type, "text/plain");
r->headers_out.status = NGX_HTTP_OK;
r->headers_out.content_length_n = body.len;

rc = ngx_http_send_header(r);
if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
return rc;
}
```

다음은 응답 본체 전송이다. HTTP 응답 본체를 전송하려면 Nginx에서 정의한 ngx_buf_t와 ngx_chain_t 구조체를 사용한다. HTTP 응답 본체를 ngx_buf_t 구조체 타입의 버퍼에 저장하고, 버퍼를 다시 ngx_chain_t 구에 연결하고 체인 구조를 만들어 필터로 전송한다.
```c
... 

ngx_chain_t out;
ngx_buf_t *b;

...
/* 버퍼 할당과 HTTP 응답을 버퍼에 저장 */
b = ngx_pcalloc(r->pool, sizeof(ngx_buf_t));

b->pos      = body.data;
b->last     = b->pos + body.len;
b->memory   = 1;
b->last_buf = 1;
/* 버퍼를 체인 구조에 연결*/
out.buf     = b;
out.next    = NULL;

return ngx_http_output_filter(r, &out);
```
중요한 점은 C 표준 라이브러리의 malloc 함수 대신 ngx_palloc 함수를 사용하여 설정 구조체 메모리를 할당한 것이다. ngx_palloc 함수는 NGINX 모듈을 개발할 때 항상 사용해야 하는 함수로서 메모리 풀(pool) 포인터와 메모리 크기를 파라미터로 받는다. ngx_palloc 함수를 사용하면 C 표준 라이브러리의 free 함수를 명시적으로 호출하여 메모리를 해제할 필요가 없다. 어떤 메모리 풀에서 메모리를 할당받는지에 따라 ngx_palloc 함수로 할당받은 메모리는 메모리 풀을 소유한 객체의 수명(life time)이 다할 때까지 유효하다. 위 예에서 할당한 메모리는 ngx_conf_t 객체의 수명 동안 유효하다.