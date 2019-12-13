# InetAddress 

InetAddress.getLocalHost(); 메서드는 실제 호스트 네임과 IP 주소를 얻기 위해 DNS로 연결을 시도하는데 해당
정보를 얻는데 실패할 경우, 루프백 주소를 반환한다(호스트네임: localhost, ip : 127.0.0.1)

IndetAddress 클래스는 검색의 결과를 저장(cache)한다. IndetAddress 클래스는 특정 호스트에 대한 주소를 검색하여 저장한 다음에는 추가 요청에 대해서 다시 검색하지 않는다.
같은 호스트에 대한 새로운 IndetAddress 객체를 만든 경우에도 마찬가지다.

networkaddress.cache.ttl - 성공한 DNS 쿼리에 대해 자바가 저장할 시간을 초 단위로 설정
networkaddress.cache.negative.ttl - 실패한 DNS 쿼리에 대해 자바가 저장할 시간을 초 단위로 설정

