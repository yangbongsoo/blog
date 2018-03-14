### 22. Spring Security X-Frame-Options 이슈

**Default Security Headers**
```
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```
cf) Strict-Transport-Security는 HTTPS 요청일때만 추가된다. 

```
X-Frame-Options: DENY
```
response header에 X-Frame-Options를 갖고 있는 모든 사이트는 iframe 안에서 렌더링 되지 못하도록 브라우저가 막는다.

**customize**
```
X-Frame-Options: SAMEORIGIN
X-Frame-Options: ALLOW-FROM https://example.com/
```
SAMEORIGIN : 같은 도메인일때만 ifrmae을 허용한다.
```xml
<http>
	<!-- ... -->

	<headers>
		<frame-options
		policy="SAMEORIGIN" />
	</headers>
</http>
```

```java
http
	// ...
	.headers()
		.frameOptions()
			.sameOrigin();
```
ALLOW-FROM : 구체적인 도메인 지정

**Solution**
```xml
<http>
	<!-- ... -->

	<headers disabled="true" />
</http>

```
```java
http.headers().frameOptions().disable()
```

### 23. HTTP Strict Transport Security (HSTS)

https://docs.spring.io/spring-security/site/docs/4.0.x/reference/html/headers.html#headers-hsts

웹 브라우저가 HTTPS 프로토콜만을 사용해서 서버와 통신하도록 하는 기능을 한다. 만약 HTTPS로 접속에 실패하면 사이트 접근에 실패하게 된다. 서버가 HTTP 응답 헤더에 `Strict-Transport-Security`를 내려주면 브라우저는 그 사이트에 접속할 때 무조건 HTTPS로만 연결한다.

HSTS가 적용되기 위해서는 서버도 헤더를 내려줘야하고 브라우저도 그 헤더에 따른 동작을 해야하는 것이다.
HSTS를 사용하는 대신 서버에서 HTTP 접속을 HTTPS로 redirect 시키는 방법이 있지만 일단 한번 HTTP 연결을 거쳐가는 것이기 때문에 쿠키 정보 탈취 등 보안에 취약하다.

```
Strict-Transport-Security: max-age=31536000 ; includeSubDomains; preload
```
max-age : 지정 시간(단위 초)만큼 HTTPS를 사용
includeSubdomains : HSTS를 서브 도메인에도 적용
preload : 브라우저가 해당 사이트를 HSTS 적용 preload list에 추가하도록 함

**preload list**

HTTPS로 웹 사이트에 접속하기 위해 적어도 한번 웹 서버와 통신을 해야하는데, 통신 해보기 전에 미리 HTTPS로 접속하도록 목록을 미리 만들어둔 것이다.

이는 브라우저가 지원해주는 기능이고 브라우저 안에 기본적으로 내장되어 있는 사이트 목록들이 있다. 그리고 서버가 헤더에 preload값을 내려주면 이 목록에 해당 사이트도 추가하게 되는 것이다(브라우저에 캐시됌). preload에 추가된 사이트는 서버가 HSTS헤더를 삭제해도 브라우저에는 설정이 유지된다.

만약 내장되는 preload list에 추가되면 삭제되기까지(목록 삭제 및 사용자 브라우저 업데이트) 시간이 오래 걸리므로 추가는 신중하게 해야한다. preload list는 크롬이 관리하고 있고 대부분의 브라우저들(파이어폭스, 오페라, 사파리, IE11, Edge)이 이 목록을 같이 사용한다.

만약 수동으로 해제하고자 한다면 크롬은 `chrome://net-internals/#hsts`에 들어가서 Delete Domain에서 삭제할 도메인을 입력하고 삭제하면 된다.
```xml
<http>
<!-- ... -->

<headers>
	<hsts include-subdomains="true" max-age-seconds="31536000" />
</headers>
</http>

```
```java
@EnableWebSecurity
public class WebSecurityConfig extends
WebSecurityConfigurerAdapter {

@Override
protected void configure(HttpSecurity http) throws Exception {
	http
	// ...
	.headers()
	.httpStrictTransportSecurity()
	.includeSubdomains(true)
	.maxAgeSeconds(31536000);
	}
}

```

### 24. List에 있는 value를 Mybatis foreach로 insert 하기
```java
public class TestDto {
    private List<String> list;

    // getter & setter
}
```
```xml
    <insert id="insertTest" parameterType="com.test.dto.TestDto">
        INSERT
        INTO test_table (id, order)
        VALUES
        <foreach collection="list" item="value" index="order" open="" separator="," close="">
            (#{value}, #{order})
        </foreach>
    </insert>
```

### 25. 컨트롤러에서 List 객체 받는 방법 정리
**방법1**
```js
$.ajax({
    url: "/test1",
    type: "post",
    contentType: 'application/json;charset=UTF-8',
    data: JSON.stringify(list)
    }).done(function (data, status, xhr) {
    ...
```
```java
@RequestMapping(value = "/test1", produces = {"application/json;charset=UTF-8"}, method = RequestMethod.POST)
public void test1(@RequestBody List<String> list) {
    ...
}
```
**방법2**
```js
$.ajax({
    url: "/test2",
    type: "post",
    contentType: 'application/json;charset=UTF-8',
    data: JSON.stringify({
        name: yangbongsoo,
        age: 29,
        list: list
        })
    }).done(function (data, status, xhr) {
    ...

```
```java
@RequestMapping(value = "/test2", produces = {"application/json;charset=UTF-8"}, method = RequestMethod.POST)
public void test2(@RequestBody TestDto testDto) {
    ...
}
```
```java
public class TestDto {
    private String name;
    private int age;
    private List<String> list;

    //getter and setter
}
```