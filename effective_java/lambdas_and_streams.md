# 람다와 스트림
### item44 : 표준 함수형 인터페이스를 사용해라
Java에 람다 (lambdas)가 생겨서 API를 작성하는 모범 사례가 많이 바뀌었다.
예를 들어, 템플릿 메서드 패턴은 이제 덜 매력적이다.
동일한 효과를 얻기 위해 함수 객체를 받는 static factory나 생성자가 모던한 대체제다.
더 일반적으로, 함수 객체를 파라미터로 받는 생성자나 메서드를 작성할거다.
올바른 함수형 파라미터 타입을 선택하는것은 주의가 필요하다.

LinkedHashMap을 생각해보자. 아래의 removeEldestEntry 메서드를 오버라이딩해서 map에 put할 때마다 호출하려고 한다.
```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
```
이 메서드가 true를 리턴하면, 맵은 eldest entry를 제거한다. 아래와 같이 오버라이드한다면 맵 사이즈가 100개를 항상 유지시킬 수 있다.

```java
@Override
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > 100;
    }
```
이 기법은 올바르게 동작하지만 람다로 더 좋게 표현할 수 있다.
LinkedHashMap이 오늘 작성된 경우 함수 객체를 사용하는 정적 팩토리 또는 생성자를 갖는다.
removeEldestEntry 메서드 선언을 보면, 함수 객체는 `Map.Entry<K, V>`를 받고 boolean을 리턴해야된다고 생각할 수 있다.
그러나 그렇지 않다. removeEldestEntry 메서드 안에서 size() 메서드를 호출하는데, removeEldestEntry 메서드가 인스턴스 메서드이기 때문에 가능하다.
생성자에 전달하는 함수 객체는 인스턴스 메서드가 아니기 때문에 캡쳐를 할 수 없다(맵은 팩토리나 생성자가 호출될 때 아직 존재하지 않기 때문).
그러므로, 맵은 함수 객체에 자신을 넘기고, eldest entry도 넘겨야 한다. 함수형 인터페이스를 선언한다면 아래와 같을것이다.
```java
@FunctionalInterface
interface EldestEntryRemovalFunction<K,V> {
	boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```
이 인터페이스는 잘 동작하겠지만, 새로운 인터페이스를 선언할 필요는 없다. `java.util.function` 패키지는 다양한 표준 함수형 인터페이스를 제공한다.
**표준 함수형 인터페이스 중 하나가 작업을 수행하는 경우 일반적으로 특수 기능 인터페이스보다 우선적으로 사용해야한다.**
표준 함수형 인터페이스를 사용하는 것은 개념적 표면적을 줄임으로써 API를 배우기 쉽게 만들고, 많은 표준 함수형 인터페이스가 유용한 디폴트 메서드들을 제공하므로 중요한 상호 운용성 이점을 제공한다.

예를들어 Predicate 인터페이스는 predicates 결합하기 위한 메서드를 제공한다. LinkedHashMap 예제에서 표준 `BiPredicate<Map<K,V> map, Map.Entry<K,V>>` 인터페이스를 이용하는게 낫다.
```java
@FunctionalInterface
public interface BiPredicate<T, U> {

    /**
     * Evaluates this predicate on the given arguments.
     *
     * @param t the first input argument
     * @param u the second input argument
     * @return {@code true} if the input arguments match the predicate,
     * otherwise {@code false}
     */
    boolean test(T t, U u);

	...
}
```
 `java.util.function` 패키지에는 43개의 인터페이스가 있다. 모두 다 기억할 순 없지만, 기본 인터페이스 6개만 기억한다면 나머지는 필요할 때 이끌어낼수 있다. 기본 인터페이스들은 object reference types에서 동작한다.

6개의 기본 함수형 인터페이스 요약은 아래와 같다.
| Interface | Function Signature | Example |
| -------- | -------- | -------- |
| `UnaryOperator<T>`     | T apply(T t)     |  String::toLowerCase  |
|  `BinaryOperator<T>`     |  T apply(T t1, T t2)     |  BigInteger::add     |
| `Predicate<T>`     |  boolean test(T t)    | Collection::isEmpty   |
|  `Function<T,R>`     |  R apply(T t)     | Arrays::asList  |
| `Supplier<T>`     |  T get()     | Instant::now  |
|  `Consumer<T>`    |  void accept(T t)     |  System.out::println   |

또한 6개 기본 인터페이스 각각에 int, long, double 3가지 primitive type에서 동작하는 변종 인터페이스도 있다(네이밍은 기본 인터페이스 이름 prefix에 추가).
예를 들어 int를 받는 predicate는 IntPredicate이고 두개의 long 값을 받고 long을 리턴하는 binary operator는 LongBinaryOperator다.
이런 변종 인터페이스는 Function 류만 리턴 타입을 파라미터화한다. 예를 들어, `LongFunction<int[]>`는 `int[]`를 리턴한다.
Function 인터페이스에는 9개의 추가적인 변종들이 있는데 result 타입이 primitive일 때 사용된다. source와 result 타입이 항상 다른데, 똑같다면 UnaryOperator 인터페이스다. 만약 source와 result 타입이 primitive이면, SrcToResult 형식으로 네이밍이 붙는다(ex LongToIntFunction)
만약 source는 primitive인데 result type이 object reference라면 `<Src>ToObj` 형식으로 네이밍한다(ex DoubleToObjFunction)

3개의 기본 함수형 인터페이스에 두개인자를 갖는 버전들이 있다. `BiPredicate<T,U>` `BiFunction<T,U,R>` `BiConsumer<T,U>`
또한 리턴을 primitive 타입으로 하는 BiFunction 류가 있다. `ToIntBiFunction<T,U>` `ToLongBiFunction<T,U>` `ToDoubleBiFunction<T,U>` 그리고 하나는 object reference와 나머지 하나는 primitive 타입을 받는 Consumer 류가 있다.
`ObjDoubleConsumer<T>` `ObjIntConsumer<T>` `ObjLongConsumer<T>` 이렇게 총 9개가 기본 인터페이스의 두개 인자받는 유형들이다.

마지막으로 BooleanSupplier 인터페이스가 있는데, return 타입이 boolean이다. 기본 함수형 인터페이스 네이밍에서 Boolean이 명시적으로 사용되는 유일한 인터페이스이다. boolean return 타입은 Predicate와 그 변종들이 지원하고 있기 때문이다.
이전 단락에서 설명한 BooleanSupplier 인터페이스와 42개 인터페이스들은 모든 43개 표준 함수형 인터페이스들을 설명한다.
솔직히 삼켜야 될게 많지만(알아야 될게 많지만) 끔찍할 정도는 아니다. 다른 한편으로 필요한 함수형 인터페이스의 대부분은 당신을 위해 쓰여졌고 그 이름은 충분히 규칙적이어서 필요할 때 사용하기 어려움을 겪지않아야한다.

대부분의 표준 함수형 인터페이스는 primitive 타입을 위해 제공된다.
**primitive 함수형 인터페이스를 사용하는것 대신에 기본 함수형 인터페이스에 boxed primitive를 함께 사용하려는 유혹을 받지 마라.**
동작은 하겠지만, 규칙 61에 위배된다(Prefer primitive types to boxed primitives). 대량의 operation에서 boxed primitive의 성능은 안좋다.

이제 일반적으로 직접 작성한 함수형 인터페이스보다 표준 함수형 인터페이스를 사용해야된다는 것을 알았을것이다. 그러나 언제 직접 작성한걸 써야할까?
당연히 표준에 없는거라면 직접 작성해 써야한다. 예를들어, 3개의 파라미터가 필요한 Predicate 라던지, checked exception을 throw 해야된다던지.
그러나 표준 인터페이스 중 하나와 구조적으로 동일한 경우에도 사용자 고유의 함수형 인터페이스를 작성해야 할 때가 있다.

`Comparator<T>`를 생각해보면 `ToIntBiFunction<T,T>`와 구조적으로 동일하지만 사용하면 안된다. 여러가지 이유가 있는데 첫째, 함수의 네이밍은 API를 사용할 때 최고의 문서로써 제공된다.
둘째, Comparator 인터페이스를 사용할 때는 각 인스턴스들을 비교하고자 하는 강한 요구사항이 있다.
셋째, 인터페이스에는 comparators를 변환하고 결합하는 유용한 default 메소드가 많이 필요하다.

아래와 같은 Comparator 특성에 따라 신중하게 생각해서 직접 작성한 인터페이스를 사용해야한다.
- 일반적으로 사용되며 설명이 포함된 이름으로 이득이 된다.
- 강한 계약이 있다(it has a strong contract associated with it).
- 커스텀 디폴트 메서드의 이점을 얻는다.

만약 직접 작성하기로 했다면 그것이 인터페이스임을 명심하고 주의해서 설계해라.

EldestEntryRemovalFunction 인터페이스에 @FunctionalInterface 애노테이션이 붙은것을 주목해라.
이 애노테이션은 @Override 정신과 유사하다. 이 애노테이션은 다음과 같은 3가지 목적의 의도를 말해준다.
첫째, 람다사용이 가능하도록 인터페이스가 설계되었다 둘째, 정확히 하나의 abstract 메서드만 있지 않는 한 컴파일 되지 않으므로 작성자가 실수하지 않는다.
셋째, 인터페이스가 진화함에 따라 유지관리자가 abstract 메서드를 추가하는 실수를 막아준다. **함수형 인터페이스에는 항상 @FunctionalInterface 애노테이션을 붙여라**

마지막 포인트는 API에서 함수형 인터페이스를 사용할때다. 클라이언트에서 가능한 모호성을 만들 수있는 경우 동일한 인수 위치에서 다른 함수형 인터페이스를 사용하는 여러 오버로드가 있는 메소드를 제공하지 마라. 이것은 단지 이론적인 문제가 아니다. ExecutorService 의 submit 메서드는 `Callable<T>`나 Runnable을 받을 수 있고 올바른 오버로딩을 나타내기 위해 cast가 필요한 클라이언트 프로그램을 작성하는게 가능하다.
```java
   <T> Future<T> submit(Callable<T> task);
   <T> Future<T> submit(Runnable task, T result);
   Future<?> submit(Runnable task);
```
이러한 문제를 피하는 가장 쉬운 방법은 같은 인자 위치에 다른 함수형 인터페이스를 갖는 오버로딩 메서드를 작성하지 않는것이다.
규칙 52 use overloading judiciously 에서 다루는 특별한 케이스다.

요약하면, 지금 자바는 람다가 가능하고 API를 설계하는데 람다를 생각하는걸 피할수 없다. 함수형 인터페이스 타입을 input과 output에 두는걸 받아들여라.
`java.util.function.Function`가 제공하는 표준 인터페이스를 사용하는게 베스트다. 그러나 상대적으로 드문 케이스로 직접 작성하는 함수형 인터페이스가 더 나을 수 있으니 keep your eys open해라.
