# 메서드
###규칙38 : 인자의 유효성을 검사하라 
메서드나 생성자를 구현할 때는 받을 수 있는 인자에 제한이 있는지 따져봐야 한다. 제한이 있다면 그 사실을 문서에 남기고 메서드 앞부분에서 검사하도록 해야 한다. **인자 유효성을 검사하지 않았을 때 최고로 심각한 유형의 문제는, 메서드가 정상적으로 반환값을 내기는 하지만 어떤 객체의 상태가 비정상적으로 바뀌는 경우다.** 그러면 나중에 해당 메서드와는 아무 상관도 없는 부분에서 오류가 뜨는데, 그 시간과 위치는 프로그램을 실행할 때마다 바뀐다. <br>

public 메서드라면, 인자 유효성이 위반되었을 경우에 발생하는 예외를 Javadoc의 @throws 태그를 사용해서 문서화하라 보통 IllegalArgumentException, IndexOutOfBoundsException, NullPointerException이 이용된다. <br>
public이 아닌 메서드는 일반적으로 인자 유효성을 검사할 때 확증문(assertion)을 이용한다. 
```
// 재귀적으로 정렬하는 private 도움 함수 
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >=0 && length <= a.length - offset;
	… // 계산 수행 
}
```
확증문은 클라이언트가 패키지를 어떻게 이용하건 확증 조건은 항상 참이 되어야 한다고 주장하는 것이다. 통상적인 유효성검사와는 달리, 확증문은 확증 조건이 만족되지 않으면 AssertionError를 낸다. **또한 통상의 유효성 검사와는 달리, 활성화되지 않은 확증문은 실행되지 않으므로 비용이 0이다. ** 확증문을 활성화시키려면 java 인터프리터에 -ea(또는 -enableassertions) 옵션을 주어야 한다. <br>

생성자는 클래스 불변식을 위반하는 객체가 만들어지는 것을 막으려면, 생성자에 전달되는 인자의 유효성을 반드시 검사해야 한다. cf) 불변 클래스는 설계, 구현이 쉽다. 객체의 상태를 변경하는 어떤 메서드도 제공안한다. 모든 필드는 fianl. 상속 불가. private 선언. <br>

이번 절에서 다룬 내용을 잘못 받아들여 “인자에 제약을 두는 것은 바람직하다”고 믿어버리면 곤란하다. 메서드는 가능하면 일반적으로 적용될 수 있도록 설계해야 한다. 메서드가 받을 수 있는 읹에 제약이 적으면 적을수록 더 좋다. 

###규칙39 : 필요하다면 방어적 복사본을 만들라 
여러분이 만드는 클래스의 클라이언트가 불변식을 망가뜨리기 위해 최선을 다할 것이라는 가정하에, 방어적으로 프로그래밍해야 한다. 아래의 클래스는 기간을 나타내는 객체에 대한 변경 불가능 클래스다. 
```
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
```
// Period 객체의 내부 구조를 공격
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 변경 ! 
```
**따라서 Period 객체의 내부를 보호하려면 생성자로 전달되는 변경 가능 객체를 반드시 방어적으로 복사해서 그 복사본을 Period 객체의 컴포넌트로 이용해야 한다. **
```
// 수정된 생성자 - 인자를 방어적으로 복사함
public Period(Date start, Date end){
	this.start = new Date(start.getTime());
	this.end = new Date(end.getTime());

	if(this.start.compareTo(this.end) > 0)
		throw new IllegalArgumentException(this.start + “after” + this.end);
	
}
```
인자의 유효성을 검사하기 전에 방어적 복사본을 만들었다는 것에 유의하자. 유효성 검사는 복사본에 대해서 시행한다. 자연스러워 보이지 않을지도 모르나, 필요한 절차다. 인자를 검사한 직후 복사본이 만들어지기 직전까지의 시간, 그러니까 “취약 구간” 동안에 다른 스레드가 인자를 변경해 버리는 일을 막기 위한 것이다. (TICTOU 공격) <br>

방어적 복사본을 만들 때 Date의 clone 메서드를 이용하지 않았다. Date 클래스는 final 클래스가 아니므로, clone 메서드가 반드시 java.util.Date 객체를 반환할 거라는 보장이 없다. 공격을 위해 특별히 설계된 하위 클래스 객체가 반환될 수도 있다. **이런 공격을 막으려면 인자로 전달된 객체의 자료형이 제 3자가 계승할 수 있는 자료형일 경우, 방어적 복사본을 만들 때  clone을 사용하지 않도록 해야 한다. **<br>

**접근자를 통한 공격**<br>
위의 생성자를 사용하면 생성자 인자를 통한 공격은 막을 수 있으나 접근자를 통한 공격은 막을 수 없다. 접근자를 호출하여 얻은 객체를 통해 Period 객체 내부를 변경할 수 있기 때문이다.
```
// Period 객체 내부를 노린 두 번째 공격 형태
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부를 변경 ! 
```
이런 공격을 막으려면 변경 가능 내부 필드에 대한 방어적 복사본을 반환하도록 접근자를 수정해야 한다. 
```
// 수정된 접근자 - 내부 필드의 방어적 복사본 생성 
public Date start(){
	return new Date(start.getTime());
}

public Date end(){
	return new Date(end.getTime());
}
```
이렇게 수정하고 나면 Period는 진정한 변경 불가능 클래스가 된다. 객체 안에 확실히 캡슐화된 필드가 된 것이다. <br>

방어적 복사는 변경 불가능 클래스에서만 쓰이는 기법이 아니다. **클라이언트가 제공한 객체를 내부 자료 구조에 반영하는 생성자나 메서드에는 사용 가능하다. ** 예를 들어, 클라이언트가 제공한 객체 참조를 내부 Set 객체의 요소로 추가해야 하거나 내부 Map 객체의 키로 써야 하는 경우, 삽입된 객체가 나중에 변경된다면 집합이나 맵의 불변식은 깨지고 말 것이다. <br>

방어적 복사본을 만들도록 하면 성능에서 손해를 보기 때문에, 적절치 않을 때도 있다. 클라이언트가 같은 패키지 안에 있다거나 하는 이유로, 클라이언트가 객체의 내부 상태를 변경하려 하지 않는다는 것이 확실하다면 방어적 복사본은 만들지 않아도 될 것이다. **여기서 배워야 할 진짜 교훈은, 객체의 컴포넌트로는 가능하다면 변경 불가능 객체를 사용해야 한다는 것이다.** 그래야 방어적 복사본에 대해서는 신경 쓸 필요가 없어지기 때문이다. <br>

###규칙40 : 메서드 시그니처는 신중하게 설계해라 
**메서드 이름은 신중하게 고르라.**<br>
**편의 메서드를 제공하는 데 너무 열 올리지 마라. **클래스에 메서드가 너무 많으면 학습, 사용, 테스트, 유지보수 등의 모든 측면에서 어렵다. <br>
**인수 리스트(parameter list)를 길게 만들지 마라. **4개 이하가 되도록 애쓰라. 긴 인자 리스트를 짧게 줄이는 방법으로는 첫째, 여러 메서드로 나누는 방법이 있고 둘째, 도움 클래스를 만들어 인자들을 그룹별로 나누는 것이다. 보통 이 도움 클래스들은 static 멤버 클래스다(자주 등장하는 일련의 인자들이 어떤 별도 개체를 나타낼 때 쓰면 좋다). 셋째, 빌더 패턴이 있다. <br>
**인자의 자료형으로는 클래스보다 인터페이스가 좋다.**<br>
**인자 자료형으로 boolean을 쓰는 것보다는, 원소가 2개인 enum 자료형을 쓰는 것이 낫다.**<br> 

###규칙 41 : 오버로딩할 때는 주의하라 
아래의 프로그램 목적은 컬렉션을 종류별로(집합이냐, 리스트냐, 아니면 다른 종류의 컬렉션이냐) 분류하는 것이다.
```
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
이 프로그램이 Set, List, Unkwon Collection을 순서대로 출력하지 않을까 기대하겠지만 실제로는 Unknown Collection을 세 번 출력한다. 그 이유는 **classify 메서드가 오버로딩되어 있으며, 오버로딩된 메서드 가운데 어떤 것이 호출될지는 컴파일 시점에 결정되기 때문이다. **루프가 세 번 도는 동안, 인자의 컴파일 시점 자료형은 전부 `Collection<?>`으로 동일하다. 각 인자의 실행시점 자료형(runtime type)은 전부 다르지만, 선택 과정에는 영향을 끼치지 못한다. 인자의 컴파일 시점 자료형이 `Collection<?>`이므로 호출되는 것은 항상 `classify(Collection<?>)` 메서드다. <br>

이 예제 프로그램은 직관과는 반대로 동작한다. **오버로딩된 메서드는 정적(static)으로 선택되지만, 재정의된 메서드는 동적(dynamic)으로 선택되기 때문이다.** 재정의된 메서드의 경우 선택 기준은 메서드 호출 대상 객체의 자료형이다. 객체 자료형에 따라 실행 도중에 결정되는 것이다. 그렇다면 재정의된 메서드란 무엇인가? 상위 클래스에 선언된 메서드와 같은 시그니처를 갖는 하위 클래스 메서드가 재정의된 메서드다. 하위 클래스에서 재정의한 메서드를 하위 클래스 객체에 대해 호출하면, 해당 객체의 컴파일 시점 자료형과는 상관없이, 항상 하위 클래스의 재정의 메서드가 호출된다. 
```
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
name 메서드는 Wine 클래스에 선언되어 있고, sparklingWine과 Champagne은 그 메서드를 재정의한다. 기대대로 위의 프로그램은 순서대로 출력한다. 순환문의 매 루프마다 객체의 컴파일 시점 자료형은 항상 Wine이었는데도 말이다. **재정의 메서드 가운데 하나를 선택할 때 객체의 컴파일 시점 자료형은 영향을 주지 못한다. 오버로딩에서는 반대로 실행시점 자료형이 아무 영향도 주지 못한다.** 실행될 메서드는 컴파일 시에, 인자의 컴파일 시점 자료형만을 근거로 결정된다. <br>

오버로딩은 직관적인 예측에 반하므로 혼란스럽다. 따라서 오버로딩을 사용할 때는 혼란스럽지 않게 주의해야 한다. 혼란을 피하는 안전하고 보수적인 전략은, 같은 수의 인자를 갖는 두 개의 오버로딩 메서드를 API에 포함시키지 않는 것이다. 예를 들어 ObjectOutputStream의 경우를 생각해 보자. 이 메서드들은 writeBoolean(boolean), writeInt(int), writeLong(long) 같이 정의되어 있다. 이런 작명 패턴을 따르면 오버로딩에 비해 어떤 점이 좋을까? 각 메서드에 대응되는 read 메서드를 정의할 수 있게 된다. (readBoolean(), readInt(), readLong() 등을 정의할 수 있게 된다). <br>

하지만 생성자에는 다른 이름을 사용할 수 없다. 생성자가 많다면, 그 생성자들은 항상 오버로딩된다. 그게 문제라면 생성자 대신 정적 팩터리 메서드를 사용하는 옵션을 사용할 수도 있다. 하지만 같은 수의 인자를 받는 오버로딩 메서드가 많더라도, 어떤 오버로딩 메서드가 주어진 인자 집합을 처리할 것인지가 분명히 결정된다면 프로그래머는 혼란을 겪지 않을 것이다. 그 조건은, 두 개의 오버로딩 메서드를 비교했을 때 그 형식 인자 가운데 적어도 하나가 “확실히 다르다”면 만족된다. 확실히 다르다라는 것은 두 자료형을 서로 형변환 할 수 없다면 확실히 다른 것이다. 예를 들어 ArrayList에는 int를 받는 생성자와 Collection을 인자로 받는 생성자가 있다. 어떤 상황에서라도 이 두 생성자 간에 혼란의 여지가 있으리라고는 보기 어렵다. <br>