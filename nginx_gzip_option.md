# nginx gzip 옵션

응답을 압축하면 전송되는 데이터의 크기가 크게 줄어들기도 한다. 그러나 압축은 런타임에 발생하기 때문에 상당한 처리 오버헤드가 추가되어 성능에 부정적인 영향을 미칠 수 있다. NGINX는 클라이언트에 응답을 보내기 전에 압축을 수행하지만 이미 압축 된 응답 \(예 : 프록시 서버\)을 "이중 압축"하지 않는다.

압축을 enable하기 위해서 gzip 지시어를 추가한다. 기본적으로 NGINX는 응답을 text/html MIME type으로만 압축한다.

```text
gzip on;
```

다른 MINE type 응답을 압축하기 위해선 gzip\_types 지시어를 추가하고 추가적인 type 리스트를 적는다.

```text
gzip_types text/plain application/x-javascript text/xml text/css application/xml application/javascript;
```

압축 할 응답의 최소 길이를 지정하기 위해서는 gzip\_min\_length 지시어을 사용한다. default는 20 byte다. 작은 크기의 파일은 압축되지 않도록 10kb로 설정했다.

```text
gzip_min_length 10240;
```

NGINX는 기본적으로 프록시 서버에서 오는 요청에 대한 응답은 압축하지 않는다\(요청의 Via 헤더 필드가 있는지 여부에 따라 프록시 서버에서 오는 요청인지 결정된다\).

Nginx1 -&gt; Nginx2 -&gt; Upstream Server의 구조일 때, Via 헤더가 자동으로 추가되지 않는다. 따라서 Nginx1에서 아래와 같이 셋팅해주면 Nginx2에서는 gzip설정이 on 되어 있어도 패스한다. Via 헤더의 유무가 중요하다. 헤더 value는 어떤 값이라도 상관없다.

```text
proxy_set_header Via '10.xx.xx.xx';
```

주의 : 아래와 같이 add\_header를 사용하면 response 헤더에 추가되기 때문에 의도한대로 동작하지 않는다.

```text
add_header Via '10.xx.xx.xx';
```

이러한 응답의 압축을 설정하기 위해서는 gzip\_proxied 지시어를 사용한다. gzip\_proxied 지시어는 NGINX가 압축해야 하는 프록시된 요청의 종류를 지정하는 여러가지 파라미터가 있다. 예를 들어 프록시 서버에 캐시되지 않는 요청에만 응답을 압축하는 것이 합리적이다.

이를 위해 gzip\_proxied 지시어는 NGINX가 응답의 Cache-Control 헤더 필드를 확인하고 값이 no-cache, no-store 또는 private 인 경우 응답을 압축하도록 지시하는 파라미터를 포함한다. 또한 expired 파라미터를 포함시켜 만료 헤더 필드의 값을 확인해야한다. 이러한 파라미터는 Authorization 헤더 필드가 있는지 확인하는 auth 파라미터와 함께 다음 예제에서 설정된다\(인증 된 응답은 최종 사용자에게 고유하며 일반적으로 캐시되지 않는다\).

```text
gzip_proxied no-cache no-store private expired auth;
```

대부분의 다른 지시어와 마찬가지로 압축을 구성하는 지시어는 http context or server or location configuration block에 포함될 수 있다.

```text
gzip_http_version 1.1
```

압축이 필요한 HTTP Request의 최소 버전이다. 디폴트는 1.1이다.

Nginx1 -&gt; Nginx2 -&gt; Upstream Server의 구조일 때, Nginx `proxy_http_version` 디폴트는 1.0이기 때문에 `proxy_http_version` 을 1.1로 명시하거나, `gzip_http_version` 을 1.0으로 명시해야 한다.

gzip\_vary 옵션을 키면 response headers에 `Vary: Accept-Encoding`이 추가적으로 나온다.

```text
gzip_vary on;
```

Vary 헤더를 이해하기 위해서는 먼저 내용 협상\(Content-negotiation\)에 대한 이해가 필요하다.

종종 하나의 URL이 여러 리소스에 대응해야 할 경우가 있다. 콘텐츠를 여러 언어로 제공하려고 하는 웹 사이트의 예를 들어보자. 사용자에 맞게 서버가 알아서 영어나 프랑스어로 제공할 수 있도록 HTTP는 내용 협상\(Content-negotiation\) 방법을 제공한다. 여기서는 서로 다른 버전\(영어인지 프랑스어인지\)을 배리언트\(variant\)라고 부른다

내용 협상 기법은 3가지가 있다\(클라이언트 주도 협상, 서버 주도 협상, 투명 협상\). 클라이언트 주도 협상은 클라이언트가 요청을 보내면, 서버는 클라이언트에게 선택지를 보내주고, 클라이언트가 선택한다. 서버 입장에서는 구현하기 가장 쉽고 클라이언트도 최선의 선택을 할 수 있지만 최소 두 번의 요청으로 인해 대기시간이 길어질 수 밖에 없다.

서버 주도협상은 서버가 어떤 페이지를 돌려줄 것인지 결정하게 하는것이다. 그러나 이렇게 하려면 클라이언트는 반드시 자신의 무엇을 선호하는지에 대한 충분한 정보를 서버에게 주어서 서버가 현명한 결정을 할 수 있게 해주어야 한다. 서버는 이 정보를 클라이언트의 요청 헤더에서 얻는다.

HTTP 서버가 클라이언트에게 보내줄 적절한 응답을 계산하기 위해 사용하는 메커니즘은 다음 두 가지다.

* 내용 협상 헤더들을 살펴본다. 서버는 클라이언트의 Accept 관련 헤더들을 들여다보고 그에 알맞은 응답 헤더를 준비한다.
* 내용 협상 헤더 외의 다른 헤더들을 살펴본다. 예를 들어, 서버는 클라이언트의 User-Agent 헤더에 기반하여 응답을 보내줄 수도 있다.

**내용 협상 헤더**  
 Accept : 서버가 어떤 미디어 타입으로 보내도 되는지 알려준다.   
 Accept-Language : 서버가 어떤 언어로 보내도 되는지 알려준다.  
 Accept-Charset : 서버가 어떤 charset으로 보내도 되는지 알려준다.  
 Accept-Encoding : 서버가 어떤 인코딩으로 보내도 되는지 알려준다.  


이 헤더들이 엔터티 헤더들과 비슷함에 주목하라. 그러나 이 두 종류의 헤더는 서로 분명한 차이가 있다 엔터티 헤더는 선적 화물에 붙이는 라벨과 비슷하다. 그들은 메시지를 서버에서 클라이언트로 전송할 때 필요한 메시지 본문의 속성을 가리킨다.  
 **엔터티 헤더**  
 Content-Type  
 Content-Language  
 Content-Encoding  


한편 내용 협상 헤더들은 클라이언트와 서버가 선호 정보를 서로 교환하고 문서들의 여러 버전 중 하나를 선택하는 것을 도와, 클라이언트의 선호에 가장 잘 맞는 문서를 제공해 주기 위한 목적으로 사용된다.

**내용 협상 헤더의 품질값**  
 HTTP 프로토콜은 클라이언트가 각 선호의 카테고리마다 여러 선택 가능한 항목을 선호도와 함께 나열할 수 있도록 품질값을 정의하였다. 예를 들어, 클라이언트는 Accept-Language 헤더를 다음과 같은 형식으로 보낼 수 있다.

```text
Accept-Language: en;q=0.5, fr;q=0.0, nl;q=1.0, tr;q=0.0
```

q값은 0.0부터 1.0까지의 값을 가질 수 있다\(0.0이 가장 낮은 선호도, 1.0이 가장 높은 선호도를 의미한다\). 따라서 위의 헤더는 클라이언트가 네덜란드어\(nl\)로 된 문서를 받기를 원하고 있으나, 영어\(en\)로 된 문서라도 받아들일 것임을 의미하고 있다. 그러나 어떠한 경우에도 클라이언트는 프랑스어\(fr\)나 터키어\(tr\) 버전을 원하지는 않는다.

**그 외의 헤더들에 의해 결정**  
 서버는 또한 User-Agent와 같은 클라이언트의 다른 요청 헤더들을 이용해 알맞은 요청을 만들어내려고 시도할 수 있다. 예를 들어 서버가 오래된 버전의 웹브라우저는 자바스크립트를 지원하지 않는다는 것을 알고 있다면, 그들에게는 자바스크립트를 포함하지 않은 페이지를 돌려줄 수도 있다.

이 사례에서 '최선'에 가장 가까운 대응을 찾아낼 수 있는 q값 메커니즘은 없다. 서버는 정확한 대응을 찾아내거나 아니면 그냥 갖고 있는 것을 제공해주어야 한다.

캐시는 반드시 캐시된 문서의 올바른 '최선의' 버전을 제공해주려 해야 하기 때문에, HTTP 프로토콜은 서버가 응답에 넣어 보낼 수 있는 Vary 헤더를 정의한다. Vary 헤더는 캐시에게\(그리고 클라이언트나 그 외의 모든 다운스트림 프락시에게\) 서버가 내줄 응답의 최선의 버전을 결정하기 위해 어떤 요청 헤더를 참고하고 있는지 말해준다.

HTTP Vary 응답 헤더는 서버가 문서를 선택하거나 커스텀 콘텐츠를 생성할 때 고려한 클라이언트 요청 헤더 모두\(일반적인 내용 협상 헤더 외에 추가로 더해서\)를 나열한다. 예를 들어, 제공된 문서가 User-Agent 헤더에 의존한다면, Vary 헤더는 반드시 "User-Agent"를 포함해야 한다.

새 요청이 도착했을 때, 캐시는 내용 협상 헤더들을 이용해 가장 잘 맞는 것을 찾는다. 그러나 캐시가 문서를 클라이언트에게 제공해 줄 수 있게 되기 전에, 캐시는 반드시 캐시된 응답 안에 서버가 보낸 Vary 헤더가 들어있는지 확인해야 한다. 만약 Vary 헤더가 존재한다면, 그 Vary 헤더가 명시하고 있는 헤더들은 새 요청과 오래된 캐시된 요청에서 그 값이 서로 맞아야만 한다. 왜냐하면 서버는 클라이언트의 요청 헤더에 따라 그들의 응답이 달라질 수 있기 때문이다.

만약 서버의 Vary 헤더가 이렇다면, 거대한 수의 다른 User-Aget와 Cookie 값이 많은 배리언트\(variant\)를 만들어 낼 것이다

```text
Vary: User-Agent, Cookie
```

캐시는 각 배리언트마다 알맞은 문서 버전을 저장해야 한다. 캐시가 검색을 할 때, 먼저 내용 협상 헤더로 적합한 콘텐츠를 맞춰보고, 다음에 요청의 배리언트를 캐시된 배리언트와 맞춰본다. 만약 맞는 것이 없으면, 캐시는 문서를 서버에서 가져온다.

참고 : HTTP 완벽가이드 17장 내용 협상과 트랜스코딩

gzip 압축의 전체 구성은 다음과 같다.

```text
gzip             on;
gzip_vary        on;
gzip_comp_level  5;
gzip_min_length  10240;
gzip_proxied     expired no-cache no-store private auth;
gzip_types       text/plain application/x-javascript text/xml text/css application/xml application/javascript;
```

```text
gzip_comp_level 5;
```

5는 크기와 CPU 사용량간에 완벽한 절충안으로, 대부분의 ASCII 파일 \(레벨 9와 거의 동일\)에 대해 약 75 %의 감소를 제공한다\(기본값 : 1\).

[https://github.com/h5bp/server-configs-nginx/blob/master/nginx.conf](https://github.com/h5bp/server-configs-nginx/blob/master/nginx.conf)

추가적으로 ie 6이하는 gzip 옵션이 제공 되지 않으므로 disable 시킬 수도 있다. gzip\_disable 옵션에서 정규표현식으로 비활성화 시킬 수 있는데 "User-Agent"헤더에서 일치하는 것에 국한된다.

```text
gzip_disable "MSIE [1-6]\.";
```

참고 : [https://www.nginx.com/resources/admin-guide/compression-and-decompression/\#intro](https://www.nginx.com/resources/admin-guide/compression-and-decompression/#intro)

