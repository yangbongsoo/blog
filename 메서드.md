# 메서드
###규칙38 : 인자의 유효성을 검사하라 
메서드나 생성자를 구현할 때는 받을 수 있는 인자에 제한이 있는지 따져봐야 한다. 제한이 있다면 그 사실을 문서에 남기고 메서드 앞부분에서 검사하도록 해야 한다. **인자 유효성을 검사하지 않았을 때 최고로 심각한 유형의 문제는, 메서드가 정상적으로 반환값을 내기는 하지만 어떤 객체의 상태가 비정상적으로 바뀌는 경우다.** 그러면 나중에 해당 메서드와는 아무 상관도 없는 부분에서 오류가 뜨는데, 그 시간과 위치는 프로그램을 실행할 때마다 바뀐다.

public 메서드라면, 인자 유효성이 위반되었을 경우에 발생하는 예외를 Javadoc의 @throws 태그를 사용해서 문서화하라 보통 IllegalArgumentException, IndexOutOfBoundsException, NullPointerException이 이용된다.
public이 아닌 메서드는 일반적으로 인자 유효성을 검사할 때 확증문(assertion)을 이용한다. 
```java
// 재귀적으로 정렬하는 private 도움 함수 
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >=0 && length <= a.length - offset;
	… // 계산 수행 
}
```
확증문은 클라이언트가 패키지를 어떻게 이용하건 확증 조건은 항상 참이 되어야 한다고 주장하는 것이다. 통상적인 유효성검사와는 달리, 확증문은 확증 조건이 만족되지 않으면 AssertionError를 낸다. **또한 통상의 유효성 검사와는 달리, 활성화되지 않은 확증문은 실행되지 않으므로 비용이 0이다. ** 확증문을 활성화시키려면 java 인터프리터에 -ea(또는 -enableassertions) 옵션을 주어야 한다.

생성자는 클래스 불변식을 위반하는 객체가 만들어지는 것을 막으려면, 생성자에 전달되는 인자의 유효성을 반드시 검사해야 한다. cf) 불변 클래스는 설계, 구현이 쉽다. 객체의 상태를 변경하는 어떤 메서드도 제공안한다. 모든 필드는 fianl. 상속 불가. private 선언.

이번 절에서 다룬 내용을 잘못 받아들여 “인자에 제약을 두는 것은 바람직하다”고 믿어버리면 곤란하다. 메서드는 가능하면 일반적으로 적용될 수 있도록 설계해야 한다. 메서드가 받을 수 있는 읹에 제약이 적으면 적을수록 더 좋다. 

###규칙39 : 필요하다면 방어적 복사본을 만들라 
여러분이 만드는 클래스의 클라이언트가 불변식을 망가뜨리기 위해 최선을 다할 것이라는 가정하에, 방어적으로 프로그래밍해야 한다. 아래의 클래스는 기간을 나타내는 객체에 대한 변경 불가능 클래스다. 
```java
//변경 불가능성이 보장되지 않는 변경 불가능 클래스 
public fianl class Period{
	private final Date start; // 기간의 시작 지점
	private final Date end;  // 기간의 끝 지점. start보다 작은 값일 수 없다. 

	//@throws IllegalArgumentException start가 end보다 뒤면 발생
	//@throws NullPointerException start나 end가 null이면 발생 

	public Period(Date start, Date end){
		if(start.compareTo(end) > 0)
			throw new IllegalArgumentException(start + “after” + end);
		this.start = start;
		this.end = end;
	}

	public Date start(){
		return start;
	}
 
	public Date end(){
		return end;
	}

	… // 이하 생략
	
}
```
얼핏 변경이 불가능한 것으로 보이고, 기간 시작점이 기간 끝점 이후일 수 없다는 불변식도 만족되는 것처럼 보인다. 하지만 Date가 변경 가능 클래스라는 점을 이용하면 불변식을 깨트릴 수 있다.
```java
// Period 객체의 내부 구조를 공격
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 변경 ! 
```
**따라서 Period 객체의 내부를 보호하려면 생성자로 전달되는 변경 가능 객체를 반드시 방어적으로 복사해서 그 복사본을 Period 객체의 컴포넌트로 이용해야 한다. **
```java
// 수정된 생성자 - 인자를 방어적으로 복사함
public Period(Date start, Date end){
	this.start = new Date(start.getTime());
	this.end = new Date(end.getTime());

	if(this.start.compareTo(this.end) > 0)
		throw new IllegalArgumentException(this.start + “after” + this.end);
	
}
```
인자의 유효성을 검사하기 전에 방어적 복사본을 만들었다는 것에 유의하자. 유효성 검사는 복사본에 대해서 시행한다. 자연스러워 보이지 않을지도 모르나, 필요한 절차다. 인자를 검사한 직후 복사본이 만들어지기 직전까지의 시간, 그러니까 “취약 구간” 동안에 다른 스레드가 인자를 변경해 버리는 일을 막기 위한 것이다. (TICTOU 공격)

방어적 복사본을 만들 때 Date의 clone 메서드를 이용하지 않았다. Date 클래스는 final 클래스가 아니므로, clone 메서드가 반드시 java.util.Date 객체를 반환할 거라는 보장이 없다. 공격을 위해 특별히 설계된 하위 클래스 객체가 반환될 수도 있다. **이런 공격을 막으려면 인자로 전달된 객체의 자료형이 제 3자가 계승할 수 있는 자료형일 경우, 방어적 복사본을 만들 때  clone을 사용하지 않도록 해야 한다. **

**접근자를 통한 공격**
위의 생성자를 사용하면 생성자 인자를 통한 공격은 막을 수 있으나 접근자를 통한 공격은 막을 수 없다. 접근자를 호출하여 얻은 객체를 통해 Period 객체 내부를 변경할 수 있기 때문이다.
```java
// Period 객체 내부를 노린 두 번째 공격 형태
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부를 변경 ! 
```
이런 공격을 막으려면 변경 가능 내부 필드에 대한 방어적 복사본을 반환하도록 접근자를 수정해야 한다. 
```java
// 수정된 접근자 - 내부 필드의 방어적 복사본 생성 
public Date start(){
	return new Date(start.getTime());
}

public Date end(){
	return new Date(end.getTime());
}
```
이렇게 수정하고 나면 Period는 진정한 변경 불가능 클래스가 된다. 객체 안에 확실히 캡슐화된 필드가 된 것이다.

방어적 복사는 변경 불가능 클래스에서만 쓰이는 기법이 아니다. **클라이언트가 제공한 객체를 내부 자료 구조에 반영하는 생성자나 메서드에는 사용 가능하다. ** 예를 들어, 클라이언트가 제공한 객체 참조를 내부 Set 객체의 요소로 추가해야 하거나 내부 Map 객체의 키로 써야 하는 경우, 삽입된 객체가 나중에 변경된다면 집합이나 맵의 불변식은 깨지고 말 것이다.

방어적 복사본을 만들도록 하면 성능에서 손해를 보기 때문에, 적절치 않을 때도 있다. 클라이언트가 같은 패키지 안에 있다거나 하는 이유로, 클라이언트가 객체의 내부 상태를 변경하려 하지 않는다는 것이 확실하다면 방어적 복사본은 만들지 않아도 될 것이다. **여기서 배워야 할 진짜 교훈은, 객체의 컴포넌트로는 가능하다면 변경 불가능 객체를 사용해야 한다는 것이다.** 그래야 방어적 복사본에 대해서는 신경 쓸 필요가 없어지기 때문이다.

###규칙40 : 메서드 시그니처는 신중하게 설계해라 
**메서드 이름은 신중하게 고르라.**
**편의 메서드를 제공하는 데 너무 열 올리지 마라. **클래스에 메서드가 너무 많으면 학습, 사용, 테스트, 유지보수 등의 모든 측면에서 어렵다.
**인수 리스트(parameter list)를 길게 만들지 마라. **4개 이하가 되도록 애쓰라. 긴 인자 리스트를 짧게 줄이는 방법으로는 첫째, 여러 메서드로 나누는 방법이 있고 둘째, 도움 클래스를 만들어 인자들을 그룹별로 나누는 것이다. 보통 이 도움 클래스들은 static 멤버 클래스다(자주 등장하는 일련의 인자들이 어떤 별도 개체를 나타낼 때 쓰면 좋다). 셋째, 빌더 패턴이 있다.
**인자의 자료형으로는 클래스보다 인터페이스가 좋다.**
**인자 자료형으로 boolean을 쓰는 것보다는, 원소가 2개인 enum 자료형을 쓰는 것이 낫다.** 

###규칙 41 : 오버로딩할 때는 주의하라 
아래의 프로그램 목적은 컬렉션을 종류별로(집합이냐, 리스트냐, 아니면 다른 종류의 컬렉션이냐) 분류하는 것이다.
```java
//잘못된 프로그램
public class CollectionClassifier {
    public static String classify(Set<?> s){
        return "Set";
    }

    public static String classify(List<?> lst){
        return "List";
    }

    public static String classify(Collection<?> lst){
        return "Unknown Collection";
    }

    public static void main(String[] args){
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for(Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
``` 
이 프로그램이 Set, List, Unkwon Collection을 순서대로 출력하지 않을까 기대하겠지만 실제로는 Unknown Collection을 세 번 출력한다. 그 이유는 **classify 메서드가 오버로딩되어 있으며, 오버로딩된 메서드 가운데 어떤 것이 호출될지는 컴파일 시점에 결정되기 때문이다. **루프가 세 번 도는 동안, 인자의 컴파일 시점 자료형은 전부 `Collection<?>`으로 동일하다. 각 인자의 실행시점 자료형(runtime type)은 전부 다르지만, 선택 과정에는 영향을 끼치지 못한다. 인자의 컴파일 시점 자료형이 `Collection<?>`이므로 호출되는 것은 항상 `classify(Collection<?>)` 메서드다.

이 예제 프로그램은 직관과는 반대로 동작한다. **오버로딩된 메서드는 정적(static)으로 선택되지만, 재정의된 메서드는 동적(dynamic)으로 선택되기 때문이다.** 재정의된 메서드의 경우 선택 기준은 메서드 호출 대상 객체의 자료형이다. 객체 자료형에 따라 실행 도중에 결정되는 것이다. 그렇다면 재정의된 메서드란 무엇인가? 상위 클래스에 선언된 메서드와 같은 시그니처를 갖는 하위 클래스 메서드가 재정의된 메서드다. 하위 클래스에서 재정의한 메서드를 하위 클래스 객체에 대해 호출하면, 해당 객체의 컴파일 시점 자료형과는 상관없이, 항상 하위 클래스의 재정의 메서드가 호출된다. 
```java
class Wine {
    String name() { return "wine";}
}

class SparklingWine extends Wine {
    @Override String name() { return "sparklingWine"; }
}

class Champagne extends SparklingWine{
    @Override String name() { return "champagne"; }
}

public class Overriding {
    public static void main(String[] args) {
        Wine[] wines = {
                new Wine(), new SparklingWine(), new Champagne()
        };

        for(Wine wine : wines)
            System.out.println(wine.name());
    }
}
```
name 메서드는 Wine 클래스에 선언되어 있고, sparklingWine과 Champagne은 그 메서드를 재정의한다. 기대대로 위의 프로그램은 순서대로 출력한다. 순환문의 매 루프마다 객체의 컴파일 시점 자료형은 항상 Wine이었는데도 말이다. **재정의 메서드 가운데 하나를 선택할 때 객체의 컴파일 시점 자료형은 영향을 주지 못한다. 오버로딩에서는 반대로 실행시점 자료형이 아무 영향도 주지 못한다.** 실행될 메서드는 컴파일 시에, 인자의 컴파일 시점 자료형만을 근거로 결정된다.

오버로딩은 직관적인 예측에 반하므로 혼란스럽다. 따라서 오버로딩을 사용할 때는 혼란스럽지 않게 주의해야 한다. 혼란을 피하는 안전하고 보수적인 전략은, 같은 수의 인자를 갖는 두 개의 오버로딩 메서드를 API에 포함시키지 않는 것이다. 예를 들어 ObjectOutputStream의 경우를 생각해 보자. 이 메서드들은 writeBoolean(boolean), writeInt(int), writeLong(long) 같이 정의되어 있다. 이런 작명 패턴을 따르면 오버로딩에 비해 어떤 점이 좋을까? 각 메서드에 대응되는 read 메서드를 정의할 수 있게 된다. (readBoolean(), readInt(), readLong() 등을 정의할 수 있게 된다).

하지만 생성자에는 다른 이름을 사용할 수 없다. 생성자가 많다면, 그 생성자들은 항상 오버로딩된다. 그게 문제라면 생성자 대신 정적 팩터리 메서드를 사용하는 옵션을 사용할 수도 있다. 하지만 같은 수의 인자를 받는 오버로딩 메서드가 많더라도, 어떤 오버로딩 메서드가 주어진 인자 집합을 처리할 것인지가 분명히 결정된다면 프로그래머는 혼란을 겪지 않을 것이다. 그 조건은, 두 개의 오버로딩 메서드를 비교했을 때 그 형식 인자 가운데 적어도 하나가 “확실히 다르다”면 만족된다. 확실히 다르다라는 것은 두 자료형을 서로 형변환 할 수 없다면 확실히 다른 것이다. 예를 들어 ArrayList에는 int를 받는 생성자와 Collection을 인자로 받는 생성자가 있다. 어떤 상황에서라도 이 두 생성자 간에 혼란의 여지가 있으리라고는 보기 어렵다.

**remove(E) vs remove(int)**
자바 1.5 이전에는 모든 기본 자료형은 참조 자료형과는 확실히 달랐다. 하지만 자동 객체화(autoboxing)라는 기능이 도입된 후 이제는 “확실히 다르다”라고 말할 수 없게 됐다. 
```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for(int i = -3 ; i < 3; i++){
            set.add(i);
            list.add(i);
        }

        for(int i =0; i< 3; i++){
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```
결과를 [-3, -2, -1] [-3, -2, -1] 이렇게 기대하지만 실상은 그렇지 않다. [-3, -2, -1] [-2, 0, 2] 가 출력된다. set.remove(i)는 인자의 값을 가진 모든 원소가 제거된다. 하지만 list.remove(i)는 해당 i번째 요소의 원소값을 지우게 된다. 그래서 해결을 하려면 Integer로 형변환하여 올바른 오버로딩을 하거나 Integer.valueOf를 적용해야 한다. 
```java
for(int i =0; i< 3; i++){
            set.remove(i);
            // 인자로 들어온 값을 지우고 싶기 때문에 remove(int)가 아닌 remove(E)형태가 되야 한다.
            list.remove(Integer.valueOf(i)); // 아니면 remove((Integer) i);
}
 
```
이런 일이 발생하는 원인은, `List<E>` 인터페이스에 remove(E)와 remove(int)라는 오버로딩 메서드 두 개가 존재하기 때문이다.(remove(E)는 인자로 들어온 값을 지우는 메서드, remove(int)는 인자 position의 값을 지우는 메서드) 제네릭이 도입된 자바 1.5 이전에는 List 인터페이스에 remove(E) 대신 remove(Object)가 있었다. Object와 int는 완전히 다른 자료형이므로 문제가 될 것이 없었다. 하지만 제네릭과 자동 객체화(autoboxing)가 도입되면서, E와 int는 더 이상 완전히 다르다고 말할 수 없게 되었다.

인자 개수가 같은 오버로딩 메서드를 추가하는 것은 일반적으로 피해야 한다. 하지만 특히 생성자에 대해서라면 이 충고를 따를 수 없을 지도 모른다. 그럴 때는, 형변환만 추가하면 같은 인자 집합으로 여러 오버로딩 메서드를 호출할 수 있는 상황은 피하는 것이 좋다. 

###규칙 42 : varargs는 신중히 사용하라 
자바 1.5부터 추가된 varargs 메서드는 가변 인자 메서드라고 부른다. 이 메서드는 지정된 자료형의 인자를 0개 이상 받을 수 있다. 동작 원리는 이렇다. 우선 클라이언트에서 전달한 인자 수에 맞는 배열이 자동 생성되고, 모든 인자가 해당 배열에 대입된다. 그리고 마지막으로 해당 배열이 메서드에 인자로 전달된다.
```java
//varargs의 간단한 사용 예 
static int sum(int… args) {
	int sum = 0;
	for ( int arg : args )
		sum += arg;
	return sum; 
}
```
그런데 때로는 0 이상이 아니라, 하나 이상의 인자가 필요할 때가 있다. 예를 들어 주어진 int 인자 가운데 최소치를 구해야 한다고 생각해 보자. 아래의 함수는 인자 없이 호출될 수 있다고 생각하면 깔끔하게 구현되지 않는다. 실행시점에 배열 길이를 검사해야만 한다.
```java
// 하나 이상의 인자를 받아야 하는 varargs 메서드를 잘못 구현한 사례
static int min(int … args){
	if(args.length == 0)
		throw new IllegalArgumentException(“Too few arguments”);
	int min = args[0];
	for(int i = 1; i < args.length; i++)
		if(args[i] < min )
			min = args[i];
	return min;
}
```
그러나 이 방법에는 몇 가지 문제가 있다. 클라이언트가 인자 없이 메서드를 호출하는 것이 가능할 뿐 아니라, 컴파일 시점이 아니라 실행 도중에 오류가 난다는 것이다. 또 한 가지 문제는 보기 흉한 코드라는 것이다. args의 유효성을 검사하는 코드를 명시적으로 넣어야 하고, min을 Integer.MAX_VALUE로 초기화하지 않는 한 for-each 문을 사용할 수도 없다.

다행히 더 좋은 방법이 있다. 메서드가 인자를 두 개 받도록 선언하는 것이다. 하나는 지정된 자료형을 갖는 일반 인자고, 다른 하나는 같은 자료형의 varargs 인자다. 이 해법은 앞서 살펴본 방법의 모든 문제를 해결한다. 
```java
// 하나 이상의 인자를 받는 varargs 메서드를 제대로 구현한 사례 
static int min(int firstArg, int … remainingArgs){
	int min = firstArg;
	for (int arg : remainingArgs) 
		if(arg < min)
			min = arg;
	return min; 
}
```
이 예제로 알 수 있듯, varargs는 임의 개수의 인자를 처리하는 메서드를 만들어야 할 때 효과적이다. varargs가 추가된 것은 자바 1.5부터 플랫폼에 추가된 printf 메서드와, varargs를 이용할 수 있도록 개선된 핵심 리플렉션 기능 때문이다. printf와 리플렉션은 varargs를 엄청나게 많이 이용한다.

**Arrays.asList**
이 메서드는 원래 여러 인자를 하나의 리스트로 합칠 목적으로 설계된 것이 아니었다. 그래서 varargs가 플랫폼에 추가되었을 때, 아래와 같은 코드를 지원할 수 있도록 Arrays.asList를 수정한 것은 좋은 생각 같았다. `List<String> homophones = Arrays.asList(“to”, “too”, “two”);` 하지만 배열에 직접 toString을 호출하면 쓸모없는 문자열이 출력된다. 
```java
int[] myArr = {11, 22};
System.out.println(Arrays.asList(myArr)); // [[I@64889c4e]
```

이 숙어는 객체 참조 자료형에 대해서만 동작했고 기본 자료형 값의 배열에 적용하면 원하는 것과 거리가 멀고 쓸모없는 값이 나온다. (자바 1.4에서는 컴파일 조차 안됐지만 1.5에서 Arrays.asList를 varargs 메서드로 바꾼 덕분에 오류나 경고 없이 컴파일 된다.)
```java
int[] digits = {3,1,2,3,4,1,2,6,7,5};
System.out.println(Arrays.asList(digits));  // [[I@4678f83a]

Integer[] digits = {3,1,2,3,4,1,2,6,7,5};
System.out.println(Arrays.asList(digits));  // [3, 1, 2, 3, 4, 1, 2, 6, 7, 5]
```
기본 자료형에 이런 결과가 나오는 이유에 대해서 알아보자. Arrays.asList 메서드는 객체 참조를 모아 배열로 만드는데, 그 결과로 int 배열 digits에 대한 참조가 담긴 길이 1짜리 배열, 즉 배열의 배열이 만들어진다. `List<int[]>` 객체가 만들어지는 것이다. 이 리스트에 toString을 호출하면 다시 그 내부의 원소(int 배열)의 toString 메서드가 호출되는데, 방금 본 이상한 문자열은 그렇게 만들어지는 것이다.  

**그나마 다행인 것은 Arrays.asList를 사용하여 배열을 문자열로 변환하는 숙어는 이제 폐기되었다는 것이다. 뒤이어 나온 숙어는 좀 더 안정적이다. Arrays 클래스에는 어떤 자료형의 배열이라도 문자열로 변환할 수 있도록 설계된 Arrays.toString 메서드가 구비되었다(varargs 메서드가 아니다). Arrays.asList 대신 Arrays.toString을 사용하도록 프로그램을 고치면 원하는 결과를 얻을 수 있다. **
```java
// 배열을 출력하는 올바른 방법
System.out.println(Arrays.toString(myArr));
```
varargs는 정말로 임의 개수의 인자를 처리할 수 있는 메서드를 만들어야 할 때만 사용하라. 의심스런 메서드들은 아무 인자 리스트나 받을 수 있는 메서드들이다. 
```java
ReturnType1 suspect1 (Object ... args) { }
<T> ReturnType2 suspect2 (T ... args) { }
```

###규칙 43 : null 대신 빈 배열이나 컬렉션을 반환하라
아래와 같이 정의된 메서드는 어렵지 않게 만날 수 있다.
```java
private final List<Cheese> cheeseInStock = …;
//@return 재고가 남은 모든 치즈를 반환. 치즈가 남지 않았을 때는 null을 반환
public Cheese[] getCheeses() {
	if (cheesesInStock.size() == 0)
		return null;
	...
}
```
그런데 치즈 재고가 없는 상황을 특별하게 처리하도록 강제하는 코드는 바람직하지 않다. 클라이언트 입장에서는 null이 반환될 때를 대비한 코드를 만들어야 하기 때문이다. 아래의 예를 보자.
```java
Cheese[] cheeses = shop.getCheeses();
if( cheeses != null && Arrays.asList(cheeses).contains(Cheese.STILTON)
	System.out.println(“Jolly good, just the thing.”);
```
null이 반환되지 않는다면 아래와 같이 할 수 있었을 것이다. 
```java
if(Arrays.asList(cheeses).contains(Cheese.STILTON)
	System.out.println(“Jolly good, just the thing.”);
```
빈배열이나 컬렉션을 반환하는 대신에 null을 반환하는 메서드를 사용하면 이런 상황을 겪게 된다. 이런 메서드는 오류를 쉽게 유발한다. 클라이언트가 null 처리를 잊어버릴 수 있기 때문이다.

배열 할당 비용을 피할 수 있으니 null을 반환해야 바람직한 것 아니냐는 주장도 있을 수 있으나, 이 주장은 두 가지 측면에서 틀렸다. 프로파일링 결과로 해당 메서드가 성능 저하의 주범이라는 것이 밝혀지지 않는 한, 그런 수준까지 성능 걱정을 하는 것은 바람직하지 않다는 것이 첫 번째다. **두 번째는, 길이가 0인 배열은 변경이 불가능 하므로 아무 제약없이 재사용 할 수 있다는 것이다.** 사실, 컬렉션에 들어 있는 원소를 배열에 퍼 나를 때 사용하는 표준적 숙어가 바로 이 규범을 따른다.

```java
// 컬렉션에서 배열을 만들어 반환하는 올바른 방법
private final List<Cheese> cheesesInStock = …;
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
// @return 재고가 남은 모든 치즈 목록을 배열로 만들어 반환 
public Cheese[] getCheeses() { 
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
위 숙어에서 toArray 메서드에 전달되는 빈 배열 상수는 반환값의 자료형을 명시하는 구실을 한다. **보통  toArray는 반환되는 원소가 담길 배열을 스스로 할당하는데, 컬렉션이 비어 있는 경우에는 인자로 주어진 빈 배열을 쓴다.** 그리고 Collection.toArray(T[])의 명세를 보면, 인자로 주어진 배열이 컬렉션의 모든 원소를 담을 정도로 큰 경우에는 해당 배열을 반환값으로 사용한다고 되어 있다. 따라서 위의 숙어대로 하면 빈 배열은 절대로 자동 할당되지 않는다. 
```java
// ArrayList의 toArray 구현 예 
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
마찬가지로, 컬렉션을 반환하는 메서드도 빈 컬렉션을 반환해야 할 때마다 동일한 변경 불가능 빈 객체를 반환하도록 구현할 수 있다. Collections.emptySet, emptyList, emptyMap 메서드가 그런 용도로 사용된다. 다음 예제를 보자. 
```java
// 컬렉션 복사본을 반환하는 올바른 방법
public List<Cheese> getCheeseList() {
	if (cheesesInStock.isEmpty())
		return Collections.emptyList(); //언제나 같은 리스트 반환
	else
		return new ArrayList<Cheese>(cheesesInStock);
}
```

###규칙44 : 모든 API 요소에 문서화 주석을 달라 
좋은 API 문서를 만들려면 API에 포함된 모든 클래스, 인터페이스, 생성자, 메서드, 그리고 필드 선언에 문서화 주석을 달아야 한다. 직렬화가 가능한 클래스라면 직렬화 형식도 밝혀야 한다.

**메서드 주석**
메서드에 대한 무서화 주석은 메서드와 클라이언트 사이의 규약을 간명하게 설명해야 한다. 계승을 위해 설계된 메서드가 아니라면 **메서드가 무엇을 하는지를 설명해야지 어떻게 그 일을 하는지를 설명해서는 안된다.** 아울러 문서화 주석에는 해당 메서드의 모든 선행조건과 후행조건을 나열해야 한다. 선행조건은 클라이언트가 메서드를 호출하려면 반드시 참이 되어야 하는 조건들이다. 후행조건은 메서드 실행이 성공적으로 끝난 다음에 만족되어야 하는 조건들이다. 보통 선행조건은 무점검 예외(unchecked exception)에 대한 @throws 태그를 통해 암묵적으로 기술한다. 관계된 인자의 @param 태그를 통해 명시할 수도 있다.

선행조건과 후행조건 외에도, 메서드는 부작용(side effect)에 대해서도 문서화 해야한다. 예를 들어 어떤 메서드가 후면 스레드(background thread)를 실행한다면 문서에는 그 사실이 명시되어야 한다.

관습상 @param, @return, @throws 태그 다음에 오는 구나 절에는 마침표를 찍지 않는다. 
```java
/**
 * Returns the element at the specified position in this list
 * 
 * <p>This method is <i>not</i> guaranteed to run in constant time. In some implementations it may run in time proportional to the element position.
 *
 * @param index index of element to return; must be
 *              non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int indxe);
```
주석에 HTML 태그`<p> <i>`가 사용되었다는 것에 유의하자. Javadoc 유틸리티는 문서화 주석을 HTML 문서로 변환한다. @throws 절에 폼함된 코드 주변에 {@code} 태그가 사용된 것도 유의하자. 이 태그는 두 가지 일을 한다. 첫 번째는 해당 코드가 코드 서체로 표시되도록 하는 것이고 두 번째는 그 안에 포함된 모든 HTML 마크업이나 Javadoc 태그가 위력을 발휘하지 못하도록 하는 것이다. {@literal} 태그를 사용하면 그 태그 안에 포함된 HTML 마크업이나 Javadoc 태그는 전부 단순 문자로 취급된다. {@code} 태그와 유사하지만, 코드 서체로 표시되지는 않는다는 차이가 있다.

모든 문서화 주석의 첫 번째 “문장”은, 해당 주석에 담긴 내용을 요약한 것이다. 혼란을 막기 위해, 클래스나 인터페이스의 멤버나 생성자들 가운데 요약문이 같은 것은 없어야 한다. 오버로딩을 진행할 때는 특히 주의하라. 같은 요약문을 쓰는 것이 자연스러울 때가 종종 있어서다(하지만 문서화 주석의 경우, 동일한 첫 문장은 곤란하다).

문서화 주석의 요약문은 반드시 완벽한 문장일 필요가 없다. 메서드나 생성자의 경우, 요약문은 메서드가 무슨 일을 하는지 기술하는 완전한 동사구여야 한다. 클래스나 인터페이스의 요약문은 해당 클래스나 인터페이스로 만들어진 객체가 무엇을 나타내는지를 표현하는 명사구여야 한다. 필드의 요약문은 필드가 나타내는 것이 무엇인지를 설명하는 명사구여야 한다.

**제네릭, enum, 애노테이션 문서화 주석**
제네릭 자료형이나 메서드에 주석을 달 때는 모든 자료형 인자들을 설명해야 한다. 
```java
/**
 * An object that maps keys to values. A map cannot contain
 * duplicate keys; each key can map to at most one value.
 *
 * (중간 생략)
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values 
 */
public interface MyMap<K,V> {
}
```
enum 자료형에 주석을 달 때는 자료형이나 public 메서드뿐 아니라 상수 각각에도 주석을 달아 주어야 한다.
```java
/**
 * 교향악단에서 쓰이는 악기 종류
 */
public enum OrchestraSection {
    /** 플루트, 클라리넷, 오보에 관한 목관악기.*/ 
    WOODWIND,

    /** 프렌치 혼이나 트럼펫 같은 금관악기. */
    BRASS,

    /** 팀파니나 심벌즈 같은 타악기. */
    PERCUSSION,

    /** 바이올린이나 첼로 같은 현악기. */
    STRING;
}
```
애노테이션 자료형에 주석을 달 때는 자료형뿐 아니라 모든 멤버에도 주석을 달아야 한다. 멤버에는 마치 필드인 것처럼 명사구 주석을 달라. 자료형 요약문에는 동사구를 써서, 언제 이 자료형을 애노테이션으로 붙어야 하는지 설명하라. 
```java
/**
 * 지정된 예외를 반드시 발생시켜야 하는 테스트 메서드임을 명시. 
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{

	/**
         * 애노테이션이 붙은 테스트 메서드가 테스트를 통과하기 위해
         * 반드시 발생시켜야 하는 예외. (이 Class 객체가 나타내는 자료형의 
         * 하위 자료형이기만 하면 어떤 예외든 상관없다.)
         */
	Class<? extends Throwable> value();
}
```