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
### Set 클래스 중 무엇이 가장 빠를까?
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

### List 관련 클래스 중 무엇이 빠를까?
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

### Map 관련 클래스 중에서 무엇이 빠를까?
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