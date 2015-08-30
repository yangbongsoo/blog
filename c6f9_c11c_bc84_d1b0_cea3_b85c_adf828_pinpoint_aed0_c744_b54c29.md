###3초마다 체크 
###총 10단계 
###30초 주기로 UdpDataSender 패킷 Send


---


**1. PinpointSocketFactory$ConnectEvent Try reconnect. Connectaddress:/52.69.198.82:29994**
```
public PinpointSocket scheduledConnect(String host, int port) {
        PinpointSocket pinpointSocket = new PinpointSocket(new ReconnectStateSocketHandler());
        SocketAddress address = new InetSocketAddress(host, port);
        reconnect(pinpointSocket, address); <ㅡㅡ 여기
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
jboss 오픈소스를 통해 timeout 설정하는건가 ???  잘 모르겠네 


timeout이 발생하면 ConnectEvent 클래스의 run메소드가 실행되는거 같다.
```
        @Override
        public void run(Timeout timeout) {
            if (timeout.isCancelled()) {
                return;
            }

            // Just return not to try reconnection when event has been fired but pinpointSocket already closed.
            if (pinpointSocket.isClosed()) {
                logger.debug("pinpointSocket is already closed.");
                return;
            }

            logger.warn("try reconnect. connectAddress:{}", socketAddress);
            final ChannelFuture channelFuture = reconnect(socketAddress);
            Channel channel = channelFuture.getChannel();
            final SocketHandler socketHandler = getSocketHandler(channel);
            socketHandler.setConnectSocketAddress(socketAddress);
            socketHandler.setPinpointSocket(pinpointSocket);

            channelFuture.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    if (future.isSuccess()) {
                        Channel channel = future.getChannel();
                        logger.warn("reconnect success {}, {}", socketAddress, channel);
                        pinpointSocket.reconnectSocketHandler(socketHandler);
                    } else {
                         if (!pinpointSocket.isClosed()) {

                         /*
                            // comment out because exception message can be taken at exceptionCaught
                            if (logger.isWarnEnabled()) {
                                Throwable cause = future.getCause();
                                logger.warn("reconnect fail. {} Caused:{}", socketAddress, cause.getMessage());
                            }
                          */
                            reconnect(pinpointSocket, socketAddress);
                        } else {
                            logger.info("pinpointSocket is closed. stop reconnect.");
                        }
                    }
                }
            });
        }
```

