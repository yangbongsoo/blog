###3초마다 체크 
###총 10단계 
###30초 주기로 UdpDataSender 패킷 Send


---


**1. PinpointSocketFactory$ConnectEvent Try reconnect. Connectaddress:/52.69.198.82:29994**
```
public PinpointSocket scheduledConnect(String host, int port) {
        PinpointSocket pinpointSocket = new PinpointSocket(new ReconnectStateSocketHandler());
        SocketAddress address = new InetSocketAddress(host, port);
        reconnect(pinpointSocket, address);
        return pinpointSocket;
    }
```

```
void reconnect(final PinpointSocket pinpointSocket, final SocketAddress socketAddress) {
        ConnectEvent connectEvent = new ConnectEvent(pinpointSocket, socketAddress);
        timer.newTimeout(connectEvent, reconnectDelay, TimeUnit.MILLISECONDS);
    }
```
reconnectDelay = 3초  

```
package org.jboss.netty.util;

import java.util.Set;
import java.util.concurrent.TimeUnit;
import org.jboss.netty.util.Timeout;
import org.jboss.netty.util.TimerTask;

public interface Timer {
    Timeout newTimeout(TimerTask var1, long var2, TimeUnit var4);

    Set<Timeout> stop();
}
```
jboss 오픈소스를 통해 timeout 설정하는건가 

