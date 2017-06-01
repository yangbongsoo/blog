## Story02 내가 만든 프로그램의 속도를 알고 싶다

시스템의 성능이 느릴 때 가장 먼저 해야 하는 작업은 병목 지점을 파악하는 것이다.

### System 클래스

모든 System 클래스의 메서드는 static으로 되어 있고, 그 안에서 생성된 in, out, err와 같은 객체들도 static으로 선언되어 있으며, 생성자도 없다(private으로 되어 있음).

System 클래스에서 자주 사용하지는 않지만 알아두면 매우 유용한 메서드에는 어떤 것들이 있는지 알아보자.

```java
static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
```

```java
String[] arr = new String[]{"AAA", "BBB", "CCC", "DDD", "EEE"};

String[] copiedArr = new String[3];
System.arraycopy(arr,2,copiedArr, 1, 2);

결과 : null CCC DDD
```

예를 들어 위와 같이 arraycopy 메서드를 사용했다면, arr 배열의 2번 인덱스부터 copiedArr 배열의 1번 인덱스 위치에 2개 복사하겠다는 뜻이다.

자바의 JVM에서 사용할 수 있는 설정은 크게 두가지로 나뉜다. 하나는 속성\(Property\)값이고, 다른 하나는 환경\(Environment\)값이다. 속성은 JVM에서 지정된 값들이고, 환경은 장비\(서버\)에 지정되어 있는 값들이다. 자바에서는 영어 단어 그대로 "속성"은 Properties로, "환경"은 env로 사용한다.  
먼저 Properties를 사용하는 메서드에 대해서 알아보자.

```java
// 현재 자바 속성 값들을 받아 온다.
static Properties getProperties()

// key에 지정된 자바 속성 값을 받아 온다.
static String getProperty(String key)

// key에 지정된 자바 속성 값을 받아온다. def는 해당 key가 존재하지 않을 경우 지정할 기본값이다.
static String getProperty(String key, String def)

// props 객체에 담겨 있는 내용을 자바 속성에 지정한다.
static void setProperties(Properties props)

// 자바 속성에 있는 지정된 key의 값을 value 값으로 변환한다.
static void setProperty(String key, String value)
```

이러한 자바 속성 관련 메서드를 어떻게 사용하는지 다음의 예를 통해 알아보자.

```java
public class GetProperties {
    public static void main(String args[]) {
        System.setProperty("JavaTuning", "Tune Lee");
        Properties prop = System.getProperties();
        Set key = prop.keySet();
        Iterator it = key.iterator();

        while(it.hasNext()) {
            String curKey = it.next().toString();
            System.out.format("%s=%s \n", curKey, prop.getProperty(curKey));
        }
    }
}
```

이 소스는 'Java Tuning'이라는 키를 갖는 시스템 속성에 'Tune Lee'라는 값을 지정한 후, 시스템 속성 전체 값을 화면에 출력해 주는 프로그램이다. 이 프로그램을 수행하면 수십 개의 자바 시스템 속성값을 출력한다. 그 결과 중 우리가 지정한 'Java Tuning' 키를 갖고 'Tune Lee' 값을 가지는 속성이 추가되어 출력될 것이다.

```java
// 현재 시스템 환경 값 목록을 String 형태의 맵으로 리턴한다.
static Map<String, String> getenv()

// name에 지정된 환경 변수 값을 얻는다.
static String getenv(String name)
```

```java
public class GetEnv {
    public static void main(String args[]) {
        Map<String, String> envMap = System.getenv();
        Set key = envMap.keySet();
        Iterator it = key.iterator();

        while(it.hasNext()) {
            String curKey = it.next().toString();
            System.out.format("%s=%s \n", curKey, envMap.get(curKey));
        }
    }
}
```

그리고 운영중인 코드에 절대로 사용해서는 안되는 메서드가 있다.

```java
// 자바에서 사용하는 메모리를 명시적으로 해제하도록 GC를 수행하는 메서드다.
static void gc()

// 현재 수행중인 자바 VM을 멈춘다. 
static void exit(int status)

// Object 객체에 있는 finalize()라는 메서드가 자동으로 호출되는데, GC가 알아서 해당 객체를 더 이상 참조할 필요가 없을 때 호출한다.
// 하지만 이 메서드를 호출하면 참조 해제 작업을 기다리는 모든 객체의 finalize() 메서드를 수동으로 수행해야 한다.
static void runFinalization()
```

다시 이야기하지만, 이 세 개의 메서드들은 절대 사용하면 안된다.

### System.currentTimeMillis와 System.nanoTime

이제 시간 관련 메서드에 대해서 본격적으로 알아보자.

```java
// 현재의 시간을 ms로 리턴한다(1/1,000초)
static long currentTimeMillis()
```

currentTimeMillis() 메서드에서 리턴해 주는 결과 값은 ms(밀리초)다. UTC라는 시간 표준 체계를 따르는데, 1970년 1월 1일부터의 시간을 long타입으로 리턴해 준다. 따라서 호출할 때마다 다르다.
```
20 ms 0.02초
1,200 ms 1.2초
6,000,000 ms 6,000초 
```

```java
// 현재의 시간을 ns로 리턴한다(1/1,000,000,000초)
static long nanoTime()
```
nanoTime() 메서드는 JDK 5.0부터 추가된 메서드다. 되도록이면 nanoTime() 메서드의 결과로 판단하도록 하자.

## Story03 왜 자꾸 String을 쓰지 말라는거야
###StringBuffer 클래스와 StringBuilder 클래스
StringBuilder 클래스는 JDK 5.0에서 새로 추가되었다. StringBuffer 클래스가 제공하는 메서드와 동일하다. StringBuffer 클래스는 스레드에 안전하게(ThreadSafe) 설계되어 있으므로, 여러 개의 스레드에서 하나의 StringBuffer 객체를 처리해도 전혀 문제가 되지 않는다. 하지만 StringBuilder는 단일 스레드에서의 안전성만 보장한다. 그렇기 때문에 여러 개의 스레드에서 하나의 StringBuilder 객체를 처리하면 문제가 발생한다.

StringBuffer를 기준으로 생성자와 메서드를 확인하고 정리해 보자.
```java
// 생성자
    public StringBuffer() {
        super(16); //기본 용량은 16개의 char다
    }
    
    public StringBuffer(int capacity) {
        super(capacity);
    }
    
    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str); // str 값을 갖는 StringBuffer를 생성한다
    }
    
    // CharSequence를 매개변수로 받아 그 seq 값을 갖는 StringBuffer를 생성한다
    public StringBuffer(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }            
```
cf) CharSequence는 인터페이스다. 구현체로는 CharBuffer, String, StringBuffer, StringBuilder가 있으며 StringBuffer나 StringBuilder로 생성한 객체를 전달할 때 사용된다.

```java
    @Test
    public void stringTest() throws Exception {
        StringBuilder sb = new StringBuilder();
        sb.append("ABCDE");
        check(sb);
    }

    private void check(CharSequence cs) {
        StringBuffer sb = new StringBuffer(cs);
        System.out.println(sb.length()); // 5
    }
```
StringBuffer나 StringBuilder로 값을 만든 후 굳이 toString을 수행하여 필요 없는 객체를 만들어서 넘겨주기보다는 CharSequence로 받아서 처리하는 것이 메모리 효율에 더 좋다.

주로 사용하는 메서드는 append()와 insert()다. 
```java
    @Test
    public void stringTest() throws Exception {
        StringBuffer sb = new StringBuffer();
        sb.append("ABC");
        sb.insert(1,"123");
        System.out.println(sb); // A123BC
    }
```

**많은 양의 문자열을 연산 할때 String을 쓰지 말아야 하는 이유**는 String은 immutable 객체(생성 후 변경 불가한 객체)이기 때문이다. 그래서 연산할 때마다 새로운 String 클래스 객체가 만들어지고 이전 객체는 필요 없는 쓰레기 값이 되어 GC 대상이 된다. 이런 작업이 반복 수행되면서 메모리를 많이 사용하게 되고, 응답속도에도 많은 영향을 미치게 된다. 반면에 StringBuffer나 StringBuilder는 새로운 객체를 생성하지 않고, 기존에 있는 객체의 크기를 증가시키면서 값을 더한다.


cf) JDK 5.0 이상을 사용하면 아래와 같이 StringBuilder로 변환된다. 개발자의 실수를 어느 정도는 피할 수 있게 된 것이다.
```java
public class VersionTest {
    String str = "Here" + "is " + "a " + "sample.";
    public VersionTest() {
        int i = 1;
        String str2 = "Here " + "is " + i + " sample.";
    }
}

// 역 컴파일한 소스
public class VersionTest {
    public VersionTest() {
        str = "Here is a sample.";
        int i = 1;
        String str2 = (new StringBuilder("Here is "))
            .append(i).append(" sample.").toString();
    }
    String str;
}
```

## Story04 어디에 담아야 하는지...
### JMH 설치 및 설정 방법
JMH는 JDK를 오픈 소스로 제공하는 OpenJDK에서 만든 성능 측정용 라이브러리다. 

**소스 받기**
먼저 Mercurial이라는 분산 저장소에 접근하는 hg라는 툴을 이용하여 소스 코드를 받는다.
url : https://www.mercurial-scm.org/downloads

hg 설치를 마쳤으면 원하는 디렉터리에서 다음 명령을 실행한다.

```
$ hg clone http://hg.openjdk.java.net/code-tools/jmh/ jmh
```

정상적으로 코드 다운로드가 완료되었으면 다음의 명령을 사용하여 메이븐 빌드를 실행한다.

```
$ cd jmh
$ mvn clean install -DskipTests=true
```

정상적으로 프로젝트 빌드가 완료되었다면 메이븐 로컬 저장소에 JMH 라이브러리가 등록되어 있을 것이다.

그리고 동일 디렉토리에서 벤치마크 프로젝트를 빌드한다.
```
$ mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.openjdk.jmh -DarchetypeArtifactId=jmh-java-benchmark-archetype -DgroupId=org.sample -DartifactId=test -Dversion=1.0
```

마지막으로 test 디렉토리에 있는 것을 빌드하면 기본적인 설정은 끝난다.
```
$ cd test
$ mvn clean install
```

**간단한 예제**
```java
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class MyBenchmark {

    @Benchmark
    public DummyData makeObjectWithSize() {
        HashMap<String, String> map = new HashMap<>(1000000);
        ArrayList<String> list = new ArrayList<>(1000000);
        return new DummyData(map, list);
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(MyBenchmark.class.getSimpleName())
                .warmupIterations(5)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```

`@Benchmark`라는 애노테이션을 메서드에 선언하면 JMH에서 측정 대상 코드라고 인식한다. 
- 하나의 클래스에 여러 개의 메서드가 존재할 수 있다. 
- 애노테이션을 선언한 메서드가 끝나지 않으면 측정도 끝나지 않는다. 
- 예외가 발생할 경우 해당 메서드의 측정을 종료하고, 다음 측정 메서드로 이동한다.


위의 수행 결과는 다음과 같다.
```
# JMH version: 1.19
# VM version: JDK 1.8.0_60, VM 25.60-b23
# VM invoker: /Library/Java/JavaVirtualMachines/jdk1.8.0_60.jdk/Contents/Home/jre/bin/java
# VM options: -Didea.launcher.port=7537 -Didea.launcher.bin.path=/Volumes/IntelliJ IDEA/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8
# Warmup: 5 iterations, 1 s each
# Measurement: 5 iterations, 1 s each
# Timeout: 10 min per iteration
# Threads: 1 thread, will synchronize iterations
# Benchmark mode: Average time, time/op
# Benchmark: org.sample.MyBenchmark.makeObjectWithSize

# Run progress: 0.00% complete, ETA 00:00:10
# Fork: 1 of 1
# Warmup Iteration   1: 1.608 ms/op
# Warmup Iteration   2: 0.612 ms/op
# Warmup Iteration   3: 0.564 ms/op
# Warmup Iteration   4: 0.572 ms/op
# Warmup Iteration   5: 0.553 ms/op
Iteration   1: 0.571 ms/op
Iteration   2: 0.555 ms/op
Iteration   3: 0.558 ms/op
Iteration   4: 0.557 ms/op
Iteration   5: 0.557 ms/op


Result "org.sample.MyBenchmark.makeObjectWithSize":
  0.560 ±(99.9%) 0.025 ms/op [Average]
  (min, avg, max) = (0.555, 0.560, 0.571), stdev = 0.007
  CI (99.9%): [0.534, 0.585] (assumes normal distribution)


# Run complete. Total time: 00:00:10

Benchmark                       Mode  Cnt  Score   Error  Units
MyBenchmark.makeObjectWithSize  avgt    5  0.560 ± 0.025  ms/op

Process finished with exit code 0
```
###Set 클래스 중 무엇이 가장 빠를까?
먼저 HashSet, TreeSet, LinkedHashSet add를 비교해보면 다음과 같다.
```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class SetAdd {
    int LOOP_COUNT = 1000;
    Set<String> set;
    String data = "abcdefghijklmnopqrstuvwxyz";

    @Benchmark
    public void addHashSet() {
        set = new HashSet<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            set.add(data + loop);
        }
    }

    @Benchmark
    public void addTreeSet() {
        set = new TreeSet<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            set.add(data + loop);
        }
    }

    @Benchmark
    public void addLinkedHashSet() {
        set = new LinkedHashSet<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            set.add(data + loop);
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(SetAdd.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```
# Run complete. Total time: 00:00:25

Benchmark                Mode  Cnt    Score    Error  Units
SetAdd.addHashSet        avgt    5   79.923 ±  7.503  us/op
SetAdd.addLinkedHashSet  avgt    5   84.106 ± 10.006  us/op
SetAdd.addTreeSet        avgt    5  297.550 ± 56.801  us/op
```
HashSet과 LinkedHashSet의 성능이 비슷하고, TreeSet은 성능 차이가 발생한다 TreeSet은 레드블랙 트리에 데이터를 담는다. 값에 따라서 순서가 정해진다. 데이터를 담으면서 동시에 정렬을 하기 때문에 HashSet보다 성능상 느리다. 

이번에는 Set 클래스들이 데이터를 읽을 때 얼마나 많은 차이가 발생하는지 확인해보자.
```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class SetIterate {
    int LOOP_COUNT = 1000;
    Set<String> hashSet;
    Set<String> treeSet;
    Set<String> linkedHashSet;

    String data = "abcdefghijklmnopqrstuvwxyz";
    String[] keys;
    String result = null;

    @Setup(Level.Trial)
    public void setUp() {
        hashSet = new HashSet<>();
        treeSet = new TreeSet<>();
        linkedHashSet = new LinkedHashSet<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            String tempData = data+loop;
            hashSet.add(tempData);
            treeSet.add(tempData);
            linkedHashSet.add(tempData);
        }
    }

    @Benchmark
    public void iterateHashSet() {
        Iterator<String> iter = hashSet.iterator();
        while (iter.hasNext()) {
            result = iter.next();
        }
    }

    @Benchmark
    public void iterateTreeSet() {
        Iterator<String> iter = treeSet.iterator();
        while (iter.hasNext()) {
            result = iter.next();
        }
    }

    @Benchmark
    public void iterateLinkedHashSet() {
        Iterator<String> iter = linkedHashSet.iterator();
        while (iter.hasNext()) {
            result = iter.next();
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(SetIterate.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
```

```
# Run complete. Total time: 00:00:25

Benchmark                        Mode  Cnt   Score   Error  Units
SetIterate.iterateHashSet        avgt    5   8.154 ± 2.364  us/op
SetIterate.iterateLinkedHashSet  avgt    5  11.204 ± 0.577  us/op
SetIterate.iterateTreeSet        avgt    5  12.678 ± 1.558  us/op
```
읽기에서는 크게 차이나지 않는다. Set을 Iterator 돌려서 사용하기도 하지만, 일반적으로 Set은 여러 데이터를 넣어 두고 해당 데이터가 존재하는지를 확인하는 용도로 많이 사용된다. 따라서 데이터를 Iterator로 가져오는 것이 아니라, 랜덤하게 가져와야만 한다.

```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class SetContains {
    int LOOP_COUNT = 1000;
    Set<String> hashSet;
    Set<String> treeSet;
    Set<String> linkedHashSet;

    String data = "abcdefghijklmnopqrstuvwxyz";
    String[] keys;
    String result = null;

    @Setup(Level.Trial)
    public void setUp() {
        hashSet = new HashSet<>();
        treeSet = new TreeSet<>();
        linkedHashSet = new LinkedHashSet<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            String tempData = data+loop;
            hashSet.add(tempData);
            treeSet.add(tempData);
            linkedHashSet.add(tempData);
        }

        if (keys == null || keys.length != LOOP_COUNT) {
            keys = RandomKeyUtil.generateRandomSetKeysSwap(hashSet);
        }
    }

    @Benchmark
    public void containsHashSet() {
        for (String key : keys) {
            hashSet.contains(key);
        }
    }

    @Benchmark
    public void containsTreeSet() {
        for (String key : keys) {
            treeSet.contains(key);
        }
    }

    @Benchmark
    public void containsLinkedHashSet() {
        for (String key : keys) {
            linkedHashSet.contains(key);
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(SetContains.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```java
public class RandomKeyUtil {
    public static String[] generateRandomSetKeysSwap(Set<String> set) {
        int size = set.size();
        String[] result = new String[size];
        Random random = new Random();
        int maxNumber = size;
        Iterator<String> iterator = set.iterator();
        int resultPos = 0;
        while (iterator.hasNext()) {
            result[resultPos++] = iterator.next();
        }
        for (int loop = 0; loop < size; loop++) {
            int randomNumber1 = random.nextInt(maxNumber);
            int randomNumber2 = random.nextInt(maxNumber);
            String temp = result[randomNumber2];
            result[randomNumber2] = result[randomNumber1];
            result[randomNumber1] = temp;
        }
        return result;
    }

    public static int[] generateRandomNumberKeysSwap(int loop_count) {
        int[] result = new int[loop_count];
        Random random = new Random();
        for (int i=0; i< loop_count; i++) {
            int randomNumber = random.nextInt(loop_count);
            result[i] = randomNumber;
        }
        return result;
    }
}
```
```
# Run complete. Total time: 00:00:26

Benchmark                          Mode  Cnt    Score    Error  Units
SetContains.containsHashSet        avgt    5    9.446 ±  2.284  us/op
SetContains.containsLinkedHashSet  avgt    5    9.875 ±  4.718  us/op
SetContains.containsTreeSet        avgt    5  239.340 ± 88.700  us/op
```
HashSet과 LinkedHashSet의 속도는 빠르지만, TreeSet의 속도는 느리다는 것을 알 수 있다. 그러면 왜 결과가 항상 느리게 나오는 TreeSet 클래스를 만들었을까? 

```
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
```
TreeSet은 데이터를 저장하면서 정렬한다. 구현한 인터페이스 중에 NavigableSet이 있다. 이 인터페이스는 특정 값보다 큰 값이나 작은 값, 가장 큰 값, 가장 작은 값 등을 추출하는 메서드를 선언해 놓았으며 JDK 1.6부터 추가된 것이다. 즉, 데이터를 순서에 따라 탐색하는 작업이 필요할때는 TreeSet을 사용하는 것이 좋다는 의미다. 하지만 그럴 필요가 없을 때는 HashSet이나 LinkedHashSet을 사용하는 것을 권장한다.

###List 관련 클래스 중 무엇이 빠를까?
데이터를 넣는 속도부터 비교해보자. 
```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class ListAdd {
    int LOOP_COUNT = 1000;
    List<Integer> arrayList;
    List<Integer> vector;
    List<Integer> linkedList;

    @Benchmark
    public void addArrayList() {
        arrayList = new ArrayList<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            arrayList.add(loop);
        }
    }

    @Benchmark
    public void addVector() {
        vector = new Vector<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            vector.add(loop);
        }
    }

    @Benchmark
    public void addLinkedList() {
        linkedList = new LinkedList<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            linkedList.add(loop);
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(ListAdd.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```
# Run complete. Total time: 00:00:26

Benchmark              Mode  Cnt   Score    Error  Units
ListAdd.addArrayList   avgt    5  12.038 ±  3.928  us/op
ListAdd.addLinkedList  avgt    5  11.656 ±  8.859  us/op
ListAdd.addVector      avgt    5  12.376 ± 21.172  us/op
```
어떤 클래스든 큰 차이가 없다는 것을 알 수 있다.

이번에는 데이터를 꺼내는 속도를 확인해보자.
```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class ListGet {
    int LOOP_COUNT = 1000;
    List<Integer> arrayList;
    List<Integer> vector;
    List<Integer> linkedList;
    int result = 0;

    @Setup
    public void setUp() {
        arrayList = new ArrayList<>();
        vector = new Vector<>();
        linkedList = new LinkedList<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            arrayList.add(loop);
            vector.add(loop);
            linkedList.add(loop);
        }
    }

    @Benchmark
    public void getArrayList() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            result = arrayList.get(loop);
        }
    }

    @Benchmark
    public void getVector() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            result = vector.get(loop);
        }
    }

    @Benchmark
    public void getLinkedList() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            result = linkedList.get(loop);
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(ListGet.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```
# Run complete. Total time: 00:00:26

Benchmark              Mode  Cnt    Score     Error  Units
ListGet.getArrayList   avgt    5    1.282 ±   0.243  us/op
ListGet.getLinkedList  avgt    5  435.129 ± 105.285  us/op
ListGet.getVector      avgt    5   29.920 ±   6.519  us/op
```
ArrayList의 속도가 가장 빠르고, Vector와 LinkedList는 속도가 매우 느리다. LinkedList가 터무니없이 느리게 나온 이유는 LinkedList가 Queue 인터페이스를 상속받기 때문이다. 이를 수정하기 위해서는 순차적으로 결과를 받아오는 peek() 메서드를 사용해야 한다.
```java
@Benchmark
public void getPeekLinkedList() {
    for (int loop = 0; loop < LOOP_COUNT; loop++) {
        result = linkedList.peek();
    }
}
```
```
# Run complete. Total time: 00:00:34

Benchmark                  Mode  Cnt    Score    Error  Units
ListGet.getArrayList       avgt    5    1.242 ±  0.180  us/op
ListGet.getLinkedList      avgt    5  425.459 ± 54.941  us/op
ListGet.getPeekLinkedList  avgt    5    0.038 ±  0.003  us/op
ListGet.getVector          avgt    5   27.923 ±  0.632  us/op
```
LinkedList 클래스를 사용할 때는 get() 메서드가 아닌 peek()이나 poll() 메서드를 사용해야 한다. 그런데 왜 ArrayList와 Vector의 성능 차이가 이렇게 클까? ArrayList는 여러 스레드에서 접근할 경우 문제가 발생할 수 있지만, Vector는 여러 스레드에서 접근할 경우를 방지하기 위해서 get() 메서드에 synchronized가 선언되어 있다. 따라서 성능 저하가 발생할 수 밖에 없다.

마지막으로 데이터를 삭제하는 속도를 비교해보자.
```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class ListRemove {
    int LOOP_COUNT = 10;
    List<Integer> arrayList;
    List<Integer> vector;
    LinkedList<Integer> linkedList;
    int result = 0;

    @Setup(Level.Trial)
    public void setUp() {
        arrayList = new ArrayList<>();
        vector = new Vector<>();
        linkedList = new LinkedList<>();
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            arrayList.add(loop);
            vector.add(loop);
            linkedList.add(loop);
        }
    }

    @Benchmark
    public void removeArrayListFromFirst() {
        ArrayList<Integer> tempList = new ArrayList<>(arrayList);
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            tempList.remove(0);
        }
    }

    @Benchmark
    public void removeArrayListFromLast() {
        ArrayList<Integer> tempList = new ArrayList<>(arrayList);
        for (int loop = LOOP_COUNT -1 ; loop >= 0; loop--) {
            tempList.remove(loop);
        }
    }

    @Benchmark
    public void removeVectorFromFirst() {
        List<Integer> tempList = new Vector<>(vector);
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            tempList.remove(0);
        }
    }

    @Benchmark
    public void removeVectorFromLast() {
        List<Integer> tempList = new Vector<>(vector);
        for (int loop = LOOP_COUNT -1 ; loop >= 0; loop--) {
            tempList.remove(loop);
        }
    }

    @Benchmark
    public void removeLinkedListFromFirst() {
        LinkedList<Integer> tempList = new LinkedList<>(linkedList);
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            tempList.remove(0);
        }
    }

    @Benchmark
    public void removeLinkedListFromLast() {
        LinkedList<Integer> tempList = new LinkedList<>(linkedList);
        for (int loop = LOOP_COUNT -1 ; loop >= 0; loop--) {
            tempList.remove(loop);
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(ListRemove.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```
# Run complete. Total time: 00:00:51

Benchmark                             Mode  Cnt  Score   Error  Units
ListRemove.removeArrayListFromFirst   avgt    5  0.089 ± 0.012  us/op
ListRemove.removeArrayListFromLast    avgt    5  0.023 ± 0.001  us/op
ListRemove.removeLinkedListFromFirst  avgt    5  0.161 ± 0.094  us/op
ListRemove.removeLinkedListFromLast   avgt    5  0.137 ± 0.021  us/op
ListRemove.removeVectorFromFirst      avgt    5  0.097 ± 0.003  us/op
ListRemove.removeVectorFromLast       avgt    5  0.038 ± 0.007  us/op
```
결과를 보면 첫 번째 값을 삭제하는 메서드와 마지막 값을 삭제하는 메서드의 속도 차이는 크다. 그리고 LinkedList는 별 차이가 없다. 그 이유가뭘까? ArrayList와 Vector는 실제로 그 안에 배열을 사용하기 때문에 0번째 인덱스 값을 삭제하면 나머지를 다 옮겨야 하기 때문이다.

###Map 관련 클래스 중에서 무엇이 빠를까?
```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class MapGet {
    int LOOP_COUNT = 1000;
    Map<Integer, String> hashMap;
    Map<Integer, String> hashTable;
    Map<Integer, String> treeMap;
    Map<Integer, String> linkedHashMap;
    int[] keys;

    @Setup(Level.Trial)
    public void setUp() {
        if (keys == null || keys.length != LOOP_COUNT) {
            hashMap = new HashMap<>();
            hashTable = new Hashtable<>();
            treeMap = new TreeMap<>();
            linkedHashMap = new LinkedHashMap<>();
            String data = "abcdefghijklmnopqrstuvwxyz";
            for (int loop = 0; loop < LOOP_COUNT; loop++) {
                String tempData = data+loop;
                hashMap.put(loop, tempData);
                hashTable.put(loop, tempData);
                treeMap.put(loop, tempData);
                linkedHashMap.put(loop, tempData);
            }
            keys = RandomKeyUtil.generateRandomNumberKeysSwap(LOOP_COUNT);
        }
    }

    @Benchmark
    public void getSeqHashMap() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            hashMap.get(loop);
        }
    }

    @Benchmark
    public void getRandomHashMap() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            hashMap.get(keys[loop]);
        }
    }

    @Benchmark
    public void getSeqHashtable() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            hashTable.get(loop);
        }
    }

    @Benchmark
    public void getRandomHashtable() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            hashTable.get(keys[loop]);
        }
    }

    @Benchmark
    public void getSeqTreeMap() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            treeMap.get(loop);
        }
    }

    @Benchmark
    public void getRandomTreeMap() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            treeMap.get(keys[loop]);
        }
    }

    @Benchmark
    public void getSeqLinkedHashMap() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            linkedHashMap.get(loop);
        }
    }

    @Benchmark
    public void getRandomLinkedHashMap() {
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            linkedHashMap.get(keys[loop]);
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(MapGet.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```
# Run complete. Total time: 00:01:08

Benchmark                      Mode  Cnt   Score    Error  Units
MapGet.getRandomHashMap        avgt    5   7.069 ±  0.735  us/op
MapGet.getRandomHashtable      avgt    5  28.774 ±  2.440  us/op
MapGet.getRandomLinkedHashMap  avgt    5   8.615 ±  2.757  us/op
MapGet.getRandomTreeMap        avgt    5  72.531 ±  3.146  us/op
MapGet.getSeqHashMap           avgt    5   7.663 ±  3.393  us/op
MapGet.getSeqHashtable         avgt    5  28.862 ± 11.409  us/op
MapGet.getSeqLinkedHashMap     avgt    5   5.933 ±  0.450  us/op
MapGet.getSeqTreeMap           avgt    5  55.169 ±  8.445  us/op
```
트리 형태로 처리하는 TreeMap 클래스가 가장 느린 것을 알 수 있다. 그리고 동기화처리를 하 Hashtable도 HashMap과 속도차이를 보인다.

지금까지 Set, List, Map 관련 클래스에 어떤 것들이 있고, 각각의 성능이 얼마나 되는지 정확하게 측정해 보았다. 하지만 일반적인 웹을 개발할때는 Collection 성능 차이를 비교하는 것은 큰 의미가 없다. 각 클래스에는 사용 목적이 있기 때문에 목적에 부합하는 클래스를 선택해서 사용하는 것이 바람직하다.

## Story05 지금까지 사용하던 for루프를 더 빠르게 할 수 있다고?
switch문은 JDK 6까지는 byte, short, char, int 이렇게 네 가지 타입을 사용한 조건 분기만 가능했지만, JDK 7부터는 String도 사용 가능하다. 일반적으로 if문에서 분기를 많이 하면 시간이 많이 소요된다고 생각한다. if문 조건 안에 들어가는 비교 구문에서 속도를 잡아먹지 않는한, if문장 자체에서는 그리 많은 시간이 소요되지 않는다. 

###반복 구문에서의 속도는?
JDK 5.0 이전에는 for 구문을 다음과 같이 사용하였다. 여기서 list는 값이 들어있는 ArrayList이다.
```java
for (int loop = 0; loop < list.size(); loop++) 
```
이렇게 코딩을 하는 습관은 좋지 않다. 매번 반복하면서 list.size() 메서드를 호출하기 때문이다. 이럴 때는 다음과 같이 수정해야 한다.
```java
int listSize = list.size();
for (int loop = 0; loop < listSize; loop++) 
```
이렇게 하면 필요 없는 size() 메서드 반복 호출이 없어지므로 더 빠르게 처리된다. JDK 5.0부터는 다음과 같이 for-each 를 사용할 수 있다.
```java
ArrayList<String> list = new ArrayList<String>();
…
for (String str : list)
```

```java
@State(Scope.Thread)
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class ForLoop {
    int LOOP_COUNT = 100000;
    List<Integer> list;

    @Setup
    public void setUp() {
        list = new ArrayList<>(LOOP_COUNT);
        for (int loop = 0; loop < LOOP_COUNT; loop++) {
            list.add(loop);
        }
    }

    @Benchmark
    public void traditionalForLoop() {
        int listSize = list.size();
        for (int loop = 0; loop < listSize; loop++) {
            resultProcess(list.get(loop));
        }
    }

    @Benchmark
    public void traditionalSizeForLoop() {
        for (int loop = 0; loop < list.size(); loop++) {
            resultProcess(list.get(loop));
        }
    }

    @Benchmark
    public void timeForEachLoop() {
        for (Integer loop : list) {
            resultProcess(loop);
        }
    }

    @Benchmark
    public void timeForEachLoopJava8() {
        list.forEach(this::resultProcess);
    }


    int current;
    public void resultProcess(int result) {
        current = result;
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(ForLoop.class.getSimpleName())
                .warmupIterations(3)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}
```
```
# Run complete. Total time: 00:00:33

Benchmark                       Mode  Cnt    Score    Error  Units
ForLoop.timeForEachLoop         avgt    5  159.759 ± 97.272  us/op
ForLoop.timeForEachLoopJava8    avgt    5  108.508 ± 11.156  us/op
ForLoop.traditionalForLoop      avgt    5  117.491 ± 15.692  us/op
ForLoop.traditionalSizeForLoop  avgt    5  131.080 ± 46.861  us/op
```
결과를 보면 자바8의 forEach를 사용한게 가장 빠른것으로 나오고, 일반 for-each가 가장 느리게 나온다. 그리고 for문 돌때마다 list.size() 메서드를 호출이 때문에 그렇지 않은것보다 약간 느리게 나왔다.

cf) 중괄호 안에서 아무런 작업을 하지 않거나, resultProcess() 메서드를 호출하지 않을 경우, 자바의 JIT(Just In Time) 컴파일러는최적화를 통해 해당 코드를 무시해 버릴 수도 있다. 그래서 메서드 호출을 하도록 했다.

###반복 구문에서의 필요 없는 반복
가장 많은 실수 중 하나는 반복 구문에서 계속 필요 없는 메서드 호출을 하는 것이다. 다음 소스를 보자.
```java
public void sample(DataVo data, String key) {
	TreeSet treeSet2 = null;
	treeSet2 = (TreeSet)data.get(key);
	if (treeSet2 != null) {
		for (int i=0; i< treeSet2.size(); i++) {
			DataVO2 data2 = (DataVO2)treeSet2.toArray()[i];
			...
		}
	}
}
```
TreeSet 형태의 테이터를 갖고 있는 DataVO에서 TreeSet을 하나 추출하여 처리하는 부분이다. 이 소스의 문제는 toArray() 메서드를
반복해서 수행한다는 것이다. 참고로 sample 메서드는 애플리케이션이 한 번 호출되면 40번씩 수행된다. 또한 treeSet2 객체에 256개의
데이터들이 들어가 있으므로, 결과적으로 toArray() 메서드는 10,600번씩 반복 호출된다. 그러므로 이 코드는 toArray() 메서드가
반복되지 않도록 for 문 앞으로 옮기는 것이 좋다. 게다가 이 소스의 for문을 보면 treeSet2.size() 메서드를 지속적으로 호출하도록 되어 있다. 수정한 결과는 다음과 같다.
```java
public void sample(DataVo data, String key) {
	TreeSet treeSet2 = null;
	treeSet2 = (TreeSet)data.get(key);
	if (treeSet2 != null) {
		DataVO2[] dataVO2 = (DataVO2)treeSet2.toArray();
		int treeSetSize = treeSet2.size();
		for (int i=0; i< treeSetSize; i++) {
			DataVO2 data2 = dataVO2[i];
			...
		}
	}
}
```