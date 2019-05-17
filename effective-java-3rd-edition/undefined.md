# 제네릭

## 규칙26 : 새 코드에는 무인자 제네릭 자료형을 사용하지 마라

`List<String>`는 원소 자료형이 String인 리스트를 나타내는 형인자 자료형이다.

각 제네릭 자료형은 새로운 무인자 자료형을 정의하는데, 무인자 자료형은 실 형인자 없이 사용되는 제네릭 자료형이다. ex\) `List list = new ArrayList<>();` 무인자 자료형 List는 제네릭 도입 이전의 인터페이스 자료형 List와 똑같이 동작한다. 하지만 **제네릭 자료형을 형인자 없이 사용하면 안된다.** 무인자 자료형을 쓰면 형 안전성이 사라지고, 제네릭의 장점 중 하나인 표현력 측면에서 손해를 보게 된다.

제네릭 자료형을 쓰고 싶으나 실제 형 인자가 무엇인지는 모르거나 신경 쓰고 싶지 않을 때는 형 인자로 '?'를 쓰면 된다. 하지만 null 이외의 어떤 원소도 넣을 수 없기 때문에 한정적 와일드 카드 자료형을 쓰면 된다. ex\) `<T extends Number>`

새로 만든 코드에는 무인자 자료형을 쓰면 안된다고 했지만, 그 규칙에도 사소한 예외가 두 가지 있다. 제네릭 자료형 정보가 프로그램이 실행될 때는 지워지기 때문에 생긴 예외들이다. 첫 번째는 클래스 리터럴에는 반드시 무인자 자료형을 사용해야 한다. 자바 표준에 따르면, 클래스 리터럴에는 형인자 자료형을 쓸 수 없다.\(배열 자료형이나 기본 자료형은 가능\) 예를 들어, `List.class, String[].class, int.class`는 가능하지만 `List<String>.class나 List<?>.class`는 사용할 수 없다는 뜻이다. 두 번째는 제네릭 자료형에 instanceof 연산자를 적용할 때는 다음과 같이 하는 것이 좋다.

```java
//instanceof 연산자에는 무인자 자료형을 써도 OK
if(o instanceof Set){ //무인자 자료형
    Set<?> m = (Set<?>) o; //와일드카드 자료형 
}
```

## 규칙27 : 무점검\(unchecked warning\) 경고를 제거하라

제거할 수 없는 경고 메세지는 형 안전성이 확실할 때만 `@SupressWarnings("unchecked")`애노테이션을 사용해 억제해라. 개별 지역 변수 선언부터 클래스 전체에까지, 어떤 크기의 단위에도 적용할 수 있지만 가능한 한 작은 범위에 적용해라. 보통은 변수 선언이나 아주 짧은 메서드 또는 생성자에 붙인다. **절대로 클래스 전체에 SupressWarnings을 적용하지 마라.** 그리고 이 애노테이션을 사용할 때마다, 왜 형 안전성을 위반하지 않는지 밝히는 주석을 반드시 붙여라.

무점검 경고를 무시하면 프로그램 실행 도중에 CalssCastException이 발생할 가능성이 있다.

## 규칙28 : 배열 대신 리스트를 써라

배열은 공변 자료형이다. Sub가 Super의 하위 자료형이라면 `Sub[]`도 `Super[]`의 하위 자료형이라는 것이다. 반면 제네릭은 불변 자료형이다. Type1과 Type2가 있을 때, `List<Type1>`은 `List<Type2>`의 상위 자료형이나 하위 자료형이 될 수 없다.

```java
// 실행 중에 문제를 일으킴
Object[] objectArray = new Long[1];
objectArray[0] = "hello"; // ArrayStoreException 예외 발생

// 컴파일 되지 않는 코드
List<Object> ol = new ArrayList<Long>(); //자료형 불일치
ol.add("hello");
```

배열을 쓰면 실수를 저지른 사실을 프로그램 실행 중에나 알 수 있지만 리스트를 사용하면 컴파일 할 때 알 수 있다.

배열과 제네릭의 두 번째로 중요한 차이는, **배열은 실체화\(reification\) 되는 자료형**이라는 것이다. 즉, 배열의 각 원소의 자료형은 실행시간\(runtime\)에 결정된다는 것이다. 그래서 컴파일 타임에 형 안전성을 보장하지 못한다. 반면 **제네릭은 삭제 과정을 통해 구현된다.** 즉, 자료형에 관계된 조건들은 컴파일 시점에만 적용되고, 그 각 원소의 자료형 정보는 프로그램이 실행될 때는 삭제된다는 것이다. 자료형 삭제 덕에, 제네릭 자료형은 제네릭을 사용하지 않고 작성된 오래된 코드와도 문제없이 연동한다.

이런 기본적인 차이점 때문에 배열과 제네릭은 섞어 쓰기 어렵다. 예를 들어, 제네릭 자료형이나 형인자 자료형, 또는 형인자의 배열을 생성하는 것은 문법적으로 허용되지 않는다. 즉, `new List<E>[], new List<String>[], new E[]`는 전부 컴파일되지 않는 코드다. 컴파일 하려고 하면 제네릭 배열 생성 \(generic array creation\)이라는 오류가 발생할 것이다.

제네릭 배열 생성이 허용되지 않는 이유는 형 안전성\(typesafe\)이 보장되지 않기 때문이다.

```java
// 저네릭 배열 생성이 허용되지 않는 이유 - 아래의 코드는 컴파일 되지 않는다.
List<String>[] stringList = new List<String>[1]; // (1)
List<Integer> inList = Arrays.asList(42);        // (2)
Object[] objects = stringList;                   // (3)
objects[0] = inList;                             // (4)
String s = stringList[0].get(0);                 // (5)
```

\(1\)은 generic array creation 오류가 발생한다. 하지만 문제없이 컴파일 된다고 가정해보자. 그러면 제네릭 배열이 만들어질 것이다. \(2\)는 하나의 원소를 갖는 배열 `List<Integer>`를 초기화한다. \(3\)은 `List<String>` 배열을 Object 배열 변수에 대입한다. 배열은 공변 자료형이므로 가능하다. \(4\)에서는 `List<Integer>`를 Object 배열에 있는 유일한 원소에 대입한다. 제네릭이 형 삭제\(erasure\)를 통해 구현되므로 여기에도 하자는 없다. `List<Integer>` 객체의 실행시점 자료형\(runtime type\)은 List이며, `List<String>[]`의 실행시점 자료형 또한 `List[]`이다. 이 대입문은 ArrayStoreException을 발생시키지 않는다. 문제는 지금부터다. `List<String>` 객체만을 담는다고 선언한 배열에 `List<Integer>`를 저장한 것이다. \(5\)에서는 이 배열 안에 있는 유일한 원소를 꺼내는 작업을 하고 있는데, 컴파일러는 꺼낸 원소의 자료형을 자동적으로 String으로 변환할 것이다. 사실은 Integer인데 말이다. 그러니 프로그램 실행 도중에 ClassCastException이 발생하고 말 것이다. 이런 일이 생기는 것을 막으려면 \(1\)처럼 제네릭 배열을 만들려고 하면 컴파일 할 때 오류가 발생해야 한다.

`E, List<E>, List<String>`와 같은 자료형은 실체화 불가능 자료형으로 알려져 있다. 쉽게 말하자면, 프로그램이 실행될 때 해당 자료형을 표현하는 정보의 양이 컴파일 시점에 필요한 정보의 양보다 적은 자료형이 실체화 불가능 자료형이다. 실체화 가능한 형인자 자료형은 `List<?>`나 `Map<?,?>` 같은 비한정적 와일드카드 자료형 뿐이다. 쓸 일이 별로 없긴 하지만, 비한정적 와일드카드 자료형의 배열은 문법상 허용된다.

제네릭 배열을 만들 수 없다는 것이 짜증스럽게 느껴질 수도 있다. 예를 들자면, 제네릭 자료형에 담긴 원소들의 자료형으로 만든 배열을 반환하는 것은 일반적으로 불가능하다. 또한 제네릭 자료형을 varargs 메서드와 함께 사용하면 혼란스런 경고 메세지들을 보게된다. varargs 메서드를 호출할 때마다 varargs 인자들을 담을 배열이 생성되기 때문이다\(자바 1.7부터는 이런 상황에서 발생하는 경고 메세지가 개선되었다\). 이런 경고는 억제하거나, 제네릭을 varargs와 혼용하지 않도록 주의하는 것 말고는 처리할 방법이 별로 없다.

3rd edition추가 : The SafeVarargs annotation can be used to address this issue \(Item 32\)

제네릭 배열 생성 오류에 대한 가장 좋은 해결책은 보통 `E[]` 대신 `List<E>`를 쓰는 것이다. 성능이 저하되거나 코드가 길어질 수는 있겠으나, 형 안전성과 호환성은 좋아진다.

예를 들어, 아래와 같이 생성자로 Collection을 받고 choose\(\) 메서드로 랜덤으로 선택된 collection element를 리턴한다고 해보자. 생성자에 전달하는 컬렉션에 따라 선택기를 게임 다이, 마술 8 볼 또는 몬테카를로 시뮬레이션을위한 데이터 소스로 사용할 수 있다.

```java
// 제네릭을 필요로 하는 클래스
public class Chooser1 {
    private final Object[] choiceArray;

    public Chooser1(Collection choices) {
        this.choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        int i = rnd.nextInt(choiceArray.length);
        Object o = choiceArray[i];
        return o;
    }
}
```

이 클래스를 사용하려면 메서드 호출을 사용할 때마다 Object에서 반환되는 choose 메서드의 반환 값을 원하는 형식으로 캐스팅해야하며 형식이 잘못되면 런타임에 캐스트가 실패한다. Item 29의 조언을 받아 Chooser1를 제네릭으로 만든다.

```java
// A first cut at making Chooser genric - won't compile
public class Chooser2<T> {
    private final T[] choiceArray;

    public Chooser2(Collection<T> choices) {
        this.choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

컴파일을 시도하면 다음과 같은 에러 메세지가 나온다.

```text
Incompatible types.
Required: T[]
Found: java.lang.Object[]
```

아래와 같이 수정하면 에러는 없어지지만 경고가 뜬다.

```java
this.choiceArray = (T[])choices.toArray();
```

```text
Unchecked cast: 'java.lang.Object[]' to 'T[]
```

컴파일러는 런타임에 프로그램이 어떤 유형의 T를 나타내는지 알 수 없으므로 런타임 시 캐스트의 안전성을 보장할 수 없다고 알려준다. 런타임 시 제네릭에서 요소 유형 정보가 지워진다는 것을 기억해라.

unchecked cast warning을 지우기 위해 배열 대신 리스트를 사용해라.

```java
public class Chooser3<T> {
    private final List<T> choiceList;

    public Chooser3(Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        int size = choiceList.size();
        int index = rnd.nextInt(size);
        return choiceList.get(index);
    }
}
```

이 버전은 좀 더 길고, 다소 느린 편이지만, 런타임시 ClassCastException을 얻지 못할 것이라는 점에 마음이 편한 가치가 있다.

요약하면 제네릭과 배열은 자료형 규칙이 다르다. 배열은 공변 자료형이자 실체화 가능 자료형이다. 제네릭은 불변 자료형이며, 실행 시간에 형인자의 정보는 삭제된다. 따라서 배열은 컴파일 시간에 형 안전성을 보장하지 못하며, 제네릭은 그 반대다. 일반적으로 배열과 제네릭은 쉽게 혼용할 수 없다. 만일 배열과 제네릭을 뒤섞어 쓰다가 컴파일 오류나 경고 메세지를 만나게 되면, 배열을 리스트로 바꿔야겠다는 생각이 본능적으로 들어야한다.

## 규칙29 : 가능하면 제네릭 자료형으로 만들 것

클래스를 제네릭화하는 첫 번째 단계는 선언부에 형인자를 추가하는 것이다. `public class Stack<E>` 그 다음 단계는 Object를 자료형으로 사용하는 부분들을 전부 찾아서, 형인자 E로 대체하고 컴파일해 보는 것이다.

E와 같은 실체화 불가능 자료형으로는 배열을 생성할 수 없다.

```java
private E[] elements; 
private static final int DEFAULT_INITIAL_CAPACITY = 16;

public Stack(){
    elements = new E[DEFAULT_INITIAL_CAPACITY]; //문제!!
}

//이렇게 해야한다
@SuppressWarnings("unchecked")
public Stack(){
    elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY]; 
}
```

위의 방식이 일반적으로 형 안전성을 보장하는 방법이 아니지만\(컴파일러가 프로그램의 형 안전성을 입증할 수 없어서\) 개발자가 프로그램의 형 안전성을 해치지 않음을 확실히 해야 한다. 위에서 문제가 되고 있는 배열 elements는 private 필드이고 클라이언트에 반환되지 않으며 다른 어떤 메서드에도 전달되지 않는다. push 메서드에 전달되는 원소만이 배열에 저장되며, 그 타입은 전부 E다. 따라서 무점검 형변환을 해도 아무런 문제가 없다. 무점검 형변환이 안전함을 증명했다면, 경고를 억제하되 범위는 최소한으로 줄요야 한다.\(규칙 24\)

## 규칙30 : 가능하면 제네릭 메서드로 만들 것

형인자를 선언하는 형인자 목록은 메서드의 수정자와 반환값 자료형 사이에 둔다.`public static <E> Set<E> union(Set<E> s1, Set<E> s2)`

`<T extends Comparable<T>>` 재귀적 자료형 한정이라 한다. "자기 자신과 비교 가능한 모든 자료형 T"라는 뜻으로 읽을 수 있다.

## 규칙31 : 한정적 와일드카드를 써서 API 유연성을 높여라

PECS\(Produce - extends, Consumer - Super\)

```java
//E 객체 생산자 역할을 하는 인자에 대한 와일드카드 자료형
public void pushAll(Iterable<? extends E> src){
    for(E e : src)
        push(e);
}

//E의 소비자 구실을 하는 인자에 대한 와일드카드 자료형
public void popAll(Collection<? super E> dst){
    while(!isEmpty())
        dst.add(pop());
}
```

Stack 예제에서 pushAll의 인자 src는 스택이 사용할 E 형의 객체를 만드는 생산자이므로 src의 자료형은 Iterable&lt;? extends E&gt;가 되어야 한다. popAll의 dst는 Stack에 보관된 E 객체를 소비하므로 dst의 자료형은 Collection&lt;? super E&gt;가 되어야 한다.

`public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)` 반환값에는 와일드카드 자료형을 쓰면 안 된다. 클라이언트 코드 안에도 와일드카드 자료형을 명시해야 하기 때문이다.

이제 규칙 27에서 다룬 max 메서드를 고쳐보자 `public static <T extends Comparable<? super T>> T max(List<? extends T> list` 이 리스트는 T 객체의 생산자이므로 자료형을 `List<T>`에서 `List<? extends T>`로 바꾸었다. 까다로운 부분은 형인자 T에 PECS 원칙을 적용한 과정이다. T에 대한 Comparable은 T 객체를 소비하므로\(그리고 순서 관계를 나타내는 정수 값을 생산하므로\) 형인자 자료형 `Comparable<T>`를 한정적 와일드카드 자료형 `Comparable<? super T>`로 바꿨다. Comparable과 Comparator는 언제나 소비자라서 `<? super T>`를 사용해야 한다.

**유용한 swap 방법**

```java
public static void swap(List<?> list, int i, int j){
    //이렇게 하면 메서드에 아무 리스트나 인자로 전달하면 첨자가 가리키는 원소들을 바꿔준다.
    //형인자를 신경 쓸 필요가 없어서 좋지만 문제는 list의 자료형이 List<?>라는 것이다.
    //List<?>에는 null 이외에 어떤 값도 넣을 수 없다.
    list.set(i, list.set(j, list.get(i)));

    //그래서 도움 메서드가 필요
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j){
    list.set(i, list.set(j, list.get(i)));
}
```

swapHelper 메서드는 list가 `List<E>`라는 것을 안다. 따라서 해당 리스트에서 꺼낸 값의 자료형은 E다. 그리고 E 형의 값은 리스트에 넣어도 안전하다.

## item 32 : Combine generics and varargs judiciously

## 규칙33 : 형 안전 다형성 컨테이너를 쓰면 어떨지 따져보라

하나의 원소만을 담는 컨테이너 대신 키\(key\)에 형인자를 지정하는 유연한 방법.

```java
//형 안전 다형성 컨테이너 패턴 - 구현
public class Favorites{
    private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();

    public <T> void putFavorite(Class<T> type, T instance){
        if(type == null)
            throw new NullPointerException("Type is null");
        //동적 형변환으로 실행시간 형 안전성 확보
        favorites.put(type, type.cast(instance));
    }

    public <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
    }
}
```

비한정적 와일드카드 자료형을 사용했으니 이 맵에는 아무것도 넣을 수 없다고 생각할 수 있지만, 와일드카드 자료형이 쓰인 곳은 맵이 아니라 키다. 모든 키가 상이한 형인자 자료형을 가질 수 있다는 의미다.

두 번째로 주의할 것은 favorites 맵의 값 자료형이 Object라는 것이다. 그래서 값의 자료형이 키가 나타내는 자료형이 되도록 보장하지 않는다. 이를 해결하기 위해 Class 객체의 cast 메서드를 사용해서, Object 형 객체 참조를 Class 객체가 나타내는 자료형으로 동적 형변환한다. cast 메서드는 자바의 형변환 연산자의 동적 버전이다. 이 메서드가 하는 일은 단순히 주어진 인자가 Class 객체가 나타내는 자료형의 객체인지를 검사하는 것이다. 맞는다면 인자로 주어진 객체를 반환하고, 그렇지 않은 경우에는 ClassCastException을 던진다. `favorites.put(type, type.cast(instance))` 에서도 cast 메서드를 사용했는데 의도적으로 형 안전성을 깨뜨리는걸 방지하기 위해서다.

Favorites 클래스의 단점은 실체화 불가능 자료형에는 쓰일 수 없다는 것이다. `List<String>.class`가 문법적으로 옳지 않은 표현이기 때문이다.

