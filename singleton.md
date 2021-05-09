# Singleton

Singleton 내용은 티스토리로 옮김
https://yangbongsoo.tistory.com/25


## Double checked locking

악명 높은 피해야 할 패턴이다\(하지만 이펙티브 자바 규칙 71에서는 사용에 문제삼지 않는다\). 굉장히 초기에 사용하던 JVM은 경쟁이 별로 없는 상태라고 해도 동기화를 맞추려면 성능에 엄청난 영향을 주었다. 그 결과 동기화 기법이 주는 영향을 최소화하고자 하는 여러 가지 기발한 방법이 나타나기 시작했다. 일부 괜찮은 방법도 있었고, 안좋은 방법도 있었고, 정말 문제 많은 방법도 있었다. DCL은 정말 문제 많은 방법에 속하는 놈이다.

```java
@NotThreadSafe 
public class DoubleCheckedLocking { 
     private static Resource resource; 
     public static Resource getInstance() { 
          if (resource == null) { 
              synchronized(this) { 
                   if (resource == null)  
                        resource = new Resource(); 
              }  
          } 
          return resource; 
     } 
}
```

DCL은 먼저 동기화 구문이 없는 상태로 초기화 작업이 필요한지를 확인하고, resource 변수의 값이 null이 아니라면\(다시 말해 초기화가 돼 있다면\) resource 변수에 참조된 객체를 사용한다. 만약 초기화 작업이 필요하다면 동기화 구문을 사용해 락을 걸고 Resource 객체가 초기화됐는지 다시 한 번 확인하는데, 이렇게 하면 Resource 객체를 초기화하는 작업은 한 번에 하나의 스레드만 가능하긴 하다.

여기서 가장 자주 사용하는 부분, 즉 이미 만들어진 Resource 인스턴스에 대한 참조를 가져오는 부분은 동기화돼 있지 않았다. 바로 이 부분이 문제인데 ‘부분 구성’된 Resource 인스턴스를 사용하게 될 가능성이 있다. 그래서 이미 초기화된 필드에는 락을 걸지 않으므로, 필드는 반드시 volatile로 선언해야 한다.

DCL이 갖고 있는 더 큰 문제는 동기화돼 있지 않은 상태에서 발생할 수 있는 가장 심각한 문제가 스테일 값\(여기서는 null\)을 사용할 가능성이 있는 정도에 불과하다고 추정하고 있다는 점이다. 만약 stale 값을 사용하는 경우가 발생하면 락을 확보한 채로 재시도해도 문제를 해결한다. 하지만 실제 최악의 상황은 추정했던 최악의 상황보다 더 심각하다. 즉, 현재 객체에 대한 참조를 제대로 본다 하더라도 객체의 상태를 볼 때 스테일 값을 보게되는 경우, 즉 참조된 객체 내부의 상태가 올바르지 않은 상태일 경우가 생길 수 있다.

자바 1.5 이후 수정된 자바 메모리 모델의 내용을 보면 resource 변수를 volatile로 선언했을 때는 DCL마저 정상적으로 동작한다. 또한 volatile 변수에 대한 읽기 연산은 volatile이 아닌 변수의 읽기 연산보다 자원을 아주 조금 더 사용할 뿐이기 때문에 성능에 미치는 영향도 미미하다.

## Volatile과 Syncronized

1.자바 병렬프로그래밍 3장 객체공유 p67

소스코드의 특정 블록을 동기화시키고자 할 때는 항상 메모리 가시성\(memory visibility\)문제가 발생하는데, 큰 문제일 경우도 있고 별로 문제가 되지 않을 수도 있다. 다시 말하자면 특정 변수의 값을 사용하고 있을 때 다른 스레드가 해당 변수의 값을 사용하지 못하도록 막아야 할 뿐만 아니라, 값을 사용한 다음 동기화 블록을 빠져나가고 나면 다른 스레드가 변경된 값을 즉시 사용할 수 있게 해야 한다는 뜻이다.

가시성은 그다지 직관적으로 이해할 수 있는 문제가 아니기 때문에 흔히 무시하고 넘어가는 경우가 많다. 만약 단일 스레드만 사용하는 환경이라면 특정 변수에 값을 지정하고 다음번에 해당 변수의 값을 다시 읽어보면, 이전에 저장해뒀던 바로 그 값을 가져올 수 있다. 대부분 이런 경우가 정상적이라고 생각하게 된다. 한마디로 그렇다고 이해하기는 어려울 수 있지만, 특정 변수에 값을 저장하거나 읽어내는 코드가 여러 스레드에서 앞서거니 뒤서거니 실행된다면 반드시 그렇지 않을 수도 있다. 일반적으로 말하자면 특정 변수의 값을 가져갈 때 다른 스레드가 작성한 값을 가져갈 수 있다는 보장도 없고, 심지어는 값을 읽지 못할 수도 있다. 메모리상의 공유된 변수를 여러 스레드에서 서로 사용할 수 있게 하려면 반드시 동기화 기능을 구현해야 한다.

```java
public class NoVisibility { 
     private static boolean ready; 
     private static int number; 
     private static class ReaderThread extends Thread { 
          public void run() { 
               while(!ready) 
                    Thread.yield(); 
               System.out.println(number); 
          } 
     } 
     public static void main(String[] args) { 
          new ReaderThread().start(); 
          number = 42; 
          ready = true; 
     } 
}
```

위의 예제를 보면 동기화 작업이 되어 있지 않은 상태에서 여러 스레드가 동일한 변수를 사용할 때 어떤 문제가 생길 수 있는지를 알 수 있다. 메인 스레드와 읽기 스레드가 ready와 number라는 변수를 공유해 사용한다. 메인 스레드는 읽기 스레드를 실행시킨 다음 number라는 변수에 42라는 값을 넣고, ready 변수의 값을 true로 지정한다. 읽기 스레드는 ready 변수의 값이 true가 될 때까지 반복문에서 기다리다가 ready 값이 true로 변경되면 number 변수의 값을 출력한다. 일반적으로는 읽기 스레드가 42라는 값을 출력하리라고 생각할 수 있겠지만, 0이라는 값을 출력할 수도 있고, 심지어는 영원히 값을 출력하지 못하고 ready 변수의 값이 true로 바뀌기를 계속해서 기다릴 수도 있다. 말하자면 메인 스레드에서 number 변수와 ready 변수에 지정한 값을 읽기 스레드에서 사용할 수 없는 상황인데, 두 개 스레드에서 변수를 공유해 사용함에도 불구하고 적절한 동기화 기법을 사용하지 않았기 때문에 이런 일이 발생한다.

NoVisibility 클래스의 소스코드를 보면, ready 변수의 값을 읽기 스레드에서 영영 읽지 못할 수도 있기 때문에 무한 반복에 빠질 수 있다. 더 이상하게는 읽기 스레드가 메인 스레드에서 number 변수에 지정한 값보다 ready 변수의 값을 먼저 읽어가는 상황도 가능하다. 흔히 재배치\(reordering\)라고 하는 현상이다. 재배치 현상은 특정 메서드의 소스코드가 100% 코딩된 순서로 동작한다는 점을 보장할 수 없다는 점에 기인하는 문제이며, 단일 스레드로 동작할 때는 차이점을 전혀 알아챌 수 없지만 여러 스레드가 동시에 동작하는 경우에는 확연하게 나타날 수 있다\(자바 메모리 모델에서는 별다른 동기화 구조가 잡혀 있지 않은 경우에 컴파일러가 직접 코드 실행 순서를 조절하면서 하드웨어 레지스터에 데이터를 캐시하거나 CPU가 명령 실행 순서를 재배치하고 프로세서 내부의 캐시에 데이터를 보관하는 등의 작업을 할 수 있도록 되어 있다\).

NoVisibility 예제에서는 제대로 동기화되지 않은 프로그램을 실행했을 때 나타날 수 있는 여러 종류의 어이없는 상황 가운데 stale 데이터라는 결과를 체험했다. 읽기 스레드가 ready 변수의 값을 읽으려 할 때, 이미 최신 값이 아니었기 때문이다. 변수를 사용하는 모든 경우에 동기화를 시켜두지 않으면 해당 변수에 대한 최신 값이 아닌 다른 값을 사용하게 되는 경우가 발생할 수 있다. 게다가 더 큰 문제는 항상 stale 데이터를 사용하게 될 때도 있고, 정상적으로 동작하는 경우도 있다는 점이다. 다시 말하자면 특정 스레드가 어떤 변수를 사용할 대 정상적인 최신 값을 사용할 ‘수’도 있고, 올바르지 않은 값을 사용할 ‘수’도 있다는 말이다.

동기화되지 않은 상태에서 특정 스레드가 변수의 값을 읽으려 한다면 stale 상태의 값을 읽어갈 가능성이 있긴 하지만, 그래도 전혀 엉뚱한 값을 가져가는 것이 아니라 바로 이전에 다른 스레드에서 설정한 값을 가져가게 된다. 하지만 64비트를 사용하는 숫자형\(long, double 등\)에 volatile 키워드를 사용하지 않은 경우에는 난데없는 값마저 생길 가능성이 있다. 자바 메모리 모델은 메모리에서 값을 가져오고 저장하는 연산이 단일해야 한다고 정의하고 있지만, volatile로 지정되지 않은 long이나 double 형의 64비트 값에 대해서는 메모리에 쓰거나 읽을 때 두 번의 32비트 연산을 사용할 수 있도록 허용하고 있다. 따라서 volatile을 지정하지 않은 long 변수의 값을 쓰는 기능과 읽는 기능이 서로 다른 스레드에서 동작한다면, 이전 값과 최신 값에서 각각 32비트를 읽어올 가능성이 생긴다.

cf\) 락은 상호 배제뿐만 아니라 정상적인 메모리 가시성을 확보하기 위해서도 사용한다. 변경 가능하면서 여러 스레드가 공유해 사용하는 변수를 각 스레드에서 각자 최신의 정상적인 값으로 활용하려면 동일한 락을 사용해 모두 동기화시켜야 한다

자바 언어에서는 volatile 변수로 약간 다른 형태의 좀더 약한 동기화 기능을 제공하는데, 다시 말해 volatile로 선언된 변수의 값을 바꿨을 때 다른 스레드에서 항상 최신 값을 읽어 갈 수 있도록 해준다. 특정 변수를 선언 할 때 volatile 키워드를 지정하면 컴파일러와 런타임 모두 ‘이 변수는 공유해 사용하고, 따라서 실행 순서를 재배치 해서는 안 된다’고 이해한다. volatile로 지정된 변수는 프로세서의 레지스터에 캐시되지도 않고, 프로세서 외부의 캐시에도 들어가지 않기 때문에 volatile 변수의 값을 읽으면 항상 다른 스레드가 보관해둔 최신의 값을 읽어갈 수 있다.

실제로 volatile 변수가 갖는 가시성 효과는 volatile로 지정된 변수 자체의 값에 대한 범위보다 약간 확장되어 있다. 스레드 A가 volatile 변수에 값을 써넣고 스레드 B가 해당 변수의 값을 읽어 사용한다고 할 때, 스레드 B가 volatile 변수의 값을 읽고 나면 스레드 A가 변수에 값을 쓰기 전에 볼 수 있었던 모든 변수의 값을 스레드 B도 모두 볼 수 있다는 점이다. 따라서 메모리 가시성의 입장에서 본다면 volatile 변수를 사용하는 것과 synchronized 키워드로 특정 코드를 묶는 게 비슷한 효과를 가져오고, volatile 변수의 값을 읽고 나면 synchronized 블록에 진입하는 것과 비슷한 상태에 해당한다.

2.변경 가능 공유 데이터에 대한 접근은 동기화하라.\(이펙티브 자바 2ED 규칙66\)

synchronized 키워드는 특정 메서드나 코드 블록을 한 번에 한 스레드만 사용하도록 보장한다. 많은 프로그래머는 동기화\(synchronization\)를 상호 배제적\(mutually exclusive\)인 관점, 그러니까 다른 스레드가 변경 중인 객체의 상태를 관측할 수 없어야 한다는 관점으로만 바라본다. 맞는 말이나, 딱 절반만 이야기 했을 뿐이다. 동기화 없이는 한 스레드가 만든 변화를 다른 스레드가 확인 할 수 없다. 동기화는 스레드가 일관성이 깨진 객체를 관측할 수 없도록 할 뿐 아니라, 동기화 메서드나 동기화 블록에 진입한 스레드가 동일한 락의 보호 아래 이루어진 모든 변경의 영향을 관측할 수 있도록 보장한다.

성능을 높이기 위해서는 원자적 데이터를 읽거나 쓸 때 동기화를 피해야 한다는 이야기를 아마 들어보았을 것이다. 아주 위험한 이야기다. 언어 명세상으로는 필드에서 읽어낸 값은 임의의 값이 될 수 없다고 되어 있으나, 그렇다고 어떤 스레드가 기록한 값을 반드시 다른 스레드가 보게 되리라는 보장은 없다. 상호 배제성뿐 아니라 스레드 간의 안정적 통신을 위해서라도 동기화는 반드시 필요하다. 자바 언어 명세의 일부인 메모리 모델 때문이다. 메모리 모델은 한 스레드가 만든 변화를 다른 스레드가 볼 수 있게 되는 시점과 그 절차를 규정한다.

```java
//잘못된 코드  
public class StopThread { 
     private static boolean stopRequested; 
     public static void main(String[] args) throws InterruptedExcetpion{ 
          Thread backgroundThread = new Thread(new Runnable() { 
               public void run() { 
                    int i = 0; 
                    while (!stopRequested) 
                         i++; 
               } 
          }); 
          backgroundThread.start(); 
          TimeUnit.SECONDS.sleep(1); 
          stopRequested = true; 
     } 
}
```

실행한지 1초가 지나면 main 스레드가 stopRequested의 값을 true로 바꾸므로 backgroundThread가 실행하는 순환문도 그때 중지될 것 같다. 하지만 이 프로그램의 문제는 동기화 메커니즘을 적용하지 않은 탓에 main 스레드가 변경한 stopRequested의 새로운 값을 backgroundThread가 언제쯤 보게 될지 알수가 없다는 것이다. 이 문제를 수정하는 한 가지 방법은 stopRequested 필드를 동기화하는 것이다.

```java
public class StopThread { 
     private static boolean stopRequested; 
     private static synchronized void requestStop() { 
          stopRequested = true; 
     } 
     private static synchronized boolean stopRequested() { 
          return stopRequested; 
     } 
     public static void main(String[] args) throws InterruptedExcetpion{ 
          Thread backgroundThread = new Thread(new Runnable() { 
               public void run() { 
                    int i = 0; 
                    while (!stopRequested()) 
                         i++; 
               } 
          }); 
          backgroundThread.start(); 
          TimeUnit.SECONDS.sleep(1); 
          requestStop(); 
     } }
```

쓰기 메서드와 읽기 메서드에 동기화 메커니즘이 적용되었음에 유의하자. 쓰기 메서드만 동기화하는 것으로는 충분치 않다. 사실, 읽기 연산과 쓰기 연산에 전부 적용하지 않으면 동기화는 아무런 효과도 없다.

StopThread의 동기화 메서드가 하는 일은 동기화가 없이도 원자적이다. 다시 말해서, 이들 메서드에 동기화를 적용한 것은 상호 배제성을 달성하기 위해서가 아니라, 순전히 스레드 간 통신 문제를 해결하기 위해서였다. 비록 순환문의 각 단계마다 동기화를 실행하는 비용이 크진 않지만, 그 비용을 줄여서 좋은 성능을 내면서도 간결하기까지 한 대안이 있다. 위 코드에 사용된 boolean 필드 stopRequested를 volatile로 선언하는 것이다. 그러면 락은 없어도 된다. 비록 volatile이 상호 배제성을 실현하진 않지만, 어떤 스레드건 가장 최근에 기록된 값을 읽도록 보장한다.

```java
public class StopThread { 
     private static volatile boolean stopRequested; 
     public static void main(String[] args) throws InterruptedExcetpion{ 
          Thread backgroundThread = new Thread(new Runnable() { 
               public void run() { 
                    int i = 0; 
                    while (!stopRequested) 
                         i++; 
               } 
          }); 
          backgroundThread.start(); 
          TimeUnit.SECONDS.sleep(1); 
          stopRequested = true; 
     } 
}
```

그런데 volatile을 사용할 때는 주의해야 한다. 아래의 메서드를 보자. 일련번호를 만들어 내는 메서드다.

```java
//잘못된 예제 - 동기화가 필요하다 
private static volatile int nextSerialNumber = 0; 
public static int generateSerialNumber() { 
     return nextSerialNumber++; 
}
```

이 메서드의 원래 의도는, 호출 될 때마다 다른 값을 반환하는 것이었다. 그런데 동기화 없이는 제대로 동작하지 않는다. 문제는 증가 연산자 ++가 원자적이지 않다는 데 있다. 이 연산자는 nextSerialNumber 필드에 두 가지 연산을 순서대로 시행한다. 먼저 값을 읽고, 그 다음에 새로운 값, 즉 읽은 값 더하기 1을 필드에 쓴다. 첫 번째 스레드가 필드의 값을 읽은 후 새 값을 미처 기록하기 전에 두 번째 스레드가 필드에서 같은 값을 읽으면, 두 스레드는 같은 일련번호를 얻게 된다. 이것은 안전 오류다.

이 문제를 해결하는 한 가지 방법은 메서드를 synchronized로 선언하는 것이다. 그러면 여러 스레드가 동시에 호출하더라도 서로 겹쳐 실행되지 않는 메서드가 되고, 각각의 메서드 호출은 그전에 행해진 모든 호출의 영향을 관측할 수 있게 된다. synchronized 키워드를 붙였다면, nextSerialNumber에 붙였던 volatile 키워드는 삭제해야 한다. 더 좋은 방법은 AtomicLong 클래스를 쓰는 것이다. 이 클래스는 java.util.concurrent.atomic의 일부다. 원하는 일을 해주면서도, synchronized 키워드를 사용한 해법보다 성능도 좋다.

```java
private static final AtomicLong nextSerialNum = new AtomicLong(); 

public static long generateSerialNumber() { 
     return nextSerialNum.getAndIncrement(); 
}
```

## Thread-safe 하다는 것은?

자바 병렬 프로그래밍 p47

스레드에 안전한 코드를 작성하는 것은 근본적으로는 상태, 특히 공유되고 변경할 수 있는 상태에 대한 접근을 관리하는 것이다. 공유 됐다는 것은 여러 스레드가 특정 변수에 접근할 수 있다는 뜻이고, 변경할 수 있다\(mutable\)는 것은 해당 변수 값이 변경될 수 있다는 뜻이다. 스레드 안전성이 마치 코드를 보호하는 것처럼 이해하는 경우가 많지만, 실제로는 데이터에 제어 없이 동시 접근하는 걸 막으려는 의미 임을 알아두자.

스레드 안전성은 코드에 적용되는 용어일 수도 있지만 주로 상태에 대한 것이고, 상태를 캡슐화하는 코드 전체에 적용될 수 있으며, 적용되는 코드는 객체이거나 또는 프로그램 전체일 수도 있다.

스레드에 대한 납득할 만한 정의의 핵심은 모두 정확성\(correctness\) 개념과 관계 있다. 스레드 안전성에 대한 정의가 모호한 것은 정확성에 대한 명확한 정의가 없기 때문이다. 일단 “특정 코드가 동작한다”고 확신하기만 하면, 이런 코드 신뢰도는 많은 사람이 생각하는 정확성과 대략 일치한다. 이런 맥락에서 단일 스레드에서의 정확성은 ‘척 보면 아는’ 어떤 것이라고 가정하자. 낙관적으로 ‘정확성’을 인지할 수 있는 어떤 것으로 정의하면 스레드 안전성도 다소 덜 순환적으로 정의할 수 있다. **즉 여러 스레드가 클래스에 접근할 때 계속 정확하게 동작하면 해당 클래스는 스레드 안전하다.** 모든 단일 스레드 프로그램은 멀티스레드 프로그램의 한 종류라고 볼 수 있기 때문에, 애당초 단일 스레드 환경에서도 제대로 동작하지 않으면 스레드 안전할 수 없다.

스레드 안전한 클래스는 클라이언트 쪽에서 별도로 동기화할 필요가 없도록 동기화 기능도 캡슐화한다.  
상태 없는 객체는 항상 스레드 안전하다. 상태를 일관성 있게 유지하려면 관련 있는 변수들을 하나의 단일 연산으로 갱신해야 한다. \(락을 사용해서\)

## 과도한 동기화가 안좋은점은?

\(규칙67\) 상황에 따라서는 동기화를 너무 과도하게 적용하면 성능저하, 교착상태\(deadlock\), 비결정적 동작 등의 문제가 생길 수 있다. 생존 오류나 안전 오류를 피하고 싶으면, **동기화 메서드나 블록 안에서 클라이언트에게 프로그램 제어 흐름을 넘기지 마라. 다시 말해서, 동기화가 적용된 영역 안에서는 재정의\(override\) 가능 메서드나 클라이언트가 제공한 함수 객체 메서드를 호출하지 말라는 것이다. 동기화 영역이 존재하는 클래스 관점에서 보면, 그런 메서드는 불가해\(alien\) 메서드다.** 무슨 일을 하는지 알 수도 없고, 제어할 수도 없다. 불가해 메서드가 어떤 일을 하느냐에 따라, 동기화 영역 안에서 호출하게 되면 예외나 교착상태, 데이터 훼손이 발생할 수 있다.

다음은 옵저버 패턴 코드다. 각각의 add, remove, notify 메서드에 모두 동기화가 적용되어 있다.

```java
    private final List<SetObserver<E>> observers = new ArrayList<>(); 

    public void addObserver(SetObserver<E> observer){ 
        synchronized (observers){ 
            observers.add(observer); 
        } 
    } 

    public boolean removeObserver(SetObserver<E> observer){ 
        synchronized (observers){ 
            return observers.remove(observer); 
        } 
    } 

    private void notifyElementAdded(E element){ 
        synchronized (observers){ 
            for(SetObserver<E> observer : observers) 
                    observer.added(this,element); 
        } 
    }
```

이 코드를 돌리면 0에서 23까지 찍히고, 23에 도달하면 구독자는 자기 자신을 구독 해제한 후 프로그램은 나머지 작업을 조용히 계속할 거라 생각한다.

```java
        ObservableSet<Integer>set = new ObservableSet<>(new HashSet<Integer>()); 

        //옵저버 익명함수 객체 
        set.addObserver(new SetObserver<Integer>() { 
            @Override 
            public void added(ObservableSet<Integer> set, Integer element) { 
                System.out.println(element); 
                if(element == 23) { 
                    set.removeObserver(this); 
                } 
            } 
        }); 

        for(int i=0; i< 25;i++)
            set.add(i);
```

하지만 실제로는 ConcurrentModificationException이 발생한다. 문제는 add 메서드를 통해 리스트 순회가 이루어지고 있는 도중에 리스트에서 원소를 삭제하려 한 것이다. 불법이다. notifyElementAdded 메서드의 순환문은 동기화 블록 안에 있다. observers 리스트가 병렬적으로 수정되는 일을 막기 위해서였다. 하지만 이렇게 했어도 순환문을 실행하는 스레드 자신이 구독자 집합에 저장된 메서드를 역호출\(callback\)해서 observers 리스트를 변경하는 경우까지 차단할 순 없었다.

좀 더 이상한 짓을 해보자. 구독 해제를 시도하는 구독자를 만들되, removeObserver를 직접 호출하는 대신 그 일을 해줄 다른 스레드의 서비스를 이용하는 것이다. 이 구독자는 실행자 서비스\(executor service\)를 사용한다.

```java
// 괜히 후면 스레드를 이용하는 구독자 
ObservableSet<Integer>set2 = 
                new ObservableSet<>(new HashSet<Integer>());
set2.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if(element == 23){
                    ExecutorService executor =
                            Executors.newSingleThreadExecutor();
                    final SetObserver<Integer> observer = this;
                    try{
                        executor.submit(new Runnable() {
                            @Override
                            public void run() {
                                set.removeObserver(observer);
                            }
                        }).get();
                    }catch (ExecutionException ex){
                        throw new AssertionError(ex.getCause());
                    }catch (InterruptedException ex){
                        throw new AssertionError(ex);
                    }finally {
                        executor.shutdown();
                    }
                }
            } 
        });
```

이번에는 예외가 발생하진 않는 반면 교착상태가 생긴다. 후면 스레드는 set.removeObserver를 호출하는데, 이 메서드는 observers에 락을 걸려 한다. 하지만 락을 걸 수는 없다. 왜냐하면 주 스레드가 이미 락을 걸고 있기 때문이다. 주 스레드는 후면 스레드가 구독 해제를 끝내기를 기다리면서 락을 계속 들고 있는데, 그래서 교착상태가 생기는 것이다.

**동기화 영역 안에서 불가해 메서드를 호출하는것은 많은 실제 시스템에서 교착상태가 생기는 원인이다.** 그래도 앞서 살펴본 두 예제\(예외와 데드락\)는 운이 좋은 편이다. 불가해 메서드 added가 호출될 시점에, 동기화 영역이 보호하는 자원 observers의 상태는 일관되게 유지되고 있었으니까.

다행히 이런 문제는 불가해 메서드를 호출하는 부분을 동기화 영역 밖으로 옮기면 쉽게 해결 할 수 있다.

```java
//불가해 메서드를 호출하는 코드를 동기화 영역 밖으로 옮겼다
private void notifyElementAdded(E element){
        List<SetObserver<E>> snapshot = null;
        synchronized (observers){
            snapshot = new ArrayList<SetObserver<E>>(observers);
        }
        for(SetObserver<E> observer : snapshot) {
            System.out.println("this :"+this+" element:"+element);
            observer.added(this, element);
        } 
}
```

notifyElementAdded 메서드의 경우, observers 리스트의 복사본을 만들어서 락 없이도 안전하게 리스트를 순회할 수 있도록 바꾸는 것이다.

사실 불가해 메서드 호출 코드를 동기화 영역 밖으로 옮기는 문제라면 더 좋은 해결책이 있다. 릴리스 1.5부터 자바 라이브러리에는 CopyOnWriteArrayList라는 병행성 컬렉션이 추가되었다. 이 리스트는 ArrayList의 변종으로 내부 배열을 통째로 복사하는 방식으로 쓰기 연산을 지원한다. 내부 배열을 절대 수정하지 않으므로 순회 연산만큼은 락을 걸 필요가 없어져서 대단히 빠르다. 이 리스트의 성능은 대체로 끔찍한 수준이지만 구독자\(observer\) 리스트에는 딱이다. 구독자 리스트는 변경할 일이 거의 없는 데다 순회 연산이 압도적으로 많기 때문이다.

```java
// 다중 스레드에 안전한 observer 집합
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer){
            observers.add(observer);
    }
    public boolean removeObserver(SetObserver<E> observer){
            return observers.remove(observer);
    }
    private void notifyElementAdded(E element){
            for(SetObserver<E> observer : observers)
                observer.added(this,element);
    }
```

명심해야 할 것은 동기화 영역 안에서 수행되는 작업의 양을 가능한 한 줄여야 한다는 것이다.

이제 성능에 관한 내용을 살펴보자. 변경 가능\(mutable\) 클래스의 경우, 병렬적으로 이용될 클래스이거나, 내부적인 동기화를 통해 외부에서 전체 객체에 락을 걸 때보다 높은 병행성을 달성할 수 있을 때만 스레드 안전성을 갖도록 구현해야 한다. 그렇지 않다면 내부적인 동기화는 하지 마라. 예를 들어 StringBuffer 객체는 거의 항상 한 스레드만 이용하는 객체인데도 내부적으로 동기화를 하도록 구현되어 있다. 그래서 결국 StringBuilder로 대체된 것이다.

static 필드를 변경하는 메서드가 있을 때는 해당 필드에 대한 접근을 반드시 동기화해야 한다. 보통 한 스레드만 이용하는 메서드라 해도 그렇다. 클라이언트 입장에서는 그런 메서드에 대한 접근을 외부적으로 동기화할 방법이 없다. 다른 클라이언트가 무슨 짓을 할지 알 수 없기 때문이다.

## Lock

자바 병렬 프로그래밍 p59

재진입성 : 스레드가, 다른 스레드가 가진 락을 요청하면 해당 스레드는 대기 상태에 들어간다. 하지만 암묵적인 락은 재진입 가능하기 때문에 특정 스레드가 자기가 이미 획득한 락을 다시 확보할 수 있다. 재진입성은 확보 요청 단위가 아닌 스레드 단위로 락을 얻는다는 것을 의미한다\(이는 pthreads의 기본 락 동작과 다르다 pthreads에선 확보 요청 기준으로 락을 허용한다\).

재진입성 때문에 락의 동작을 쉽게 캡슐화할 수 있고, 객체 지향 병렬 프로그램을 개발하기가 단순해졌다. 재진입 가능한 락이 없으면 하위 클래스에서 synchronized 메서드를 재정의하고 상위 클래스의 메서드를 호출하는 예제와 같은 지극히 자연스러워 보이는 코드도 데드락에 빠질 것이다.

```java
public class Widget { 
     public synchronized void doSomething() { 
          ... 
     } 
} 

public class LoggingWidget extends Widget { 
     public synchronized void doSomething() { 
          super.doSomething(); 
     }    
}
```

Widget과 LoggingWidget의 doSomething 메서드는 둘 다 synchronized로 선언돼 있고, 각각 진행 전에 Widget에 대한 락을 얻으려고 시도한다. 하지만 암묵적인 락이 재진입 가능하지 않았다면, 이미 락을 누군가 확보했기 때문에 super.doSomething 호출에서 락을 얻을 수 없게 되고, 결과적으로 확보할 수 없는 락을 기다리면서 영원히 멈춰 있었을 것이다. 재진입성은 이런 경우에 데드락에 빠지지 않게 해준다.

