#Apache httpd.conf
```

ServerRoot "/usr" (mac에 기본적으로 깔려있는 apache 기준)

...

#Listen 12.34.56.78:80
Listen 80

...

LoadModule jk_module /private/etc/apache2/other/mod_jk.so

#'Main' server configuration 
이 섹션의 지시문은 <main> 서버가 사용하는 값을 설정하며, 
<VirtualHost> 정의에 의해 처리되지 않는 요청에 응답한다.
또한 이 값은 나중에 파일에 정의 할 수있는 <VirtualHost> 컨테이너의 기본값을 제공한다.
이러한 지시어는 모두 <VirtualHost> 컨테이너 안에 나타날 수 있는데, 가상 호스트가 정의 될 때 기본 설정은 무시된다.

ServerAdmin you@example.com
ServerName www.example.com:80

#문서를 제공할 디렉토리다. 기본적으로 모든 요청은 디렉토리에서 취해 지지만
#심볼릭 링크와 별칭을 사용하여 다른 위치를 가리킬 수도 있다.
DocumentRoot "/abc/def/ght"

<Directory />
    Options FollowSymLinks
    AllowOverride none
    Order deny,allow
    Deny from all
</Directory>
```
**ServerRoot :** 서버의 설정, 에러, 로그파일들이있는 디렉토리 트리의 맨 위
**Listen :** 아파치를 디폴트가 아닌 특정 IP 주소 나 포트에 바인드 할 수도 있다(prevent Apache from glomming onto all bound IP addresses).
**LoadModule jk_module :** 본인 pc에 디폴트로 mod_jk가 없어서 so 파일을 구해다 other 디렉토리에 넣었다.