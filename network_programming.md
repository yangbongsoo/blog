# Network Programming

## 동기 / 비동기 , 블로킹 / 논블로킹
출처 : https://www.youtube.com/watch?v=HKlUvCv9hvA <br>
출처 : 자바8인액션 p340 <br>

**동기(synchronous)** : syn 은 함께 라는 뜻. chrono 는 시간 이라는 뜻. 즉 함께 시간을 맞춘다. <br>
**비동기(asynchronous)** : 시간을 맞추지 않는다 는 뜻 <br>

그러므로 동기 비동기에 대한 이야기를 할 때는 반드시 두 가지 이상을 언급해야한다. <br>
무엇과 무엇이? 그리고 시간 개념이 있어야 한다. 어떤 시간을 맞춘다(동기) 안맞춘다(비동기). <br>

동기라는 것은 예를 들어서 A, B 가 함께 간다고 할 때,
두 가지가 동시에 시작하거나 동시에 종료하거나 동시에 같이 진행을 하면 동기다.

동기 예1) A, B 스레드가 동시에 작업을 시작하게 하는 CyclicBarrier

동기 예2) 메서드 리턴 시간(A)과 결과를 전달받는 시간(B)가 일치하면 동기

동기 예3) A가 작업을 시작하고 끝나면 끝나는 시간에 맞춰서 B가 시작하는 개념 (Synchronized 블록, BlockingQueue)


**블록킹, 논블로킹** <br>
동기, 비동기와는 관점이 다르다. 동기, 비동기에서 항상 블로킹, 논블로킹을 얘기할 수 있는게 아니다. <br>
같이 언급할 수 있는 경우는 `내가 직접 제어할 수 없는 대상을 어떻게 상대할 것이냐 이 부분을 설명할 때` 이다. <br>
이때는 블로킹과 논블로킹 얘기를 같이 해도 된다. 그래서 대상이 제한적이다. <br>
여기서 말하는 `내가` 의 나는 스레드 이다. 즉 스레드 관점에서 봐야한다. 

```java
ExecutorService es = Executors.newCachedThreadPool();
String res = es.submit(() -> "Hello Async").get();
```

위의 코드를 동기/비동기 , 블로킹/논블로킹 관점에서 설명해보면 다음과 같다.

`es.submit()` 메서드는 비동기다. 메서드 리턴 시간과 Callable의 실행 결과를 받는 시간이 일치하지 않는다. <br>
블로킹, 논블로킹은 고려할 대상이 아니다(억지로 하려면 할 수 있겠지만). 이거는 뭔가를 기다릴 대상이 없다. <br>
그러므로 단순히 비동기 메서드 호출이다. <br>

`.get()` 메서드는 비동기에 대한 결과를 동기로 가져온다(결과가 나올때까지 기다림). <br>
메서드 리턴시간과 결과를 가져오는 시간이 일치한다. 즉, get 메서드는 동기다. <br>
그리고 내가 비동기적으로 실행시킨 제3의 스레드에서 결과를 만들어서 값을 리턴할 때까지 블록상태에 있다(대기 상태에 들어감). <br>
그러므로 블로킹이다. <br>

```java
@Async
public String asyncCall() {
    restTemplate.exchange();
}
```

위의 코드는 asyncCall() 메서드를 비동기로 실행 시키는 메인 스레드에서 
asyncCall() 메서드를 호출하고, 결과를 기다렸나 안기다렸나에 따라 블로킹/논블로킹을 이야기 할 수 있다.
그리고 asyncCall() 메서드 안에서 restTemplate 으로 쏘고 결과를 기다리는건 동기이다.



cf) 자바8인액션에서는 동기 API, 비동기 API를 다음과 같이 설명한다.

전통적인 동기 API에서는 메서드를 호출한 다음에 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행한다.
호출자와 피호출자가 각각 다른 스레드에서 실행되는 상황이었더라도 호출자는 피호출자의 동작 완료를 기다렸을 것이다. 이처럼 **동기 API를 사용하는 상황을
블록 호출(blocking call)이라고 한다.**

반면 비동기 API에서는 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다.
이와 같은 **비동기 API를 사용하는 상황을 비블록 호출(non-blcoking call)이라고 한다.**


## 병렬성(parallelism), 동시성(concurrency)
출처 : 자바8인액션 P337

![](/assets/network-programming1.png)

## SSL 관련 예외

브라우저에서는 신뢰할 수 있는 인증서로 나오지만 자바에서는 `unable to find valid certification path to requested target`
으로 예외가 발생하는 경우가 있다. 자바는 신뢰할 수 있는 인증서 목록을 자체적으로 갖고 있는데 해당 파일은 아래의 스크립트 명령어로 확인 할 수 있다.
비밀번호를 물어보면 디폴트 패스워드 changeit 입력하면 된다.
```
$ cd $JAVA_HOME/bin` 
$ ./keytool -list -keystore ../jre/lib/security/cacerts`
```


또 https://10.1.2.3 와 같은 IP로 입력했을 때 안되는 이슈가 있는데 

SSL 관련 검증 로직을 안타게 하기 위해서 아래와 같은 설정이 있는데

일반적으로 프로덕션 코드에서는 이걸 쓰면 안되지만 프록시 서버라면 잘못된거더라도 그대로 통과시켜야 할 때가 있는데

그런점을 고려해봤을 때 아래와 같이 끄는 방법도 있다. 

참고) https://gs.saro.me/dev?tn=331

apache httpclient

```java
SSLContextBuilder sslContextBuilder = new SSLContextBuilder();
sslContextBuilder.loadTrustMaterial(null, (TrustStrategy)(x509Certificates, s) -> true);
SSLConnectionSocketFactory sf = new SSLConnectionSocketFactory(sslContextBuilder.build(), new NoopHostnameVerifier());
CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sf).build();

HttpGet httpGet = new HttpGet("https://test.com");
CloseableHttpResponse httpResponse = httpClient.execute(httpGet);
```

url connection

```java
URL url = new URL(lgr);
HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
connection.setRequestMethod("GET");

SSLContext sc = SSLContext.getInstance("SSL");
sc.init(null, createTrustManagers(), new java.security.SecureRandom());
connection.setSSLSocketFactory(sc.getSocketFactory());


public static TrustManager[] createTrustManagers() {
	return new TrustManager[] {new X509TrustManager() {
		public void checkClientTrusted(java.security.cert.X509Certificate[] x509Certificates, String s) {}

		public void checkServerTrusted(java.security.cert.X509Certificate[] x509Certificates, String s) {}

		public java.security.cert.X509Certificate[] getAcceptedIssuers() {
			return new java.security.cert.X509Certificate[] {};
		}
	}};
}
```