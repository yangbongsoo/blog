# HashMap 효율적으로 사용하기

```java
//create a map in java 11
var productPrice = new HashMap<String, Double>();
//or in java 8
Map<String, Double> productPrice = new HashMap<>();
// add value
productPrice.put("Rice", 6.9);
productPrice.put("Flour", 3.9);
productPrice.put("Sugar", 4.9);
productPrice.put("Milk", 3.9);
productPrice.put("Egg", 1.9);

//get value
Double egg = productPrice.get("Egg");
```

### HashMap의 Key, Value 출력
1. 모든 Key값 출력 <br>
```java
Set<String> keys = productPrice.keySet();
//print all the keys
for (String key : keys) {
    System.out.println(key);
}
```
```java
Set<String> keys = productPrice.keySet();
keys.forEach(key -> System.out.println(key));
```
```java
Set set = productPrice.keySet();
Iterator iterator = set.iterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

2. 모든 Value값 출력 <br>
```java
Collection<Double> values = productPrice.values();
for (Double value : values) {
	System.out.println(value);
}
```
```java
Collection<Double> values = productPrice.values();
values.forEach(value -> System.out.println(value));
```
```java
Collection values = productPrice.values();
Iterator iterator = values.iterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

3. Key, Value 함께 출력 <br>
```java
Set<Map.Entry<String, Double>> entries = productPrice.entrySet();
for (Map.Entry<String, Double> entry : entries) {
    System.out.print("key: "+ entry.getKey());
    System.out.println(", Value: "+ entry.getValue());
}
```
```java
productPrice.forEach((key, value) -> {
    System.out.print("key: "+ key);
    System.out.println(", Value: "+ value);
});
```
```java
Set set = productPrice.entrySet();
Iterator it = set.iterator();

while (it.hasNext()) {
    Map.Entry e = (Map.Entry)it.next();
    System.out.println(e.getKey() + e.getValue());
}
```
Map은 Iterator가 없기 때문에 entrySet() 메서드를 통해 Set에 key와 value 결합한 형태로 저장시킨다. 그리고 Map 내부 인터페이스인 Map.Entry를 통해 key와 value를 얻어온다. <br>
entrySet()을 이용해서 key와 value를 함께 읽어 올수도 있고 keySet()이나 values()를 이용해서 키와 값을 따로 읽어 올 수 있다. <br>
key는 중복을 허용하지 않으니까 Set타입으로 반환하고 value는 중복을 허용하니까 Collection 타입으로 반환한다. <br>

### Key 중복 확인
```
if (productPrice.containsKey("Rice") ) {
    ...
} else {
    ...
}
```
cf) Map에서 키가 중복되면 기존의 값을 덮어쓴다.

### computeIfAbsent() VS puIfAbsent()
```java
var theKey = "Fish";
// key값이 이미 존재하면 callExpensiveMethodToFindValue 메서드는 호출되지 않는다.
productPriceMap.computeIfAbsent(theKey, key -> callExpensiveMethodToFindValue(key));

// key값이 이미 존재해도 callExpensiveMethodToFindValue 메서드는 호출된다.
productPriceMap.putIfAbsent(theKey, callExpensiveMethodToFindValue(theKey));
```

### computeIfPresent() VS compute()
오라클이 Java EE를 Eclipse Foundation에 기부 한 사실은 모두 알고 있으며, 결과적으로 Jakarta EE로 이름이 바뀌었다. <br>
이 주제에 관한 기사를 제공하는 프로그램을 작성하고 단어 선택 빈도를 계산하여 사람들이이 단어를 얼마나 자주 언급하고 있는지 알아보자. <br>

```java
import java.util.HashMap;
import java.util.Map;

public class WordFrequencyFinder {
    private Map<String, Integer> map = new HashMap<>();

    {
        map.put("Java", 0);
        map.put("Jakarta", 0);
        map.put("Eclipse", 0);
    }

    public void read(String text) {
        for (String word : text.split(" ")) {
            if (map.containsKey(word)) {
                Integer value = map.get(word);
                map.put(word, ++value);
            }
        }
    }
}

```
text 문자열을 쪼개서 map에 key값으로 있는 단어들이 있으면 value를 증가시키는 간단한 코드다. 자바8에서는 computeIfPresent 메서드를 통해 더 간단하게 표현할 수 있다.
```java
public void read(String text) {
  for (String word : text.split(" ")) {
    map.computeIfPresent(word, (String key, Integer value) -> ++value);
  }
}
```
computeIfPresent 메서드는 key와 remapping function 두개의 인자를 갖는다. 그래서 key가 있을 때만 value를 compute한다. <br>
Map에는 compute 메서드도 존재하는데 key값이 있든 없든간에 무조건 remapping function을 compute한다. <br>

### getOrDefault()
key값이 없을 때도 value를 얻길 원할 때 사용된다.
```java
productPriceMap.getOrDefault("Fish", 29.4);
```

원문 : https://dzone.com/articles/how-to-use-java-hashmap-effectively?fbclid=IwAR1ZMb6aImx-7Ry-GD6S-YfxdDkrRvlSo-SlSVL08gRDxWP_zilcVnuXXtM
참고 : 자바의 정석