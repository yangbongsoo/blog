# 웹을 지탱하는 기술

HTTP는 원래 하이퍼텍스트를 전송하기 위한 프로토콜이었지만, 실제로는 하이퍼텍스트 이외의 다양한 것들을 전송하고 있다. 그것이 무엇인가 하면, 리소스 상태(Resource State)의 표현(Representation)이라는 것이 필딩의 주장이다(REST 창시자).

**리소스**<br>
REST에 있어서 중요한 개념의 하나로 리소스가 있다. 우선 웹에서 리소스의 예를 보자.
```
서울의 일기예보
청량리역의 사진
다음 웹 페이지
Dijkstra의 논문 'Go To Statement Considered Harmful'
```
웹 상에서는 이 밖에도 다양한 리소스가 존재한다. 리소스를 한마디로 설명하면, '웹상에 존재하는 이름을 가진 모든 정보'가 된다.

**리소스 명칭으로서의 URI**<br>
```
서울의 일기 예보
http://weather.daum.net/rgn/cityWetrWarea  

청량리역의 사진
http://wwwflickr.com/photos/nala2sky/2121321/ 

Dijkstra의 논문 'Go To Statement Considered Harmful'
http://www.ecn.purdue.edu/ParaMount/papers/diijkstra68goto.pdf

cf) 복합한 URI의 예
http://yohei:pass@blog.example.com:8000/search?q=test&debug=true#n10
URI 스키마(hhtp) 다음에 사용자 정보가 들어가 있다. 사용자 정보는 이 리소스에 접근할 때 이용할 사용자 이름과 패스워드로 구성된다. 
```
즉, 리소스란 웹상의 정보이다. 전 세계 무수한 리소스는 각각 URI로 의미 있는 이름을 가진다. URI를 이용함으로써, 프로그램은 리소스가 표현하는 정보에 접근할 수 있다.

**클라이언트/서버**<br>
웹은 HTTP라는 프로토콜을 이용해 클라이언트와 서버가 서로 통신하는 클라이언트/서버의 아키텍처 스타일을 채용하고 있다. 클라이언트/서버의 이점은 단일 컴퓨터 상에서 모든 것을 처리하는 것이 아니라, 클라이언트와 서버로 분리해서 처리할 수 있다는 점이다.

**URI, URL, URN**<br>
URI와 비슷한 이름으로 URL과 URN이 있다. 정확히 말하지면 URI는 URL과 URN을 총칭하는 이름이다. URL에는 도메인을 갱신하지 않았거나 서버가 어떤 장애로 인해 변경되면 액세스 할 수 없다는 문제가 있다. 이 문제에 대응하기 위해 도메인명과는 독립적으로 리소스에 항구적인 ID를 할당하기 위한 스펙이 검토되었고 그 결과가 URN이다. URN을 이용하면 리소스에 도메인명과는 독립된 이름을 붙일 수 있다. 예를 들어 서적은 ISBN이라는 세계적으로 통일된 ID를 갖고 있다. `urn:isbn:924483858493` 이렇게 URN은 도메인명에 의존하지 않는다. 이런 특성을 가진 URL과 URN을 합해 URI라고 부르게 되었다. 즉, URI는 2개의 ID체계를 합한 총칭이다.

## HTTP의 기본
HTTP란 이름대로라면 하이퍼텍스트 전송용 프로토콜이지만, 실제로는 HTML과 XML 같은 하이퍼텍스트뿐만 아니라 이미지, 음성, 동영상, JavaScript 프로그램, PDF와 각종 오피스 도큐먼트 파일 등 컴퓨터에서 다룰 수 있는 데이터라면 무엇이든 전송할 수 있다.

**미디어 타입**<br>
HTTP는 많은 데이터 타입을 다루기 때문에 웹에서 전송되는 객체 각각에 신중하게 MIME 타입이라는 데이터 포맷 라벨을 붙인다(Multipurpose Internet Mail Extensions). 원래 각기 다른 전자메일 시스템 사이에서 메세지가 오갈 때 겪는 문제점을 해결하기 위해 설계되는데 HTTP에서도 멀티미디어 콘텐츠를 기술하고 라벨을 붙이기 위해 채택되었다.
```
Content-Type: ( MIME 타입 )

HTML 텍스트 문서 : text/hml
plain ASCII 텍스트 문서 : text/plain
JPEG 이미지 : image/jpeg
GIF 이미지 : image/gif
MS ppt : application/vnd.ms-powerpoint
JSON : application/json
```

**TCP/IP**<br>

![](/assets/osi7layer.jpg)

HTTP는 자신의 메세지 데이터를 전송하기 위해 TCP를 사용한다. HTTP 클라이언트가 서버에 메세지를 전송할 수 있게 되기 전에, IP주소와 포트번호를 사용해 클라이언트와 서버 사이에 TCP/IP 커넥션을 맺어야 한다.

1. 브라우저는 서버의 URL에서 호스트 명을 추출한다.
2. 브라우저는 서버의 호스트 명을 IP로 변환한다.
3. 브라우저는 URL에서 포트번호를 추출한다.
4. 브라우저는 추출한 IP와 포트번호로 TCP 커넥션을 맺는다.
5. 브라우저는 서버에 HTTP 요청을 보낸다.
6. 서버는 브라우저에 HTTP 응답을 돌려준다.
7. 커넥션이 닫히면, 브라우저는 문서를 보여준다.

**HTTP 메세지**<br>
```
요청 메세지
GET /search?q=test&debug=true#n10 HTTP/1.1
Host : example.com:8080 
--------
바디
--------
```
요청 메세지의 둘째 줄 부터 헤더가 이어진다. 헤더는 메세지의 메타 데이터이다. 그리고 하나의 메세지는 복수의 헤더를 가질 수 있다. 마지막으로 바디에는 그 메세지를 나타내는 본질적인 정보가 들어간다.

```
응답 메세지
HTTP/1.1 200 OK
Content-Type : application/xhtml+xml; charset=utf-8

<html>
...
</html>
```
응답 메세지의 첫째 줄은 status line이라고 하며 프로토콜 버전(HTTP.1,1), status code(200), 테스트 구문(OK)으로 구성된다.
## HTTP 메서드
```
GET : 리소스 취득
POST : 서브 리소스의 작성, 리소스 데이터의 추가, 그밖의 처리
PUT : 리소스 갱신, 리소스 작성
DELETE : 리소스 삭제
HEAD : 리소스의 헤더(메타 데이터) 취득
OPTIONS : 리소스가 서포트하는 메서드의 취득
TRACE : 자기 앞으로 요청 메세지를 반환(루프 백) 시험
CONNECT : 프록시 동작의 터널 접속으로 변경

대표적인 CRUD 메서드
GET : 리소스 취득
POST : 서브 리소스의 작성, 리소스 데이터의 추가, 그밖의 처리
PUT : 리소스 갱신, 리소스 작성
DELETE : 리소스 삭제
```
POST, PUT 두개 다 리소스를 작성할 수 있다. 그럼 이 두 가지를 어떻게 구분해서 사용하면 좋을까? 정답은 없지만 설계상의 지침으로 다음과 같은 사실이 있다. POST로 리소스를 작성할 경우, 클라이언트는 리소스의 URI를 지정할 수 없다. URI의 결정권은 서버 측에 있다. 반대로 PUT으로 리소스를 작성할 경우, 리소스의 URI는 클라이언트가 결정한다.

예를 들어, Twitter와 같이 포스팅한 트윗의 URI를 서버 측이 자동적으로 결정하는 웹 서브시의 경우는 POST를 이용하는 것이 일반적이다. 반대로, Wiki 처럼 클라이언트가 결정한 타이틀이 그대로 URI가 되는 웹 서비스는 PUT을 사용하는 편이 적합하다.

특별한 이유가 없는 한, 리소스의 작성은 POST로 수행하여 URI를 서버 측에서 결정하는 설계가 바람직하다. 

OPTIONS는 그 리소스가 지원하고 있는 메서드 목록을 반환한다. 리소스마다 대응하는 메서드를 반환하도록 직접 구현해야 되며 Apache와 같은 WebDAV에 대응하는 웹 서버에서는 설정파일로 OPTIONS의 동작을 설정할 수 있다.
```
요청
OPTIONS /list HTTP/1.1
Host: example.com

응답
HTTP/1.1 200 OK
Allow: GET, HEAD, POST
```

**조건부 요청**<br>
HTTP 메서드와 갱신일자 등으로 헤더를 구성하면 메서드의 실행 여부를 리소스의 갱신일자를 조건으로 서버가 선택할 수 있다. 예를 들어, GET에 리소스의 갱신일자를 조건으로 넣기 위해서는 `If-Modified-Since`헤더를 사용한다. 이 헤더가 들어간 GET은 리소스가 이 시간 이후 갱신되어 있으면 GET한다는 의미가 된다. 마찬가지로 PUT과 조합하면 이 시간 이후로 갱신되어 있지 않으면 리소스를 갱신한다는 의미가 된다.

## HTTP 헤더
리소스에 대한 접근권한을 설정하는 인증이나 클라이언트와 서버의 통신횟수와 양을 감소시키는 캐시 같은 HTTP의 기능을 헤더로 실현한다.

**날짜와 시간**<br>
```
Date 헤더
Date: Tue, 06 Jul 2010 03:21:05 GMT
```

**MIME 미디어 타입**<br>
```
Content-Type: text/plain; charset=utf-8
```
미디어 타입은 charset 파라미터를 가질 수 있다. charset 파라미터는 생략 가능하지만 타입이 text인 경우에는 주의가 필요하다. HTTP에서 text타입 디폴트 문자 인코딩은 ISO 8859-1로 정의하고 있다. 그렇기 때문에 다음과 같은 메세지는 한글 텍스트가 들어가 있음에도 불구하고, 클라이언트가 ISO 8859-1로 해석해 문자가 깨질 가능성이 있다.

```
한글 텍스트임에도 불구하고 ISO 8859-1이 적용되는 예

HTTP/1.1 200 OK
Content-Type: text/plain

한글 텍스트
```
**더욱 까다로운 것은 XML 처럼 문서 자체에서 문자 인코딩 방식을 선언할 수 있는 경우라도, text 타입의 경우에는 Content-Type 헤더의 charset 파라미터를 우선한다는 것이다.**<br>
```
XML이 선언한 문자 인코딩 지정을 무시하는 예

HTTP/1.1 200 OK
Content-Type: text/html

<?xml version=“1.0” encoding=“utf-8”?>
<test>한글 텍스트</test>
```
이 문제는 text타입인 경우에 반드시 charset 파라미터를 붙이도록 하면 해결된다. 또한 XML 문서의 경우에는 text/html을 사용하지 않고, application/xml이나 application/xhtml+xml과 같은 파라미터를 이용하고, 반드시 charset 파라미터를 붙이는 것이 현시점에서는 가장 바람직한 운용방법이다.
```
바르게 문자 인코딩을 지정한 예

HTTP/1.1 200 OK
Content-Type: application/xml; charset=utf-8

<?xml version=“1.0” encoding=“utf-8”?>
<test>한글 텍스트</test>
```

**주요 서브타입**<br>
```
text/plain
text/csv
text/css
text/html
text/xml XML 문서(비추천)
image/jpeg
image/gif
image/png
application/xml
application/xhtml+xml
application/atom+xml Atom 문서
application/atomsvc+xml Atom의 서비스 문서
application/atomcat+xml Atom의 카테고리 문서
application/javascript
application/json
application/msword
application/vnd.ms-excel
application/vns.ms-powerpoint
application/pdf
application/zip
application/x-shockwave-flash
application/x-www-form-urlencoded HTML 폼 형식
```

**Content Negotiation**<br>
지금까지 설명했던 미디어 타입과 문자 인코딩, 언어 태그는 서버가 일방적으로 결정하는 것 뿐만 아니라, 클라이언트와 교섭해서 정할 수도 있다.
Accept - 처리할 수 있는 미디어 타입을 전달한다.
```
Accept: text/html, application/xhtml+xml, application/xml; q=0.9,*/*;q=0.8
```
q=이라는 파라미터의 값을 qvalue라고 하며, 그 미디어 타입의 우선순위를 나타낸다. qvalue는 소수점 이하 세 자리 이내의 0~1까지의 수치이며 수치가 더 큰 쪽을 우선한다. 이 예의 경우, text/html, application/xhtml+xml이 디폴트인 1, application/xml이 0.9, 그 밖의 모든 미디어 타입(`*/*`)이 0.8이라는 우선도를 가진다.

클라이언트가 Accept 헤더에 지정한 미디어 타입에 서버가 대응하고 있지 않다면 406 Not Acceptable이 반환된다.

**Content-Length와 청크(chunk) 전송**<br>
메세지의 바디가 있는 경우 기본적으로 Content-Length 헤더를 이용해 그 사이즈를 10진수의 바이트로 나타낸다. 미리 사이즈를 알고 있는 리소스인 정적인 파일 등을 전송할 때는 Content-Length 헤더를 이용하는 것이 간단하다.

하지만 동적으로 이미지를 생성하는 웹 서비스의 경우, 파일 사이즈가 정해질 때까지 응답할 수 없기 때문에 응답성능이 저하된다. 이때 사용하는 것이 Transfer-Encoding 헤더다.
```
Transfer-Encoding: chunked
```
Transfer-Encoding 헤더에 Chunked를 지정하면 최종적으로는 사이즈를 모르는 바디를 조금씩 전송할 수 있다. 다음 예에서는 ‘The brown fox jumps quickly over the lazy dog’이라는 46바이트의 문자열을 16바이트 청크 2개와 14바이트 청크 1개로 분할하여 POST한다.
```
POST /test HTTP/1.1
Host: example.com

Transfer-Encoding: chunked
Content-Type: Text/plain; charset=utf-8

10
The brow fox ju

10
maps quickly over

e
the lazy dog.

0
(여기에도 빈 줄)
```
마지막에는 반드시 길이가 0인 청크와 빈 줄을 붙이도록 스펙에서 규정하고 있다.

**인증**<br>
현재 주류인 HTTP 인증 방식에는 HTTP 1.1이 규정하고 있는 Basic 인증과 Digest 인증이 있다. 또한 웹 API에서는 WSSE(WS-Security Extension)라는 HTTP 인증의 확장 스펙을 이용하는 경우도 있다.

 어떤 리소스에 액세스 제어가 걸려 있는 경우, **status code 401 Unauthorized**(이 리소스에 접근하려면 적절한 인증이 필요)와 **WWW-Authenticate 헤더**를 이용해 클라이언트에 리소스 접근에 필요한 인증정보를 통지할 수 있다.
```
요청 
DELETE /test HTTP/1.1
Host: example.com

응답
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm=“Example.com"
```
WWW-Authenticate 헤더에 의해 클라이언트는 서버가 제공하는 인증 방식을 이해할 수 있게 되고 그 방식을 따른 형식으로 인증 정보를 보낼 수 있다. 위의 예에서는 이 서버가 Basic 인증을 지원하고 있다는 것을 알 수 있다. ‘realm’은 서버 상에서 이 리소스가 속한 URI 공간의 이름이 된다.

cf) Basic, Digest 인증 모두 패스워드만 암호화할 뿐이기 때문에 메세지 자체는 평문으로 네트워크를 흘러간다. 따라서 메세지도 암호화하고 싶을 때는 HTTPS를 이용해야 한다.

**캐시**<br>
클라이언트는 서버에서 가져온 리소스의 캐시 가능 여부를 조사하고 가능한 경우는 로컬 스토리지에 저장한다. 어떤 리소스가 캐시 가능한지는 그 리소스를 취득했을 때의 헤더로 판단한다. 리소스가 캐시 가능한지, 그 유효기간이 언제까지인지는 Pragma, Expire, Cche-Control 헤더를 이용해 서버가 지정한다.
```
Pragma - 캐시를 억제한다

HTTP/1.1 200 OK
Content-Type: application/xhtml+xml, charset=utf-8
Pragma: no-cache
```
```
Expires - 캐시의 유효기간을 나타낸다

HTTP/1.1 200 OK
Content-Type: application/xhtml+xml, charset=utf-8
Expires: Thu, 11 May 2010 16:00:00 GMT
```
이 응답에는 2010년 5월 11일 16시까지는 캐시가 유효하다는 것을 서버가 보증하고 있다. 리소스가 변경할 가능성이 없는 경우는 캐시의 유효기간을 무한으로 설정하고 싶겠지만 그런 경우라도 최장 약 1년 이내로 일시를 넣을 것을 스펙에서는 권장한다.

Pragma 헤더와 Expires 헤더는 HTTP 1.0이 정의한 헤더다. 간단한 캐시는 이들로 구현할 수 있지만 복잡한 지정은 할 수 없다. 그래서 HTTP 1.1에서는 Cache-Control 헤더를 추가했다. 따라서 Pragma 헤더와 Expires 헤더의 기능은 Cache-Control 헤더로 완전히 대응할 수 있다.
```
Pragma: no-cache 와 Cache-Control: no-cache 는 같다.
```
캐시를 시키지 않을 경우는 Pragma와 Cache-Control의 no-cache를 동시에 지정한다.

또한 Expires에서는 절대시간으로 유효기간을 표시했는데 Cache-Control에서는 현재로부터의 상대시간으로 유효기간을 설정할 수 있다. 아래의 예는 86400초, 즉 현재로부터 24시간 캐시가 유효하다는 것을 의미한다.
```
Cache-Control: max-age: 86400
```

**조건부 GET**<br>
```
GET /test HTTP/1.1
Host: example.com
If-Modified-Since: Thu, 11 May 2010 16:00:00 GMT
```
서버의 리소스가 Thu, 11 May 2010 16:00:00 GMT 이후로 변경되지 않았다면 다음과 같은 응답을 보낸다.
```
HTTP/1.1 304 Not Modified
Content-Type: application/xhtml+xml, charset=utf-8
Last-Modified: Thu, 11 May 2010 16:00:00 GMT
```
이 응답에는 바디가 포함되지 않기 때문에 그만큼 네트워크 대역을 절약할 수 있다.

If-Modified-Since 헤더와 Last-Modified 헤더에 의한 조건부 GET은 편리하지만, 시계를 가지고 있지 않은 서버와 밀리 초 단위로 변경될 가능성이 있는 리소스에는 이용할 수 없다. 그 경우 이용하는 것이 If-None-Match 헤더와 ETag(엔티티 태그)헤더다.
```
GET /test2 HTTP/1.1
Host: example.com
If-None-Match: ab3322028
```
If-None-Match 헤더는 ‘지정한 값과 매치하지 않으면’이라는 조건이 된다. If-None-Match 헤더에 지정하는 값은 캐시하고 있는 리소스의 ETag 헤더의 값이다. 서버상의 리소스가 변경되어 있지 않으면 다음의 응답을 반환한다.
```
HTTP/1.1 304 Not Modified
Content-Type: application/xhtml+xml, charset=utf-8
ETag: ab3322028
```
ETag는 리소스의 갱신 상태를 비교하기 위해서만 사용하는 문자열이다. 리소스를 갱신했을 때 다른 값이 되는 것이면 어떤 문자라도 상관없다.
## JSON
JavaSript Object Notation의 약자로 **데이터 표현 형식 중 하나**이다. 

JSON의 미디어 타입은 `application/json`이다. JSON은 스펙 상 UTF-8, UTF-16, UTF-32 중 하나로 인코딩하도록 되어 있다. 따라서 HTTP 헤더 등의 미디어 타입에서 파라미터로 지정할 수 있다. 
```
Content-Type: application/json; charset=utf-8
```
cf) front에서 AJAX 통신할 때
```
$.ajax({
          type: 'get',
          url: '/message/random',
          contentType: 'application/json',
          dataType: 'json',
          success: function (data) {
          },
          error: function(data){
          }
      });
```
contentType 을 통해서 json형식이라는 것을 알리고 response로 받아오는 데이터도 json이라는 것을 dataType을 통해 명시한다. 

JSON 자료형은 object, array, string, number, boolean, null 이렇게 총 6가지가 있다. JSON의 문자열은 반드시 이중인용부호(")로 감싸준다. 

**JSON에 의한 크로스 도메인 통신**<br>
JSON으로 리소스 표현을 제공하는 부차적 효과로서, JSONP(JSON with PAdding)을 이용할 수 있다. JSONP가 필요하게 된 배경부터 설명하면, Ajax에서 이용하는 XMLHttpRequest라는 JavaScript의 모듈은 보안상의 제한으로 인해 JavaScript 파일을 가져왔던 동일한 서버하고만 통신할 수 있다. JavaScript가 있는 서버와 다른 서버가 통신할 수 있다면 브라우저에서 입력한 정보를 사용자가 모르는 사이에 부정한 서버로 전송할 수 있게 되기 때문이다.

이와 관련해, 이렇게 불특정 다수의 도메인에 속하는 서버에 액세스하는 것을 '크로스 도메인 통신'이라고 부른다. 복수 도메인의 서버와 통신할 수 없고 단일 도메인과만 통신해야 한다는 것은 커다란 제약이다. 예를 들어, 우리 서비스에서 데이터를 직접 들고 있지 않고 다른 웹 API를 통해 받아올 수 없기 때문이다.

XMLHttpRequest에서는 크로스 도메인 통신을 할 수 없지만 대체 수단이 있다. 
```
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <script src="http://example.jp/map.js"></script> 
    <script src="http://example.jp/zip.js"></script>
    ...
  </head>
</html>
```
HTML의 script 요소를 이용하면 복수의 사이트에서 JavaScript 파일을 읽을 수 있다. script 요소는 역사적인 이유에서 일반적으로 브라우저의 보안제한을 받지 않는다.

JSONP는 브라우저의 이런 성질을 이용해 크로스 도메인 통신을 구현하는 방법이다. JSONP에서는 오리지널 JSON을 클라이언트가 지정한 콜백 함수명으로 랩핑하여 도메인이 다른 서버로부터 데이터를 취득한다. 
```
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
     <title>크로스 도메인 통신의 예</title>
  </head>
  <body>
    <script type="text/javascript">
      function foo(zip){
        alert(zip["zipcode"]);
      }
    </script>
    <script src="http://zip.ricollab.jp/1120002.json?callback=foo"></script>
  </body>
</html>
```
브라우저에서 이 html파일을 렌더링하면 두번째 script 요소인 JSONP의 URI를 자동적으로 GET하여 다음 통신이 수행된다.
```
요청
GET /1120002.json?callback=foo HTTP/1.1
Host: zip.ricollab.jp
```
```
응답
HTTP/1.1 200 OK
Content-Type: application/javascript

foo({
  "zipcode": "1120002",
  "address": {
    "prefecture": "도쿄도",
    "city": "분쿄구",
    "town": "코이시카와"
    },
  "yomi":{
    "prefecture": "토우쿄우토",
    "city": "분쿄우쿠",
    "town": "코이시카와"
    }
  });
```
요청한 URI에는 callback이라는 쿼리 파라미터에서 콜 백 함수로 foo를 지정하고 있기 때문에 응답의 바디에는 foo 함수를 호출하는 JavaScript의 코드가 들어 있다. foo 함수의 인수로는 요청 URI에서 지정한 우편번호인  1120002가 JSON으로 들어 있다. 이에 따라 첫 번째 script 요소에서 정의한 foo 함수를 1120002라는 우편번호 정보를 인수로 하여 호출하고, 결과로 브라우저가 1120002라는 알림창을 표시한다.

foo함수를 정의한 HTML 파일은 우편번호 정보를 zip.ricollab.jp에서 가져오고 있다는 점을 주목하자. 이것이 JSONP로 크로스 도메인 통신을 구현하는 방법이다. 이 예에서는 콜백 함수를 지정한 script 요소를 HTML에 직접 삽입하고 있지만, 보통은 사용자의 입력에 대응해 JavaScript로 HTML을 조적해서 동적으로 삽입한다.

## 리다이렉트 / 포워딩 / 세션 / 쿠키
**포워딩 :** Web Container 차원에서 페이지 이동만 있다. 실제로 웹 브라우저는 다른 페이지로 이동했음을 알 수 없다. 그렇기 때문에 웹 브라우저에는 최초에 호출한 URL이 표시되고 이동한 페이지의 URL 정보는 볼 수 없다. 동일한 웹 컨테이너에 있는 페이지로만 이동할 수 있다.

**리다이렉트 :** Web Container는 Redirect 명령이 들어오면 웹 브라우저에게 다른 페이지로 이동하라고 명령을 내린다.(response header인 location에 리다이렉트될 주소 적어서 보낸다) 그러면 웹 브라우저는 URL을 지시된 주소로 바꾸고 그 주소로 이동한다. 다른 Web Container에 있는 주소로 이동이 가능하다. 

**세션 :** 방문자의 요청에 따른 정보를 방문자 메모리에 저장하는 것이 아닌 웹 서버가 세션 아이디 파일을 만들어 서비스가 돌아가고 있는 서버에 저장하는 것이다. 즉 프로세스들 사이에서 통신을 하기 위해 메세지 교환을 통해 서로를 인식한 이후부터 통신을 마칠때까지의 기간 동안 서버에 잠시 방문자 정보를 저장 한다는 것. (웹사이트에 방문하여 계속 접속을 유지할 때 이전의 접속 정보를 이용할 수 있는 방법으로 많이 사용)

**쿠키 :** 특정 웹 사이트를 방문 했을 때 만들어지는 정보를 담는 파일을 지칭(상태정보를 유지하는 기술).
ex) 방문 했던 사이트에 다시 방문 하였을 때 아이디와 비밀번호 자동 입력, 팝업에서 “오늘 이 창을 다시 보지 않음” 체크