#Apache httpd.conf
```

ServerRoot "/usr" (mac에 기본적으로 깔려있는 apache 기준)
...

#Listen 12.34.56.78:80
Listen 80

```
ServerRoot : 서버의 설정, 에러, 로그파일들이있는 디렉토리 트리의 맨 위
Listen : 아파치를 디폴트가 아닌 특정 IP 주소 나 포트에 바인드 할 수도 있다(prevent Apache from glomming onto all bound IP addresses).
