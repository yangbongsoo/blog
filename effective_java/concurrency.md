# 병행성
### 규칙78 : 변경 가능 공유 데이터에 대한 접근은 동기화하라
많은 프로그래머는 동기화(synchronization)를 상호 배제적인 관점, 그러니까 다른 스레드가 변경 중인 객체의 상태를 관측할 수 없어야 한다는 관점으로만 바라본다.

이 관점에 따르면 객체는 일관된 상태를 갖도록 생성되며, 해당 객체를 접근하는 메서드는 그 객체에 락을 건다. 락을 건 메서드는 객체의 상태를 관측할 수 있으며, 선택적으로 객체 상태를 변경할 수도 있다. 하나의 일관된 상태에서 다른 일관된 상태로 전이시킬 수 있다는 것이다. 동기화 메커니즘을 적절히 사용하기만 하면, 모든 메서드가 항상 객체의 일관된 상태만 보도록 만들 수 있다.

맞는 말이나 딱 절반만 이야기 했을 뿐이다. **동기화 없이는 한 스레드가 만든 변화를 다른 스레드가 확인할 수 없다. 동기화는 스레드가 일관성이 깨진 객체를 관측할 수 없도록 할 뿐 아니라, 동기화 메서드나 동기화 블록에 진입한 스레드가 동일한 락의 보호 아래 이루어진 모든 변경의 영향을 관측할 수 있도록 보장한다.**

자바 언어 명세에는 long이나 double이 아닌 모든 변수는 원자적으로 읽고 쓸 수 있다고 되어 있다. 다시 말해, long이나 double이 아닌 변수를 읽으면 나오는 값은 항상 어떤 스레드가 저장한 값이라는 것이다. 설사 열러 스레드가 그 변수를 동기화 없이 변경했다고 해도 말이다.

"성능을 높이기 위해 원자적 데이터를 읽거나 쓸 때 동기화를 피해야 한다"는 아주 위험한 이야기다. 언어 명세상으로는 필드에서 읽어낸 값은 임의의 값이 될 수 없다고 되어 있으나, 그렇다고 어떤 스레드가 기록한 값을 반드시 다른 스레드가 보게 되리라는 보장은 없다. **상호 배제성뿐 아니라 스레드 간의 안정적 통신을 위해서도 동기화는 반드시 필요하다.** 자바 언어 명세의 일부인 메모리 모델 때문이다. 메모리 모델은 한 스레드가 만든 변화를 다른 스레드가 볼 수 있게 되는 시점과, 그 절차를 규정한다.

```java
//잘못된 코드 - 이 프로그램은 얼마나 오랫동안 실행될까?
public class StopThread {
  private static boolean stopRequested;
  
  public static void main(String[] args) throws InterruptedException {
    
    Thread backgroundThread = new Thread(new Runnable(){
        
        public void run(){
          int i = 0;
          while(!stopRequested)
            i++;
        }
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```
실행한지 1초가 지나면 main 스레드가 stopRequested의 값을 true로 바꾸므로, background thread가 실행하는 순환문도 그때 중지될 것 같지만 이 프로그램은 절대로 멈추지 않는다. **문제는 동기화 메커니즘을 적용하지 않은 탓에 main 스레드가 변경한 stopRequest의 새로운 값을 background thread가 언제쯤 보게 될지 알 수가 없다는 것이다.**
```java
while(!stopRequested)
  i++;
  
// 동기화가 적용되지 않은 경우, 가상 머신은 위의 코드를 아래와 같이 바꿀 수 있다.
if(!stopRequested)
  while(true)
    i++;
```
이런 최적화를 끌어올리기(hoisting)라고 하는데, HotSpot 서버 VM이 하는 일이 바로 이런 것이다. 그 덕에 생기는 문제가 바로 생존 오류다. 살아 있기는 하나 더 진행하지는 못하는 프로그램이 되는 것이다. 이 문제를 수정하는 한 가지 방법은 stopRequested 필드를 동기화하는 것이다. 
```java
// 적절히 동기화한 스레드 종료 예제
public class StopThread {
  private static boolean stopRequested;
  
  private static synchronized void requestStop(){
    stopRequested = true;
  }
  
  private static synchronized boolean stopRequested(){
    return stopRequested;
  }
  
  public static void main(String[] args) throws InterruptedException { 
    Thread backgroundThread = new Thread(new Runnable(){
          public void run(){
            int i = 0;
            while(!stopRequested())
              i++
          }
      });
      backgroundThread.start();

      TimeUnit.SECONDS.sleep(1);
      requestStop();
  }
}
```
쓰기 메서드와 읽기 메서드에 동기화 메커니즘이 적용되었음을 유의하자. **읽기 연산과 쓰기 연산에 전부 동기화를 적용하지 않으면 동기화는 아무런 효과도 없다.** 그런데 StopThread의 동기화 메서드가 하는 일은 동기화가 없이도 원자적이다. 다시 말해 이들 메서드에 동기화를 적용한 것은 상호 배제성을 달성하기 위해서가 아니라, 순전히 스레드 간 통신 문제를 해결하기 위해서였다는 것이다. 

비록 순환문의 각 단계마다 동기화를 실행하는 비용이 크진 않지만, 그 비용을 줄여서 좋은 성능을 내면서도 간결하기까지 한 대안이 있다. 위 코드에 사용된 boolean 필드 stopRequested를 volatile로 선언하는 것이다. 그러면 락은 없어도 된다. 비록 volatile이 상호 배제성을 실현하진 않지만, 어떤 스레드건 가장 최근에 기록된 값을 읽도록 보장한다.
```java
// volatile 필드를 사용해 스레드를 종료시키는 예제
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException{
        Thread backgroundThread = new Thread(new Runnable(){

            public void run(){
                int i = 0;
                while(!stopRequested)
                    i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;

    }
}
```
volatile을 사용할 때는 주의해야 한다. 아래의 메서드를 보자. 일련번호를 만들어 내는 메서드다.
```java
// 잘못된 예제 - 동기화가 필요하다!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber(){
  return nextSerialNumber++;
}
```
이 메서드의 원래 의도는, 호출 될 때마다 다른 값을 반환하는 것이었다. 이 메서드의 상태를 구성하는 것은 원자적으로 접근 가능한 필드 nextSerialNumber이며 이 필드가 가질 수 있는 값은 전부 유효하다. 따라서 불변식을 보호하기 위해 동기화 메커니즘을 사용할 필요가 없다. 그런데도 동기화 없이는 제대로 동작하지 않는다.

문제는 증가 연산자 ++가 원자적이지 않다는 데 있다. 이 연산자는 nextSerialNumber 필드에 두 가지 연산을 순서대로 시행한다. 먼저 값을 읽고, 그 다음에 새로운 값, 즉 읽은 값 더하기 1을 필드에 쓴다. 첫 번째 스레드가 필드의 값을 읽은 후 새 값을 미처 기록하기 전에 두 번째 스레드가 필드에서 같은 값을 읽으면, 두 스레드는 같은 일련번호를 얻게 된다. 이것은 안전 오류다. 프로그램이 잘못된 결과를 계산하는 것이다.

이 문제를 해결하는 한 가지 방법은, 메서드를 synchronized로 선언하는 것이다. 그러면 여러 스레드가 동시에 호출하더라도 서로 겹쳐 실행되지 않는 메서드가 되고, 각각의 메서드 호출은 그전에 행해진 모든 호출의 영향을 관측할 수 있게 된다. synchronized 키워드를 붙였다면, volatile 키워드는 삭제해야 한다. **더 견고한 메서드로 만들려면 int 대신 long을 쓰거나, 아니면 nextSerialNumber가 반환할 값이 다시 0으로 돌아갈 때 예외가 뜨도록 만들라.**

**더 좋은 방법은 AtomicLong 클래스를 쓰는 것이다.** 이 클래스는 java.util.concurrent.atomic의 일부다. 원하는 일은 해주면서도, synchronized 키워드를 사용한 해법보다 성능도 좋다. 
```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber(){
  return nextSerialNum.getAndIncrement();
}
```

이번 절에서 설명한 문제를 피하는 가장 좋은 방법은 변경 가능 데이터를 공유하지 않는 것이다. 굳이 공유를 해야겠다면 변경 불가능 데이터를 공유하거나 그럴 필요가 없다면 아예 공유하지 마라. 다시 말해 **변경 가능 데이터는 한 스레드만 이용하도록 하라는 것이다.**

### 규칙79 : 과도한 동기화는 피하라
상황에 따라서는 동기화를 너무 과도하게 적용하면 성능 저하, 교착 상태(deadlock), 예측할 수 없는 동작 등의 문제가 생길 수 있다.
**liveness(생존 오류, 응답 불가)나 safety failures(안전 오류, 안전 실패)를 피하고 싶으면, 동기화 메서드나 블록 안에서 클라이언트에게 프로그램 제어 흐름을 넘기지 마라.**
다시 말해 동기화가 적용된 영역 안에서는 재정의 가능 메서드나 클라이언트가 제공한 함수 객체 메서드를 호출하지 말라는 것이다.
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
    	synchronized (observers) {
        	for (SetObserver<E> observer : observers)
            		observer.added(this,element);
    	}
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // notifyElementAdded를 호출한다. 
        return result;
    }
}
```
관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두 경우 모두 다음 
콜백 인터페이스의 인스턴스를 메서드에 건넨다.
```java
@FunctionalInterface  public interface SetObserver<E> {
    //ObservableSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```
```java
public class ObservableSetTest {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver((s,e)-> System.out.println(e));

        for (int i=0; i<100; i++) {
            set.add(i);
        }
    }
}
```
위 프로그램은 0부터 99까지를 정상적으로 출력한다.

이제 조금 흥미진진한 시도를 해보자. 평상시에는 앞서와 같이 집합에 추가된 정숫값을 출력하다가, 그 값이 23이면 자기 자신을 제거하는
관찰자를 추가해보자.
```java
public class ObservableSetTest {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    s.removeObserver(this);
                }
            }
        });

        for (int i=0; i<100; i++) {
            set.add(i);
        }
    }
}
```
람다를 사용한 이전 코드와 달리 익명 클래스를 사용했다. `s.removeObserver` 메서드에 함수 객체 자신을 넘겨야 하기 때문이다.
람다는 자신을 참조할 수단이 없다.

이 프로그램은 0부터 23까지 출력한 후 관찰자 자신을 구독해지한 다음 조용히 종료할 거 같지만 실제로 실행시켜 보면
23까지 출력한 다음 ConcurrentModificationException을 던진다. 관찰자의 add 메서드 호출이 일어난 시점이
notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다.

즉 다시말하면 아래 코드처럼 notifyElementAdded 메서드에서 관찰자를 순회하는데, added 메서드에서 remove를 시켜버린다.
허용되지 않는 동작이다.
```java
    private void notifyElementAdded(E element) {
    	synchronized (observers) {
        	for (SetObserver<E> observer : observers)
            		observer.added(this,element);
    	}
    }
    
    set.addObserver(new SetObserver<Integer>() {
                @Override
                public void added(ObservableSet<Integer> s, Integer e) {
                    System.out.println(e);
                    if (e == 23) {
                        s.removeObserver(this);
                    }
                }
            });
```
notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 보장하지만,
정작 자신이 콜백을 거쳐 되돌아와 수정 하는 것까지 막지는 못한다.

이번에는 이상한 것을 시도 해보자. 구독해지를 하는 관찰자를 작성하는데, removeObserver를 직접 호출하지 않고 실행자 서비스(ExecutorService)를
사용해 다른 스레드한테 부탁할 것이다.
```java
// 쓸데없이 백그라운드 스레드를 사용하는 관찰자
public class ObservableSetTest {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    ExecutorService exec = Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i=0; i<100; i++) {
            set.add(i);
        }
    }
}
```
이 프로그램은 예외가 발생하지 않지만 교착상태가 생긴다. 백그라운드 스레드는 `s.removeObserver`를 호출하는데,
이 메서드는 observers에게 락을 걸려한다. 하지만 락을 걸 수 없다.
왜냐하면 메인 스레드가 이미 락을 잡고 있기 때문이다. 
메인 스레드는 백그라운드 스레드가 구독 해제를 끝내기를 기다리면서 락을 계속 들고 있다. 바로 교착상태다.

다행히도 이런 문제는 외계인 메서드(재정의 메서드나 클라이언트가 제공한 함수 객체 메서드)를 호출하는 부분을 동기화 영역 밖으로 옮기면 쉽게 해결할 수 있다.
notifyElementAdded 메서드의 경우, observers리스트의 복사본을 만들어서 락 없이도 안전하게 리스트를 순회할 수 있도록 바꾸는 것이다.
이렇게 바꾸면 앞서 보았던 두 예제에서는 더 이상 예외나 교착상태가 일어나지 않는다.
```java
//불가해 메서드를 호출하는 코드를 동기화 영역 밖으로 옮겼다
private void notifyElementAdded(E element) {
        List<SetObserver<E>> snapshot = null;
        synchronized (observers) {
            snapshot = new ArrayList<SetObserver<E>>(observers);
        }

        for (SetObserver<E> observer : snapshot) {
            observer.added(this, element);
        }
}
```
사실 외계인 메서드 호출 코드를 동기화 영역 밖으로 옮기는 문제라면 더 좋은 해결책이 있다. 릴리스 1.5부터 자바 라이브러리에는 CopyOnWriteArrayList라는 병행성 컬렉션이 추가되었다.
이 리스트는 ArrayList의 변종으로 내부 배열을 통째로 복사하는 방식으로 쓰기 연산을 지원한다. **내부 배열을 절대 수정하지 않으므로 순회 연산만큼은 락을 걸 필요가 없어져서 대단히 빠르다.
이 리스트의 성능은 대체로 끔찍한 수준이지만 관찰자(observer) 리스트에는 딱이다. 관찰자 리스트는 변경할 일이 거의 없는 데다 순회 연산이 압도적으로 많기 때문이다.**
```java
// 다중 스레드에 안전한 observer 집합
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer) {
            observers.add(observer);
    }

    public boolean removeObserver(SetObserver<E> observer) {
            return observers.remove(observer);
    }

    private void notifyElementAdded(E element) {
            for (SetObserver<E> observer : observers)
                observer.added(this,element);
    }
```
명심해야 할 것은 동기화 영역 안에서 수행되는 작업의 양을 가능한 한 줄여야 한다는 것이다.

이제 성능에 관한 내용을 살펴보자. 변경 가능 클래스의 경우, 병렬적으로 이용될 클래스이거나, 내부적인 동기화를 통해 외부에서 전체 객체에 락을 걸 때보다 높은 병행성을 달성할 수 있을 때만 스레드 안전성을 갖도록 구현해야 한다.
그렇지 않다면 내부적인 동기화는 하지 마라. 예를 들어 StringBuffer 객체는 거의 항상 한 스레드만 이용하는 객체인데도 내부적으로 동기화를 하도록 구현되어 있다. 그래서 결국 StringBuilder로 대체된 것이다.
비슷한 이유로, 스레드 안전한 `java.util.Random`은 동기화하지 않는 버전인 `java.util.concurrent.ThreadLocalRandom`으로 대체되었다. 선택하기 어렵다면 동기화하지 말고, 대신 문서에
"스레드 안전하지 않다"고 명기하자.

여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기해야 한다(비결정적 행동도 용인하는 클래스라면 상관없다).
그런데 클라이언트가 여러 스레드로 복제돼 구동되는 상황이라면 다른 클라이언트에서 이 메서드를 호출하는 걸 막을 수 없으니 외부에서 동기화할 방법이 없다. 결과적으로, 이 정적 필드가 심지어
private라도 서로 관련 없는 스레드들이 동시에 읽고 수정할 수 있게 된다. 사실상 전역 변수와 같아진다는 뜻이다.
```java
//잘못된 예제 - 동기화가 필요하다
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber() {
     return nextSerialNumber++;
}
```

### 규칙80 : 스레드보다는 실행자와 태스크, 스트림을 애용하라
릴리스 1.5부터 자바 플랫폼에는  java.util.concurrent가 추가되었다. 이 패키지에는 실행자 프레임워크(Executor Framework)라는 것이 들어 있는데, 유연성이 높은 인터페이스 기반 태스크 실행 프레임워크다. 
```
ExecutorService executor = Executors.newSingleThreadExecutor();
```
위의 한줄로 작업 큐를 생성할 수 있게 됐다.

```
executor.execute(runnable);
```
다음은 이 실행자에 실행할 태스크(작업)을 넘기는 방법이다.

```
executor.shutdown();
```
그리고 다음은 실행자를 우아하게 종료시키는 방법이다(이 작업이 실패하면 VM 자체가 종료되지 않을 것이다).

실행자 서비스의 기능은 이 외에도 많다.

1. 특정 태스크가 완료되기를 기다린다(규칙 79에서 본 get 메서드).
2. 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다린다.
3. 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드).
4. 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용).
5. 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용).

큐를 둘 이상의 스레드가 처리하게 하고 싶다면 간단히 다른 정적 팩토리를 이용하여 다른 종류의 실행자 서비스(스레드 풀)를 생성하면 된다.
스레드 풀에 담기는 스레드의 숫자는 고정시켜 놓을 수도 있고, 가변적으로 변하도록 설정할 수도 있다.
java.util.concurrent.Executors 클래스에는 필요한 실행자 대부분을 생성할 수 있도록 하는 정적 팩터리 메서드들이 들어 있다.
하지만 일반적이지 않은 무언가가 필요할 때는 ThreadPoolExecutor 클래스를 직접 이용할 수도 있다. 이 클래스를 이용하면 스레드 풀의 동작을 거의 모든 측면에서 세밀하게 제어할 수 있다.

작은 프로그램이나 가벼운 서버라면 `Executors.newCachedThreadPool`이 일반적으로 좋은 선택이다. 특별히 설정할 게 없고 일반적인 용도에 적합하게 동작한다.
하지만 CachedThreadPool은 무거운 프로덕션 서버에는 좋지 못하다. CachedThreadPool에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행된다. 가용한 스레드가 없다면 새로 하나를 생성한다.
서버가 아주 무겁다면 CPU 이용률이 100%로 치닫고, 새로운 태스크가 도착하는 족족 또 다른 스레드를 생성하며 상황을 더욱 악화시킨다.
따라서 부하가 심한 환경에 들어갈 서버를 만들 때는 스레드 개수를 고정한 `Executors.newFixedThreadPool`을 선택하거나 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 훨씬 낫다.

**작업 큐를 손수 구현하는 것은 삼가야 할 뿐 아니라, 스레드를 직접 다루는 것도 일반적으로 삼가야 한다.** 스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 역할을 모두 수행하게 된다.
반면 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리된다. 작업 단위를 나타내는 핵심 추상 개념이 태스크다. 태스크에는 두 가지가 있다.
Runnable과, 그 가까운 사촌격인 Callable이다(Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다는 차이가 있다).
그리고 태스크를 수행하는 일반적인 메커니즘이 바로 실행자 서비스(executor service)다.
태스크와 실행자 서비스를 분리해서 생각하게 되면 실행 정책을 더욱 유연하게 정할 수 있게 된다. **핵심은, 컬렉션 프레임워크가 데이터를 모으는 일을 처리하는 것과 마찬가지로, 실행자 프레임워크는 태스크를 실행하는 부분을 담당한다는 것이다.**

자바 7이 되면서 실행자 프레임워크는 포크-조인(fork-join) 태스크를 지원하도록 확장되었다. 포크-조인 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다. 포크-조인 태스크, 즉 ForkJoinTask의 인스턴스는
작은 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다. 이렇게 하여 모든 스레드가 바쁘게 움직여
CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다.

### 규칙81 : wait이나 notify 대신 병행성 유틸리티를 이용하라
릴리즈 1.5부터 자바 플랫폼에는 고수준 병행성 유틸리티들이 포함되어, 예전에는 wait과 notify를 사용해 구현해야만 했던 일들을 대신한다. **wait과 notify를 정확하게 사용하는 것이 어렵기 때문에, 이 고수준 유틸리티들을 반드시 이용해야 한다.** java.util.concurrent에 포함된 이 유틸리티들은 실행자 프레임워크, 병행 컬렉션(concurrent collection), 그리고 동기자(synchronizer)의 세 가지 범주로 나눌 수 있다. 이번 절에서는 병행 컬렉션과 동기자에 대해 간단히 살펴본다.

병행 컬렉션은 List, Queue, Map 등의 표준 컬렉션 인터페이스에 대한 고성능 병행 컬렉션 구현을 제공한다. 이 컬렉션들은 병행성을 높이기 위해 동기화를 내부적으로 처리한다. 따라서 컬렉션 외부에서 병행성을 처리하는 것은 불가능하다. 락을 걸어봐야 아무 효과가 없을 뿐 아니라 프로그램만 느려진다. 따라서 클라이언트는 병행 컬렉션에 대한 메서드 호출을 원자적으로 작성할 수 없다. 그래서 컬렉션 인터페이스 가운데 일부는 상태 종속 변경 연산으로 확장되었는데, 이는 몇 가지 기본 연산들을 하나의 원자적 연산으로 묶은 것이다. 예를 들어 ConcurrentMap은 Map을 확장해서 몇 가지 메서드를 추가했는데, 그 가운데 putIfAbsent(key,value)도 있다. 이 메서드는 키에 해당하는 값이 없을 때만 주어진 값을 넣고, 해당 키가 있는 경우에는 기존 값을 반환한다. 그리고 대응되는 값이 없었을 때는 null을 반환한다. 이 메서드 덕에 다중 스레드에 안전한 정규화 맵을 쉽게 구현할 수 있다.
```java
// ConcurrentMap으로 구현한 병행 정규화 맵
public static String intern(String s){
	String result = map.get(s);
	if(result == null){
		result = map.putIfAbsent(s,s);
		if(result == null)
			result = s;
	}
	return result;
}
```
**병행성이 높을 뿐 아니라, ConcurrentHashMap은 아주 빠르다. 그러니 확실한 이유가 없다면 Collections.synchronizedMap이나 Hashtable 대신 ConcurrentHashMap을 사용하도록 하자.**

컬렉션 인터페이스 가운데 몇몇은 봉쇄 연산(blocking operation)이 가능하도록 확장되었다. 성공적으로 수행될 수 있을 때까지 대기(wait)할 수 있도록 확장되었다는 것이다. 예를 들어 BlockingQueue는 Queue를 확장해서 take 같은 연산을 추가하였다. take는 큐의 맨 앞(head) 원소를 제거한 다음 반환하는데, 큐가 비어 있는 경우에는 대기한다. ThreadPoolExecutor를 비롯한 대부분의 ExecutorService 구현은 BlockingQueue를 사용한다.

동기자(synchronizer)는 스레드들이 서로를 기다릴 수 있도록 하여, 상호 협력이 가능하게 한다. 가장 널리 쓰이는 동기자로는 CountDownLatch와 Semaphore가 있다. countdown latch는 일회성 barrier로서 하나 이상의 스레드가 작업을 마칠 때까지 다른 여러 스레드가 대기할 수 있도록 한다. CountDownLatch에 정의된 유일한 생성자는 래치의 countdown 메서드가 호출될 수 있는 횟수를 나타내는 int 값을 인자로 받는다. 대기 중인 스레드가 진행할 수 있으려면 그 횟수만큼 countdown 메서드가 호출되어야 한다.

예를 들어 어떤 작업을 병렬적으로 처리하는 데 드는 시간을 재는 간단한 프레임워크를 만드는 경우를 생각해 보자. 이 프레임워크는 작업을 실행할 실행자(executor)와 병행성 수준(concurrency level)을 나타내는 int 값, 그리고 수행할 작업을 나타내는 Runnable 객체를 인자로 받는 메서드다. 시간을 재는 타이머 스레드가 실행되기 전에 모든 작업 스레드는 작업 실행 준비를 마쳐야 한다. 그 상태에서 타이머 스레드가 “출발 신호를 울리면” 작업 스레드는 작업을 수행하기 시작한다. 마지막 작업 스레드가 일을 마치면 타이머 스레드는 시계를 멈추고 시간을 잰다.
```java
public static long time(Executor executor, int concurrency, final Runnable action)
        throws InterruptedException{
    final CountDownLatch ready = new CountDownLatch(concurrency);
    final CountDownLatch start = new CountDownLatch(1);
    final CountDownLatch done = new CountDownLatch(concurrency);

    for(int i=0; i<concurrency; i++){
        executor.execute(new Runnable() {
            @Override
            public void run() {
                ready.countDown(); // 타이머에게 준비됨을 알림
                try {
                    start.await(); // 다른 작업 스레드가 준비될 때까지 대기
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown(); // 타이머에게 끝났음을 알림
                }
            }
        });
    }
    ready.await(); // 모든 작업 스레드가 준비될 때까지 대기
    long startNanos = System.nanoTime();
    start.countDown(); // 출발 !
    done.await(); // 모든 작업 스레드가 끝날 때까지 대기
    return System.nanoTime() - startNanos;
}
```
이 메서드가 세 개의 카운트다운 래치를 사용하고 있음에 유의하자. ready는 작업 스레드가 타이머 스레드에게 실행 준비가 끝났음을 알리려고 사용한다. 그런 다음 작업 스레드는 두 번째 래치 start에서 기다린다. 마지막 작업 스레드가 ready.countDown을 호출하면 타이머 스레드는 작업 시작 시간을 기록하고 start.countDown을 호출하여 모든 작업 스레드가 작업을 시작하도록 한다. 그런 다음 타이머 스레드는 세 번째 래치 done을 이용하여 마지막 작업 스레드가 일을 마친 다음에 done.countDown을 호출할 때까지 기다린다. 모든 작업 스레드가 실행을 마치면 타이머 스레드는 깨어나서 작업에 소요된 시간을 반환한다.

**특정 구간의 실행시간을 잴 때는 System.currentTimeMillis 대신 System.nanoTime을 사용해야 한다.** 그래야 더 정밀하게 잴 수 있을 뿐더러, 시스테의 실시간 클락(real-time clock) 변동에도 영향을 받지 않게 된다.

wait나 notify로 작성한 기존 코드도 유지할 필요가 있을 것이다. wait 메서드는 스레드로 하여금 어떤 조건이 만족되길 기다리도록 하고 싶을 때 사용한다. 동기화 영역 내에서 호출해야 하며, 호출 대상 객체에는 락이 걸린다. wait 메서드는 아래의 표준적 숙어대로 사용한다.
```java
// wait 메서드를 사용하는 표준적 숙어 
synchronized(obj){
	while( 이 조건이 만족되지 않을 경우에 순환문 실행 )
		obj.wait(); // 락 해제. 깨어나면 다시 락 획득

	… // 조건이 만족되면 그에 맞는 작업 실행
}
```
**wait 메서드를 호출할 때는 반드시 이 대기 순환문(wait loop) 숙어대로 하자.** 순환문 밖에서 호출하면 안된다. 이 순환문은 wait 호출 전후로 조건이 만족되었는지 검사하는 역할을 한다.

notify를 쓸 것인가 notifyAll을 쓸 것인가에 대해서는 많은 사람들이 항상 notifyAll을 쓰라고 한다. 깨어날 필요가 있는 모든 스레드를 깨우므로, 항상 정확한 결과가 나올 것이다. 필요 없는 다른 스레드도 깨어날 것이지만 프로그램의 정확성에는 영향을 끼치지 않는다. 대기 조건이 false인 것을 확인하고 나면 다시 대시 상태로 돌아갈 것이기 때문이다. 

### 규칙82 : 스레드 안전성에 대해 문서로 남겨라
클래스와 사용자 사이의 규약 가운데 중요한 것 하나는, 클래스의 객체나 정적 메서드가 병렬적으로 이용되었을 때 어떻게 동작하느냐 하는 것이다. 병렬적으로 사용해도 안전한 클래스가 되려면, 어떤 수준의 스레드 안전성을 제공하는 클래스인지 문서에 명확하게 남겨야 한다.

스레드 안전성을 수준별로 요약하였다.

**변경 불가능(immutable)** : 이 클래스로 만든 객체들은 상수다. 따라서 외부적인 동기화 메커니즘 없이도 병렬적 이용이 가능하다. String, Long, BigInteger 등이 그 예다.

**무조건적 스레드 안전성** : 이 클래스의 객체들은 변경이 가능하지만 적절한 내부 동기화 메커니즘을 갖추고 있어서 외부적으로 동기화 메커니즘을 적용하지 않아도 병렬적으로 사용할 수 있다. Random, ConcurrentHashMap 같은 클래스가 그 예다.

**조건부 스레드 안전성** : 무조건적 스레드 안전성과 거의 같은 수준이나 몇몇 스레드는 외부적 동기화가 없이는 병렬적으로 사용할 수 없다. Collections.synchronized 계열 메서드가 반환하는 포장 객체(wrapper)가 그 사례다. 이런 객체의 반복자(iterator)는 외부적 동기화 없이는 병렬적으로 이용할 수 없다.
 
**스레드 안전성 없음** : 이 클래스의 객체들은 변경 가능하다. 해당 객체들을 병렬적으로 사용하려면 클라이언트는 메서드를 호출하는 부분을 클라이언트가 선택한 외부적 동기화 수단으로 감싸야 한다. ArrayList나 HashMap 같은 일반 용도의 컬렉션 구현체들이 그 예다.

**다중 스레드에 적대적** : 이런 클래스의 객체는 설마 메서드를 호출하는 모든 부분을 외부적 동기화 수단으로 감싸더라도 안전하지 않다. 이런 클래스가 되는 것은 보통, 동기화 없이 정적 데이터를 변경하기 때문이다. 누구도 이런 클래스를 고의로 만들지는 않는다. 다행히 자바 라이브러리에는 이런 클래스가 별로 없다. System.runFinalizersOnExit 메서드는 스레드에 적대적인 대표적인 메서드로, 지금은 deprecated 되었다.

위에 언급한 범주 각각은 스레드 안전성 애노테이션 Immutable, ThreadSafe, NotThreadSafe 각각에 해당한다. 무조건적/조건적 스레드 안전성 범주는 전부 ThreadSafe 애노테이션에 해당한다.

조건부 스레드 안전성 클래스에 대한 문서를 만들 때는 신중해야 한다. 어떤 순서로 메서드를 호출할 때 외부 동기화 메커니즘을 동원해야 하는지, 그리고 그 순서로 메서드를 실행하려면 어떤 락을 사용해야 하는지 명시해야 한다. 보통은 객체 자체에 락을 걸면 되는데 예외도 있다. 다른 객체에 대한 뷰 역할을 하는 객체의 경우, 클라이언트는 원래 객체에 대해 동기화를 해야 한다. 동기화 없이 직접 변경하는 일을 막기 위해서다. 
```java
Map<K, V> m = Collections.synchronizedMap(new HashMap<K,V>();
…
Set<K> s = m.keySet(); // 동기화 블록 안에 있을 필요 없음
…
synchronized(m){ // s가 아니라 m에 대해 동기화 !
	for(K key : s)
		key.f();
}
```
enum 자료형의 경우, 변경 불가능성을 문서에 밝힐 필요는 없다. 반환값 자료형을 보고 명확하게 알 수 있는 경우를 빼고, 정적 팩토리 메서드는 자기가 반환하는 객체의 스레드 안전성을 문서에 남겨야 한다.

**내부적인 동기화 private 락 객체 패턴**
외부로 공개한 락을 통해 동기화하도록 하는 클래스의 경우, 클라이언트가 여러 메서드를 한 번에 원자적으로 호출 할 수 있다는 유연성이 있긴 하지만, 높아진 유연성만큼 대가도 따른다. ConcurrentHashMap이나 ConcurrentLinkedQueue 같은 병행 컬렉션에서 사용하는 내부적인 고속 병행성 제어 메커니즘과는 잘 어울리지 않는다. 게다가, 클라이언트가 해당 락을 오랫동안 들고 있으면 DoS 공격도 가능하다. 그런 공격을 막는 한 가지 방법은 동기화 메서드를 쓰는 대신(동기화 메서드는 클래스 외부로 공개된 락이나 다름없다) private 락 객체를 이용하는 것이다.
```java
//DoS 공격을 피하기 위한 private 락 객체 숙어
private final Object lock = new Object();

public void foo(){
	synchronized(lock){
		...
	}
}
```
이 private 락 객체는 클래스 바깥에서는 이용할 수 없으므로, 클라이언트는 객체의 동기화 메커니즘에 개입할 수 없다. lock 필드를  final로 선언한 것에 유의하자. 이렇게 하면 실수로 lock 필드의 내용을 변경하는 일을 막을 수 있다. 

**private 락 객체 패턴은 무조건적 스레드 안전성을 제공하는 클래스에만 적용할 수 있다.** 조건부 스레드 안전성을 제공하는 클래스는 이 숙어를 이용할 수 없다. 특정 순서로 메서드들을 호출할 때 클라이언트가 어떤 락을 획득하게 되는지를 문서로 만들어 남겨야 하기 때문이다.

요약하자면, 모든 클래스는 자신의 스레드 안전성 수준을 문서로 분명히 밝혀야 한다. synchronized 키워드는 이런 문서에서는 아무런 역할도 하지 못한다. 무조건적 스레드 안전성을 제공하는 클래스를 구현하는 중이라면 메서드를 synchronized로 선언하는 대신 private락 객체를 이용하면 어떨지 따져보자. 이런 락 객체를 이용하면 클라이언트나 하위 클래스가 동기화에 개입하는 것을 막을 수 있고, 다음번 릴리스에는 좀 더 복잡한 병행성 제어 전략도 쉽게 채택할 수 있게 된다.

### 규칙83 : 초기화 지연은 신중하게 하라
초기화 지연은 필드 초기화를 그 값이 쓰일 때까지 미루는 것이다. 이 기법은 static 필드와 객체 필드 모두 적용 가능하다. 초기화 지연 기법은 기본적으로 최적화 기법이다. 대부분의 최적화가 다 그렇듯이, 초기화 지연을 적용할 때 따라야 할 최고의 지침은 “정말로 필요하지 않으면 하지 마라”는 것이다. **초기화 지연 기법은 클래스를 초기화하고 객체를 생성하는 비용은 줄이지만, 필드 사용 비용은 증가시킨다.** 필드 사용 빈도가 낮고 초기화 비용이 높다면 쓸만할 것이다. 하지만 대부분의 경우 지연된 초기화를 하느니 일반 초기화를 하는 편이 낫다.

아래 코드는 통상적인 방법으로 초기화하는 전형적 필드 선언문이다. final로 선언하고 있음에 주의하자.
```java
//객체 필드를 초기화하는 일반적인 방법
private final FieldType field = computerFieldValue();
```
초기화 순환성 문제를 해소하기 위해서 초기화를 지연시키는 경우에는 동기화된 접근자를 사용하라.
```java
//동기화된 접근자를 사용한 객체 필드 초기화 지연 방법
private FieldType field;

synchronized FieldType getField(){
	if(field == null)
		field = computerFieldValue();
	return field;
}
```
**성능 문제 때문에 정적 필드 초기화를 지연시키고 싶을 때는 초기화 지연 담당 클래스 숙어를 적용하라.** 클래스는 실제로 사용되는 순간에 초기화된다는 점을 이용한 것이다.
```java
//정적 필드에 대한 초기화 지연 담당 클래스 숙어
private static class FieldHolder{
	static final FieldType field = computerFieldValue();
}
static FieldType getField(){
	return FieldHolder.field;
}
```
FieldHolder 클래스는 FieldHolder.field가 처음으로 이용되는 순간, 그러니까 getField 메서드가 처음으로 호출되는 순간에 초기화된다. 이 숙어가 좋은 점은 getField를 동기화 메서드로 선언하지 않아도 된다는 것이다. 따라서 초기화를 지연시켜도 메서드 이용 비용은 전혀 증가하지 않는다.

**성능 문제 때문에 객체 필드 초기화를 지연시키고 싶다면 이중 검사 숙어를 사용하라.** 이 숙어를 사용하면 초기화가 끝난 필드를 이용하기 위해 락을 걸어야 하는 비용을 없앨 수 있다.  이 숙어 뒤에 숨은 아이디어는 필드의 값을 두 번 검사하는 것이다. 한번은 락 없이 검사하고, 초기화가 되지 않은 것 같으면 락을 걸어서 검사한다. 이미 초기화된 필드에는 락을 걸지 않으므로, 필드는 반드시 volatile로 선언해야 한다.
```java
//이중 검사 패턴을 통해 객체 필드 초기화를 지연시키는 숙어
private volatile FieldType field; 

FieldType getField() {
	FieldType result = field;
	if (result == null) { // 첫 번째 검사(락 없음)
		synchronized(this){
			result = field;
			if(result == null) // 두 번째 검사(락)
				field = result = computerFieldValue();
		}
	}
	return result;
}
```
여기서 지역 변수 result가 하는일은, 이미 초기화된 필드는 딱 한 번만 읽도록 하는 것이다.

이중 검사 숙어의 변종 가운데는 주의할 것이 두 가지 있다. 때로 여러 번 초기화되어도 상관없는 객체 필드 초기화를 지연시키고 싶을 때가 있다. 이런 상황이라면 이중 검사 숙어의 두 번째 검사는 없애버려도 된다.
```java
//단일 검사 숙어 - 필드가 여러 번 초기화 될 수도 있다.
private volatile FieldType field; 

FieldType getField() {
	FieldType result = field;
	if(result == null) 
		field = result = computerFieldValue();
	return result;
}
```
또한 만약 모든 스레드가 필드 값을 재계산하더라도 상관없고 필드 자료형이 long이나 double이 아닌 기본 자료형인 경우에는 단일 검사 숙어에서 volatile 키워드는 빼도 된다.

### 규칙84 : 스레드 스케줄러에 의존하지 마라
좋은 프로그램이라면 스케줄링 정책에는 의존하지 말아야 한다. **정확성을 보장하거나 성능을 높이기 위해 스레드 스케줄러에 의존하는 프로그램은 이식성이 떨어진다.** 안정적이고, 즉각 반응하며 이식성이 좋은 프로그램을 만드는 가장 좋은 방법은, 실행 가능 스레드의 평균적 수가 프로세서 수보다 너무 많아지지 않도록 하는 것이다.

마찬가지로 Thread.yield나 스레드 우선순위에 의존하지도 마라. 이런 것들은 스케줄러 입장에서는 단순한 힌트일 뿐이다.