# Java8 in Action
##1장 - 자바8을 눈여겨봐야 하는 이유 
**Stream processing** : stream이란 한번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임을 말한다. <br>
**동작 파라미터화** : 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공한다. <br>
**병렬성과 공유 가변 데이터** : 다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만드려면 공유된 가변 데이터에 접근하지 말아야 한다. 이런 함수를 pure 함수, stateless 함수라 부른다. <br>

###자바 함수
자바8에서는 함수를 새로운 값의 형식으로 추가했다.(즉, 함수 자체가 값) <br>

**메서드 레퍼런스**<br>
ex) 디렉토리에서 모든 숨겨진 파일을 필터링하는 문제에서 우선 주어진 파일이 숨겨져 있는지 체크하는 기능
```
File[] hiddenFiles = new File(“.”).listFiles(new FileFilter(){
	public boolean accept(File file){
		return file.isHidden(); //숨겨진 파일 필터링. 
	}
}
```
위의 코드를 보면 자바8 전까지는 File 클래스에 이미 isHidden이라는 메서드가 있는데 FileFilter로 감싼 다음에 FileFilter를 인스턴스화해야 했다. 
```
File[] hiddenFiles = new File(“.”).listFiles(File::isHidden);
```
**자바8의 메서드 레퍼런스 ::** (이 메서드를 값으로 사용하라는 의미)를 이용해서 listFiles에 직접 전달할 수 있다. 기존에 객체 레퍼런스(new로 객체 레퍼런스를 생성함)를 이용해서 객체를 이리저리 주고받았던 것처럼 자바 8에서는 File::isHidden을 이용해서 메서드 레퍼런스를 만들어 전달할 수 있게 되었다. <br>

**람다: 익명 함수**<br>
함수도 값으로 취급할 수 있다. ex) (int x) -> x+1 : x라는 인수를 호출하면 x+1을 반환하라. <br>

**코드 넘겨주기: 예제**<br>
Apple이라는 클래스와 getColor라는 메서드가 있고, Apples 리스트를 포함하는 inventory라는 변수가 있다고 가정하자. 이때 모든 녹색 사과를 선택해서 리스트를 반환하는 프로그램을 구현하려 한다. 이처럼 특정 항목을 선택해서 반환하는 동작을 ‘필터'라고 한다. 자바8 이전에는 다음처럼 filterGreenApples라는 메서드를 구현했을 것이다.
```
public static List<Apple> filterGreenApples(List<Apple> inventory){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if(“green”.equals(apple.getColor()){
			result.add(apple);
		}
	}
	return result; 
}
```
하지만 누군가는 애플을 무게로 필터링 하고 싶을 수 있다. 그러면 전체 코드를 복붙해서 다음처럼 구현할 수 있을 것이다. 
```
public static List<Apple> filterGreenApples(List<Apple> inventory){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if(apple.getWeight() > 150){
			result.add(apple);
		}
	}
	return result; 
}
```
중복의 단점이 드러나는 안좋은 방법이다. 자바8 에서는 코드를 인수로 넘겨줄 수 있어서 filter 메서드를 중복으로 구현할 필요가 없다. 
```
public static boolean isGreenApple(Apple apple){
    return "green".equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple){
    return apple.getWeight() > 150;
}

static List<Apple> filterApples(List<Apple> inventory,
                                Predicate<Apple> p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
        if(p.test(apple)){
            result.add(apple);
        }
    }
    return result;
}
```
다음 처럼 메서드를 호출 할 수 있다. <br>
`filterApples(inventory, Apple::isGreenApple);`<br>
`filterApples(inventory, Apple::isHeavyApple);`<br>

cf) Predicate : 수학에서는 인수로 값을 받아 true / false를 반환하는 함수를 Predicate라고 한다. <br>

**메서드 전달에서 람다로 **<br>
isHeavyApple, isGreenApple처럼 한두 번만 사용할 메서드를 매번 정의하는 것은 귀찮은 일이다. 자바8 에서는 다음처럼 새로운 개념을 이용해서 코드를 구현할 수 있다. 
`filterApples(inventory, (Apple a) -> “green”.equals(a.getColor());`<br>
`filterApples(inventory, (Apple a) -> a.getWeight() > 150);`<br>
`filterApples(inventory, (Apple a) -> a.getWeight() < 80 || “brown”.equals(a.getColor());`<br>
즉, 한 번만 사용할 메서드는 따로 정의를 구현할 필요가 없다. 하지만 람다가 몇 줄 이상으로 길어진다면(복잡한 동작을 수행하는 상황) 익명 람다보다는 코드가 수행하는 일을 잘 설명하는 이름을 가진 메서드를 정의하고 메서드 레퍼런스를 활용하는 것이 바람직하다. **코드의 명확성이 우선시 되어야 한다.**<br>

###스트림
다음은 리스트에서 고가의 거래(Transcation)만 필터링한 다음에 통화로 결과를 그룹화하는 코드다. 
```
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>(); // 그룹화된 트랜잭션을 더할 Map 생성

for (Transaction transaction : transactions){ // 트랜잭션의 리스트를 반복
	if (transaction.getPrice() > 1000){ // 고가의 트랜잭션을 필터링
		Currency currency = transaction.getCurrency(); // 트랜잭션의 통화 추출
		List<Transcation> transactionsForCurrency = transactionsByCurrencies.get(currency); 
		if (transactionsForCurrency == null){ // 현재 통화의 그룹화된 맵에 항목이 없으면 새로 만든다.
			transactionsForCurrency = new ArrayList<>();
			transactionsByCurrencies.put(currency, transactionsForCurrency);
		}
		transactionsForCurrency.add(transaction); // 현재 탐색된 트랜잭션을 같은 통화의 트랜잭션 리스트에 추가한다. 
	}
}
```
위 예제 코드에는 중첩된 제어 흐름 문장이 많아서 코드를 한 번에 이해하기 어렵다. 스트림 API를 이요하면 다음처럼 문제를 해결할 수 있다. 
```
import static java.util.stream.Collections.toList;

Map<Currency, List<Transaction>> transactionByCurrencies = 
	transactions.stream()
           		   .filter((Transaction t) -> t.getPrice() > 1000) //고가의 트랜잭션 필터링
			   .collect(groupingBy(Transaction::getCurrency);	
```
컬렉션에서는 반복 과정을 직접 처리해야 했다. 이런 방식의 반복을 외부 반복이라고 한다. 반면 스트림 API를 이용하면 루프를 신경쓸 필요가 없다. 라이브러리 내부에서 모든 데이터가 처리된다. 이와 같은 반복을 내부 반복이라고 한다.<br>

컬렉션은 어떻게 데이터를 저장하고 접근할지에 중점을 두는 반면 스트림은 데이터에 어떤 계산을 할 것인지 묘사하는 것에 중점을 둔다는 점을 기억하자. 스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다는 것이 핵심이다. <br>

**컬렉션을 필터링할 수 있는 가장 빠른 방법은 컬렉션을 스트림으로 바꾸고, 병렬로 처리한 다음에, 리스트로 다시 복원하는 것이다.**<br>

###디폴트 메서드 
디폴트 메서드는 특정 프로그램을 구현하는 데 도움을 주는 기능이 아니라 미래에 프로그램이 쉽게 변화할 수 있는 환경을 제공하는 기능이다. <br> 

##2장 - 동작 파라미터화 코드 전달하기
동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. **동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다. ** 이 코드 블록은 나중에 프로그램에서 호출한다. 즉, 코드 블록의 실행은 나중으로 미뤄진다. 결과적으로 코드 블록에 따라 메서드의 동작이 파라미터화된다. <br>

**동작 파라미터화**<br>
```
public interface ApplePredicate{
	boolean test (Apple apple);
}
```
선택 조건을 결정하는 인터페이스이다. 이와 같은 동작을 프레디케이트(불린을 반환하는 함수)라고 한다. <br>
다음 예제처럼 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있다. 
```
//무거운 사과만 선택
public class AppleHeavyWeightPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return apple.getWeight() > 150;
	}
}
//녹색 사과만 선택
public class AppleGreenColorPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return “green”.equals(apple.getColor());
	}
}

// 템플릿 부분
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if (p.test(apple)){
			result.add(apple);
		}
	}
	return result;
}

```
하지만 익명 클래스를 사용해도 반복되어 지저분한 코드는 여전히 많고 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다. 

**람다 표현식 사용**<br>
자바8의 람다 표현식을 이용해서 간단히 재구현할 수 있다. <br>
`List<Apple> result = filterApples(inventory, (Apple apple) -> “red”.equals(apple.getColor()));`<br>
![](parameterization.jpg)

**리스트 형식으로 추상화**<br>
Apple 이외의 다양한 물건에서 필터링이 작동하도록 리스트 형식을 추상화할 수 있다. 
```
public interface Predicate<T>{
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
	List<T> result = new ArrayList<>();
	for (T e : list){
		if (p.test(e)){
			result.add(e); 
		}
	}
	return result; 
}
```
이제 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드를 사용할 수 있다. 
`List<Apple> redApples = filter(inventory, (Apple apple) -> “red”.equals(apple.getColor());`<br>
`List<String> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);`<br>

**실전 예제 - Comparator로 정렬하기**<br>
자바8의 List에는 sort 메서드가 포함되어 있다.(물론 Collections.sort도 존재) 다음과 같은 인터페이스를 갖는 java.util.Comparator 객체를 이용해서 sort의 동작을 파라미터화 할 수 있다. 
```
//java.uitl.Comparator
public interface Comparator<T>{
	public int compare(T o1, T o2);
}
```
Comparator를 구현해서 sort 메서드의 동작을 다양화할 수 있다. 예를 들어 익명 클래스를 이용해서 무게가 적은 순으로 목록에서 사과를 정렬할 수 있다.
```
inventory.sort(new Comparator<Apple>{
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
});
```
람다 표현식을 이용하면 다음처럼 간단하게 코드를 구현할 수 있다.
```
inventory.sort(
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
##3장 - 람다 표현식
###람다란 무엇인가
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한것이라고 할 수 있다. 
```
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
};
```
```
Comparator<Apple> byWeight =
        (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
람다는 세 부분으로 이루어진다. <br>
**파라미터 리스트** :  Comparator의 compare 메서드의 파라미터(두개의 사과).<br>
**화살표** : 화살표(->)는 람다의 파라미터 리스트와 바디를 구분한다.<br>
**람다의 바디** : 두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.<br>

**자바8의 유효한 5가지 람다 표현식**<br>
`(String s) -> s.length()` <br>
첫 번째 람다 표현식은 String 형식의 파라미터 하나를 가지며 int를 반환한다. 람다 표현식에는 return이 함축되어 있으므로 return 문을 명시적으로 사용하지 않아도 된다. <br>

`(Apple a) -> a.getWeight() > 150` <br>
두번째 람다 표현식은 Apple 형식의 파라미터를 가지며 boolean을 반환한다. <br>

```
(int x, int y) -> {
	System.out.println(“Result:”);
	System.out.println(x+y);
```
세 번째 람다 표현식은 int 형식의 파라미터 두 개를 가지며 리턴값이 없다(void 리턴). 이 예제에서 볼 수 있듯이 람다 표현식은 여러 행의 문장을 포함할 수 있다. <br>

`() -> 42` <br>
네 번째 람다 표현식은 파라미터가 없으며 int를 반환한다. <br>
`(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());`<br>
다섯 번째 람다 표현식은 Apple 형식의 파라미터 두 개를 가지며 int를 반환한다. 
```
//cf) 유효하지 않은 람다 표현식
(Integer i ) -> return “Alan” + i; 
(String s) -> {“Iron Man”;}

//return은 흐름 제어문이다. { } 안에 있어야 한다. 
(Integer i ) -> { return “Alan” + i }; 

//“Iron Man”은 구문(statement)이 아니라 표현식(expression)이다. 
(String s) -> “Iron Man” 또는 (String s) -> { return “Iron Man” }
```