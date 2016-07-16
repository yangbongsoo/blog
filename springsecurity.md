# Spring 4.0 프로그래밍 ch16 Spring Security (최범균 저)

o.s.s.core.Authentication은 스프링 시큐리티에서 현재 애플리케이션에 접근한 사용자(더 정확하게는 웹 브라우저, REST로 접근한 외부 시스템 등)의 보안 관련 정보를 보관하는 역할을 한다. 예를 들어, Authentication은 사용자의 인증 여부, 사용자가 가진 권한(authority), 이름 및 접근 주체(principal)에 대한 정보를 제공한다. 스프링 시큐리티는 이 정보를 이용해서 사용자가 요청한 자원(URL 등)에 접근 할 수 있는지 여부를 판단한다.

스프링 시큐리티가 Authentication을 사용하려면 다음과 같다.
```
SecurityContext context = SecurityContextHolder.getContext();
Authentication auth = context.getAuthentication();
```
SecurityContext는 Authentication 객체를 보관하는 역할을 한다. 스프링 시큐리티는 클라이언트로부터 요청이 들어오면 서블릿 필터를 이용해서 SecurityConext에Authentication 객체를 설정한다. 이를 위한 서블릿 필터를 직접 구현한다면 아마 다음과 같은 코드를 만들어야 할 것이다.
```
//웹 요청을 가장 먼저 받는 필터에서 Authentication을 생성해서 SecurityContext에 보관
Authentication auth = someMethodForGettingAuth(request, response);
try {
	SecurityContextHolder.getContext().setAuthentication(auth);
	chain.doFilter(request, response);
} finally {
	SecurityContextHolder.clearContext();
}
```
**하지만 대부분은 스프링 시큐리티가 제공한는 SecurityContextPersistenceFilter 클래스의 기능을 활용해서 SecurityContext를 설정하기 위한 코드 양을 줄일 수 있다.** 여러 보안 필터 체인 중 가장 먼저 적용해서 SecurityContext에 Authentication 객체를 보관한다. 이후 스프링 시큐리티는 SecurityContext를 이용해서 현재 접속한 사용자의 Authentication 객체를 구하고, Authentication 객체가 표현하는 주체가 접속한 자원에 접근할 수 있는지 여부를 판단하게 된다. 

**Authentication 인터페이스**<br>
스프링 시큐리티가 제공하는 코드가 아닌 우리가 만든 코드에서 직접 o.s.s.core.Authentication 객체를 사용해야 할 때가 있다. 이런 경우에는 앞서 살펴본 SecurityContextHolder를 이용해서 Authentication 객체를 구하면 된다.
```
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
```
Authentication 객체를 구했다면, Authentication 객체가 제공하는 메서드를 이용해서 필요한 정보를 구할 수 있다. 이들 메서드는 다음과 같다.
```
String getName() : 이름을 구한다.
Object getCrendentials() : 인증 대상 주체를 증명하는 값을 구한다. 비밀번호 등이 이에 해당된다.
Object getPrincipal() : 주체를 표현하는 객체를 구한다.
Object getDetails() : 주체에 대한 상세 정보를 구한다. 접속한 IP 등의 정보를 저장하는 용도로 사용할 수 있다.
boolean isAuthenticated() : 인증 여부를 리턴한다.
void setAuthenticated(boolean authenticated) : 인증 여부를 설정한다.
Collection<? extends GrantedAuthority> getAuthorities() : 주체가 가진 권한 목록을 구한다. GrantedAuthority가 권한을 의미한다.
```
Authentication 타입은 다음의 두 가지 목적으로 사용된다.
1. AuthenticationManager에 인증을 요청할 때 필요한 정보를 담는 목적
  1. 스프링 시큐리티는 인증을 위한 목적으로 Authentication 객체를 사용한다. 다시 말해 AuthenticationManager를 사용해서 인증을 처리하는데, 이 AuthenticationManager가 입력으로 받는 값의 타입이 Authentication이다. 
2. 현재 접속한 사용자에 대한 정보를 표현하기 위한 목적 

예를 들어 스프링 시큐리티는 **UsernamePasswordAuthenticationToken** 클래스를 제공하고 있는데, 이 클래스는 사용자 아이디와 암호를 이용해서 인증을 처리할 때 사용되는 Authentication 구현체다. 또한 **AnonymousAuthenticationToken** 클래스를 제공하고 있는데 이 클래스는 아직 인증을 거치지 않은 사용자를 표현하기 위한 구현체로 사용된다. 스프링 시큐리티가 이미 다양한 상황에 맞게 사용할 수 있는 Authentication 구현 클래스를 제공하고 있지만, 필요에 따라 직접 알맞은 Authentication 구현체를 구현해야 할 때도 있다.

**GrantedAuthority 인터페이스**<br>
o.s.s.core.GrantedAuthority 인터페이스는 권한을 표현할 때 사용된다. 앞서 Authentication 인터페이스의 getAuthorities() 메서드는 사용자가 가진 권한 목록을 리턴한다고 했는데, 이때 GrantedAuthority를 사용했다. 
```
public interface GrantedAuthority extends Serializable {
  String getAuthority();
}
```
GrantedAuthority의 getAuthority() 메서드의 리턴 타입은 String인데, 이는 스프링 시큐리티가 모든 권한을 문자열로 표현한다는 것을 의미한다.
```
// 접근 권한을 설정한 코드
<sec:intercept-url pattern="/admin/usermanager/**"
    access="hasAuthority('USER_MANAGER')"/>
```
SimpleGrantedAuthority 클래스는 GrantedAuthority 타입의 객체를 직접 생성해야 할 때 사용할 수 있는 클래스로서, 이 클래스는 생성자를 이용해서 String 타입의 권한 값을 전달받는다.
```
GrantedAuthority authority = new SimpleGrantedAuthority("USER_MANAGER");
```

###보안 필터 체인
web.xml설정에서 
```
<filter>
  <filter-name>springSecurityFilterChain</filter-name>
  <filter-class>
    org.springframework.web.filter.DelegatingFilterProxy
  </filter-class>
</filter>
<filter-mapping>
  <filter-name>springSecurityFilterChain</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```
**이름이 springSecurityFilterChain인 DelegatingFilterProxy를 서블릿 필터로 등록한다.** 이 필터는 스프링 빈 객체를 필터로 쓰고 싶을 때 사용하는데, 위 설정에서 사용할 스프링 빈의 이름을 springSecurityFilterChain으로 설정했다.  

cf) 스프링 시큐리티가 제공하는 JSP용 커스텀 태그 라이브러리가 정상 작동하려면 스프링 시큐리티의 주요 구성 요소가 루트 애플리케이션에 위치해야 한다.

spring security 의존성 
```
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-web</artifactId>
	<version>4.1.1.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-config</artifactId>
	<version>4.1.1.RELEASE</version>
</dependency>
```
```
compile 'org.springframework.security:spring-security-web:4.1.1.RELEASE'
compile 'org.springframework.security:spring-security-config:4.1.1.RELEASE'
```