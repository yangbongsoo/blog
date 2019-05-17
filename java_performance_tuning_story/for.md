## Story05 지금까지 사용하던 for루프를 더 빠르게 할 수 있다고?
switch문은 JDK 6까지는 byte, short, char, int 이렇게 네 가지 타입을 사용한 조건 분기만 가능했지만, JDK 7부터는 String도 사용 가능하다. 일반적으로 if문에서 분기를 많이 하면 시간이 많이 소요된다고 생각한다. if문 조건 안에 들어가는 비교 구문에서 속도를 잡아먹지 않는한, if문장 자체에서는 그리 많은 시간이 소요되지 않는다. 

### 반복 구문에서의 속도는?
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

### 반복 구문에서의 필요 없는 반복
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