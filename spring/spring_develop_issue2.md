# issue22 ~

## 22. Spring Security X-Frame-Options 이슈

**Default Security Headers**

```text
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```

cf\) Strict-Transport-Security는 HTTPS 요청일때만 추가된다.

```text
X-Frame-Options: DENY
```

response header에 X-Frame-Options를 갖고 있는 모든 사이트는 iframe 안에서 렌더링 되지 못하도록 브라우저가 막는다.

**customize**

```text
X-Frame-Options: SAMEORIGIN
X-Frame-Options: ALLOW-FROM https://example.com/
```

SAMEORIGIN : 같은 도메인일때만 ifrmae을 허용한다.

```markup
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

```markup
<http>
    <!-- ... -->

    <headers disabled="true" />
</http>
```

```java
http.headers().frameOptions().disable()
```

## 23. HTTP Strict Transport Security \(HSTS\)

[https://docs.spring.io/spring-security/site/docs/4.0.x/reference/html/headers.html\#headers-hsts](https://docs.spring.io/spring-security/site/docs/4.0.x/reference/html/headers.html#headers-hsts)

웹 브라우저가 HTTPS 프로토콜만을 사용해서 서버와 통신하도록 하는 기능을 한다. 만약 HTTPS로 접속에 실패하면 사이트 접근에 실패하게 된다. 서버가 HTTP 응답 헤더에 `Strict-Transport-Security`를 내려주면 브라우저는 그 사이트에 접속할 때 무조건 HTTPS로만 연결한다.

HSTS가 적용되기 위해서는 서버도 헤더를 내려줘야하고 브라우저도 그 헤더에 따른 동작을 해야하는 것이다. HSTS를 사용하는 대신 서버에서 HTTP 접속을 HTTPS로 redirect 시키는 방법이 있지만 일단 한번 HTTP 연결을 거쳐가는 것이기 때문에 쿠키 정보 탈취 등 보안에 취약하다.

```text
Strict-Transport-Security: max-age=31536000 ; includeSubDomains; preload
```

max-age : 지정 시간\(단위 초\)만큼 HTTPS를 사용

includeSubdomains : HSTS를 서브 도메인에도 적용

preload : 브라우저가 해당 사이트를 HSTS 적용 preload list에 추가하도록 함

**preload list**

HTTPS로 웹 사이트에 접속하기 위해 적어도 한번 웹 서버와 통신을 해야하는데, 통신 해보기 전에 미리 HTTPS로 접속하도록 목록을 미리 만들어둔 것이다.

이는 브라우저가 지원해주는 기능이고 브라우저 안에 기본적으로 내장되어 있는 사이트 목록들이 있다. 그리고 서버가 헤더에 preload값을 내려주면 이 목록에 해당 사이트도 추가하게 되는 것이다\(브라우저에 캐시됌\). preload에 추가된 사이트는 서버가 HSTS헤더를 삭제해도 브라우저에는 설정이 유지된다.

만약 내장되는 preload list에 추가되면 삭제되기까지\(목록 삭제 및 사용자 브라우저 업데이트\) 시간이 오래 걸리므로 추가는 신중하게 해야한다. preload list는 크롬이 관리하고 있고 대부분의 브라우저들\(파이어폭스, 오페라, 사파리, IE11, Edge\)이 이 목록을 같이 사용한다.

만약 수동으로 해제하고자 한다면 크롬은 `chrome://net-internals/#hsts`에 들어가서 Delete Domain에서 삭제할 도메인을 입력하고 삭제하면 된다.

HSTS 설정 코드

```markup
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

HSTS disable 코드

```markup
<http>
    <!-- ... -->

    <headers>
        <hsts disable="true"/>
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
                .httpStrictTransportSecurity().disable();
    }
}
```

## 24. List에 있는 value를 Mybatis foreach로 insert 하기

```java
public class TestDto {
    private List<String> list;

    // getter & setter
}
```

```markup
    <insert id="insertTest" parameterType="com.test.dto.TestDto">
        INSERT
        INTO test_table (id, order)
        VALUES
        <foreach collection="list" item="value" index="order" open="" separator="," close="">
            (#{value}, #{order})
        </foreach>
    </insert>
```

## 25. 컨트롤러에서 List 객체 받는 방법 정리

**방법1**

```javascript
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

```javascript
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

## 26. Mac에서 Spring Boot의 시작이 느릴 때

Mac에서 Spring Boot의 시작이 느리다면 아래와 같이 hostname을 지정해 본다.

```text
sudo scutil --set HostName MyMacBook
```

Spring Boot의 StartupInfoLogger 에서는 InetAddress.getLocalHost\(\).getHostName\(\);를 호출한다. Mac에서는 Hostname이 지정되어 있지 않을 경우 해당 메서드 호출에 몇초가 걸리는 것으로 파악된다.

## 27. Tomcat8 UMASK 이슈

스프링 이슈는 아니지만 따로 적을 곳이 없어서 여기에 적음

파일 업로드를 할 때 업로드된 파일 권한이 위에서 아래로 바뀌는 현상 발견

```text
-rw-r--r--
-rw-r-----
```

원인을 찾아보니 tomcat 7.0.68 catalina.sh에서는 UMASK가 주석처리 되어있었는데

```text
#JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"
```

tomcat 8.5.32 catalina.sh에서 UMASK에 대한 부분이 바꼈다.

```text
# Set UMASK unless it has been overridden
if [ -z "$UMASK" ]; then
    UMASK="0027"
fi
umask $UMASK

...

# Make the umask available when using the org.apache.catalina.security.SecurityListener
JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"
```

문제가 되는 이유는, 아파치 httpd.conf에서 User nobody를 할 경우에 640이면 파일 read를 못하게 된다. 기존 tomcat7에서는 read에 문제가 되지 않았던 부분이다.

```text
file 666 - 022 = 644(-rw-r--r--)
file 666 - 027 = 640(-rw-r-----)
```

cf\) 027은 `000 010 111`이고 보수로 전환하면 `111 101 000`이 된다. UMASK 보수 값과 파일 기본 허가권을 AND 연산하면

```text
110 110 110
111 101 000
-----------
110 100 000
```

640이 된다.

