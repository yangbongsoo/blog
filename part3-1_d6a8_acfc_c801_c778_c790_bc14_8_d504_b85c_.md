# Part3-1 효과적인 자바8 프로그래밍
##9장 디폴트 메서드

**스터디에서 나온 내용 :** 인터페이스를 구현한 클래스에서 바로 사용하기 위해 default 메서드를 사용하면 안된다. 인터페이스를 직접 사용하는 클라이언트가 쉽게 쓰기 위해 사용돼야 한다. ex  `list.sort(Compator<? super E> c)`

그리고 만약 한 인터페이스를 구현한 클래스가 10개 있는데 그 중 2개는 인터페이스의 추상 메서드를 잘 안쓰고 빈 구현만 해놨다면 2개의 구현체가 그 인터페이스를 바라보고 있는게 올바른지 의심해볼 필요가 있다.(디폴트 메서드로 만들어서 빈 구현체를 없애는 게 아니라)

---


![](스크린샷 2016-05-23 오후 8.29.25.jpg)
자바 8 이전에는 만약 인터페이스에 새로운 메서드를 정의하면 

![](스크린샷 2016-05-23 오후 8.31.24.jpg)

구현 클래스를 수정해줘야 했다.

그부분이 라이브러리 설계자 입장에서는 큰 제약이었다.
![](스크린샷 2016-05-23 오후 8.36.06.jpg)
특히 모두에게 공개된 API 경우, 사용자가 직접 구현한 클래스까지 설계자가 커버할 수 없다. 

![](스크린샷 2016-05-23 오후 8.48.00.jpg)
그래서 새롭게 나온게 자바8 디폴트 메서드이다. 
```java
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}


default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}


@Override
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, 0);
}

```

![](스크린샷 2016-05-23 오후 8.56.32.jpg)

```java
@SuppressWarnings({"unchecked", "rawtypes"})
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```
cf) Effective Java 규칙 24

제거할 수 없는 경고 메세지는 형 안전성이 확실할 때만 @SuppressWarings(“unchecked”)를 사용해 억제해라.

Rawtypes는 제너릭을 사용하는 클래스 매개 변수가 불특정일 때의 경고다. 

3장에서 소개한 Predicate, Function 등 많은 함수형 인터페이스도 다양한 디폴트 메서드를 포함한다.
![](스크린샷 2016-05-24 오후 3.44.59.jpg)
![](스크린샷 2016-05-24 오후 3.45.16.jpg)

**함수형 인터페이스는 오직 하나의 추상 메서드를 포함한다. 디폴트 메서드는 추상 메서드에 해당하지 않는다는 점을 기억하자.**

**디폴트 메서드가 생기면서 들었던 의문**

1. 이렇게 되면 추상 클래스와 다른게 뭐지?
2. 자바는 다중 상속을 허용안하는데 여러 디폴트 메서드를 상속받을 수 있게 되면서 다중 상속이 가능해진건가? 

**책에서 말하는 추상 클래스와 인터페이스의 차이점**(문법적 차이만 설명함)

1. 클래스는 하나의 추상 클래스만 상속받을 수 있지만 인터페이스는 여러 개 구현할 수 있다.
2. 추상 클래스는 인스턴스 변수로 공통 상태를 가질 수 있지만 인터페이스는 인스턴스 변수를 가질 수 없다.

**1. 추상클래스 vs 인터페이스**

클린코더스 : 추상 클래스 대신 인터페이스를 써라. Extends는 비싸니까(한번밖에 사용불가능)

Effective Java 규칙 18 : 추상 클래스 대신 인터페이스를 사용하라.

**2. 다중 상속**

책에서는 다중 상속으로 프로그램에 유연성을 제공한다고 말한다.

###디폴트 메서드에 대해서 좀 더 자세히 알아보자.
![](스크린샷 2016-05-23 오후 11.30.49.jpg)
API버전 1(p291 ~ 292)에서 Resizable 인터페이스에 새로운 메서드가 추가되었다고 해보자.
```java
public interface Resizable extends Drawable{
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    //새롭게 추가
    void setRelativeSize(int wFactor, int hFactor);
}

```
**재컴파일 하면 에러가 발생한다.**

공개된 API를 고치면 기존 버전과의 호환성 문제가 발생한다. 

cf) 공개된 API란 거창한것이 아니라 public으로 만든 것들을 의미한다.

인터페이스에 메서드를 추가했을 때는 바이너리 호환성을 유지하지만 인터페이스를 구현하는 클래스를 재컴파일
하면 에러가 발생한다. 즉, **다양한 호환성이 있다는 사실을 이해**해야 한다.

**바이너리 호환성 : 뭔가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황.**

ex) 인터페이스에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않는데 이를 바이너리 호환성이라고 한다.

**소스 호환성 : 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일 할 수 있음.**

ex) 인터페이스에 메서드를 추가하면 소스 호환성이 아니다. 추가한 메서드를 구현하도록 클래스를 고쳐야 하기 때문이다. 디폴트 메서드로 만들면 소스 호환성이 유지된다.

**동작 호환성 : 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행한다는 의미.**

ex) 인터페이스에 메서드를 추가하더라도 프로그램에서 추가된 메서드를 호출할 일은 없으므로 동작 호환성은 유지된다.

###디폴트 메서드 활용 패턴

**선택형 메서드(optional method) :** Iterator는 hasNext와 next뿐 아니라 remove 메서드도 정의하지만 사용자들이 remove는 잘 사용하지 않으므로 자바8 이전에는 Iterator를 구현하는 많은 클래스에서 remove에 빈 구현을 제공했다. 하지만 이제 디폴트 메서드를 이용하면 구현 클래스에서 빈 구현을 제공할 필요가 없어진다.
```java
 public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
}
```

**동작 다중 상속(multiple inheritance of behavior)**
![](스크린샷 2016-05-24 오후 4.27.54.jpg)
자바8에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작을 상속받을 수 있다.

**옳지 못한 상속**
상속으로 코드 재사용 문제를 모두 해결할 수 있는 것은 아니다.

예를 들어 한 개의 메서드를 재사용하려고 100개의 메서드와 필드가 정의되어 있는 클래스를 상속받는 것은 좋은 생각이 아니다. 이럴 때는 delegation(위임), 즉 멤버 변수를 이용해서 클래스에서 필요한 메서드를 직접 호출하는 메서드를 작성하는 것이 좋다. 

종종 final로 선언된 클래스를 볼 수 있는데 (ex String) 다른 클래스가 상속다지 못하게 함으로써 원래 동작이 바뀌지않길 원할 때 쓴다. 이렇게 하면 다른 누군가가 그 클래스의 핵심 기능을 바꾸지 못하도록 제한할 수 있다.

###해석 규칙(같은 디폴트 메서드 시그너처가 있을 때) p302
[해석규칙]

1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
2. 1번 규칙 이외의 상황에서는 서브 인터페이스가 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그너처를 갖는 메서드를 정의할때는 서브인터페이스가 이긴다.
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다. 

![](스크린샷 2016-05-24 오후 5.02.16.jpg)

Q) 컴파일러는 누구의 hello 메서드 정의를 사용할까? 정답은 B(2번 규칙에 의해서)

![](스크린샷 2016-05-24 오후 5.09.07.jpg)

Q) 컴파일러는 누구의 hello 메서드 정의를 사용할까? 이것도 정답은 B(2번 규칙에 의해서)

![](스크린샷 2016-05-24 오후 5.16.26.jpg)

Q) 컴파일러는 누구의 hello 메서드 정의를 사용할까? 정답은 D (1번 규칙, 클래스가 항상 이긴다)

![](스크린샷 2016-05-24 오후 5.23.57.jpg)

이번에는 인터페이스 간에 상속관계가 없으므로 2번 규칙을 적용할 수 없다. 충돌을 해결하기 위해선 개발자가 직접 클래스 C에서 hello 메서드를 오버라이드한 다음에 호출하려는 메서드를 **명시적으로 선택해야 한다.**
```java
public class C implements A,B{
    public void hello(){
        B.super.hello();
    }
}
```

###다이아몬드 문제
![](스크린샷 2016-05-24 오후 5.45.12.jpg)

```java
public class D implements B,C {
    public static void main(String... args) {
        new D().hello();
    }
}

```
A만 디폴트 메서드를 정의하고 있다. 따라서 결국 프로그램 출력 결과는 “Hello from A”가 된다.

![](스크린샷 2016-05-24 오후 5.51.12.jpg)

```java
public class D implements B,C {
    public static void main(String... args) {
        new D().hello();
    }
}

```
**B에도 같은 시그너처의 디폴트 메서드가 있다면?**

B는 A를 상속받으므로 2번 규칙에 따라 B가 선택된다.

![](스크린샷 2016-05-24 오후 5.57.00.jpg)

B와 C가 모두 디폴트 메서드 hello 메서드를 정의한다면 충돌이 발생하므로 이전에 설명한 것 처럼 둘 중 하나의 메서드를 **명시적으로 호출해야 한다.**

##10장 null 대신 Optional
값이 없는 상황에서 어떻게 처리할까? 
```java
Person/Car/Insurance 데이터 모델 

public class Person {
  private Car car;
  public Car getCar() { return car; }
}

public class Car {
  private Insurance insurance;
  public Insurance getInsurance() { return insurance; }
}

public class Insurance {
  private String name;
  public String getName() { return name; }
}
```
첫 번째 방법으로는 deep doubt(깊은 의심)이 있다.
```java
public String getCarInsuranceName(Person person) {
  if (persion != null) {
    Car car = persion.getCar();
    if (car != null) {
      Insurance insurance = car.getInsurance();
      if (insurance != null) {
        return insurance.getName();
      }
    }
  }
  return "Unknown";
}
```
두 번째 방법으로는 다양한 출구를 만드는 것이다.
```java
public String getCarInsuranceName(Person person) {
  if (person == null) {
    return "Unknown";
  }
  Car car = person.getCar();
  if (car == null) {
    return "Unknown";
  }
  Insurance insurance = car.getInsurance();
  if (insurance == null) {
    return "Unknown";
  }
  return insurance.getName();
}
```
위 코드는 중첩 if 블록을 없앴지만 네 개의 출구가 생겼기 때문에 유지보수하기 힘들어진다. 

그래서 자바8은 `java.util.Optional<T>`를 제공한다. Optional은 선택형값을 캡슐화하는 클래스다. **값이 있으면 Optional 클래스는 값을 감싼다. 반면 값이 없으면 Optional.empty 메서드로 Optional을 반환한다.** 즉, Optional 타입은 값이 없을 수 있음을 명시적으로 보여준다. 

**빈 Optional **

```java
Optional<Car> optCar = Optional.empty();
```
**null이 아닌 값으로 Optional 만들기**

car가 null이라면 즉시 NPE가 발생한다(Optional을 사용하지 않았다면 car의 프로퍼티에 접근하려 할 때 에러가 발생했을 것이다).
```java
Optional<Car> optCar = Optional.of(car);
```

**null값으로 Optional만들기**

null값을 저장할 수 있는 Optional을 만들 수 있다. 만약 car가 null이면 빈 Optional 객체가 반환된다.
```java
Optional<Car> optCar = Optional.ofNullable(car);
```

###flatMap으로 Optional 객체 연결
```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = 
  optPerson.map(Person::getCar)
            .map(Car::getInsurance)
            .map(Insurance::getName);
```
위 코드는 컴파일 되지 않는다. 그 이유는 `optPerson.map(Person::getCar)`코드가 `Optional<Optional<Car>>`를 리턴하기 때문이다. 그래서 이 문제를 해결하기 위해서는 flatMap을 써야한다. 

스트림의 flatMap은 함수를 인수로 받아서 다른 스트림을 반환하는 메서드다. 보통 인수로 받은 함수를 스트림의 각 요소에 적용하면 스트림의 스트림이 만들어진다. 하지만 flatMap은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다.

```java
Optional<Person> optPerson = Optional.of(person);
String s = optPerson
        .flatMap(Person::getCar)
        .flatMap(Car::getInsurance)
        .map(Insurance::getName)
        .orElse("Unknown");
```

만약 flatMap을 빈 Optional에 호출하면 아무 일도 일어나지 않고 그대로 반환된다. 반면 Optional이 Person을 감싸고 있다면 flatMap에 전달된 Function이 Person에 적용된다. Function을 적용한 결과가 이미 Optional이므로 flatMap 메서드는 결과를 그대로 반환할 수 있다.

cf) 도메인 모델에 Optional을 사용했을 때 데이터를 직렬화 할 수 없는 이유

Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 Serializable 인터페이스를 구현하지 않는다. 따라서 직렬화 모델이 필요하다면 Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식을 권장한다.
```java
public class Person {
  private Car car;
  public Optional<Car> getCarAsOptional() {
    return Optioanl.ofNullable(car);
  }
}
```

**Optional 인스턴스에서 값을 읽을 수 있는 다양한 인스턴스 메서드**

`get()`은 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드다. 값이 있으면 해당 값을 반환하고 없으면 NoSuchElementException을 발생시킨다.

`orElse(T other)`는 Optional이 값을 포함하지 않을 때 디폴트값을 제공할 수 있다.

`orElseGet(Supplier<? extends T> other)`은 Optional에 값이 없을 때만 Supplier가 실행된다. 디폴트 메서드를 만드는데 시간이 오래걸리거나 Optional이 비어있을 때만 디폴트값을 생성하고 싶을 때 사용한다.

`orElseThrow(Supplier<? extends X> exceptionSupplier)`는 Optional이 비어있을 때 예외를 발생시키는 점에서 get 메서드와 비슷하지만 이 메서드는 발생시킬 예외의 종류를 선택할 수 있다.

`ifPresent(Consumer<? super T> consumer)`는 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다. 값이 없으면 아무 일도 일어나지 않는다. 추가적으로 값이 존재하면 true를 반환하고 값이 없으면 false를 반환하는 `isPresent()`메서드도 있다.
```java
public boolean isPresent() {
        return value != null;
    }

public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

**filter로 특정값 거르기**

Optional 객체가 값을 가지며 프레디케이트와 일치하면 filter 메서드는 그 값을 반환하고 그렇지 않으면 빈 Optional 객체를 반환한다. Optional이 비어있다면 filter 연산은 아무 동작도 하지 않는다.

