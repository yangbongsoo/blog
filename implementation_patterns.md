# 켄트 백의 구현패턴
패턴은 반복적으로 일어나는 문제에 대한 합리적인 해결책을 제공해서 프로그래머가 남는 시간과 에너지, 창의력을 진정 독창적인 문제 해결에 사용할 수 있게 해준다.<br>

당장 이득을 얻을 수 있는 패턴을 사용하라. 당장 비용이 들어가지만, 앞으로 이득을 얻을 수 있을지 불확실한 패턴의 경우에는 일단 사용을 자제하는 편이 좋다.
이러한 패턴은 일단 사용을 보류했다가, 정말 필요한 때가 오면 그때 적용하라.<br>

## 클래스
### 추상 클래스와 인터페이스
추상 클래스와 자바 인터페이스의 장단점은 인터페이스 수정의 용이성과 단일 클래스가 여러 인터페이스를 지원할 수 있는지 여부로 귀결된다. <br>

추상 클래스의 단점은 각 클래스가 단 1개의 상위클래스만 지정 가능하다. 인터페이스의 단점은 수정이 용이하지 않다(수정을 하게 되면 그 인터페이스를 구현한 모든 클래스들의
수정이 필요하다). <br>

자바8부터는 디폴트 메서드가 추가됨으로써 인터페이스 수정 문제를 해결했다. 디폴트 메서드를 사용하지 않고 해결하는 또 다른 방법이 있는데 버전 인터페이스를 사용하면 된다. <br>

```java
interface Command {
    void run();
}
```

이 인터페이스가 공개되고 널리 사용된 상황에서 인터페이스를 바꿀 경우, 수정 비용이 엄청날 것이다. 하지만 다음과 같이 버전 인터페이스를 사용해서 그 인터페이스에
undo() 메서드를 추가하면 문제를 해결할 수 있다.

```java
interface ReversibleCommand extends Command {
    void undo();
}
```

이 경우 기존 Command 인터페이스를 사용한 코드는 여전히 잘 동작한다. 새로 선언한 ReversibleCommand 역시 기존 Command가 동작하는 모든 경우에 잘 동작한다. <br>
새로운 기능을 사용하고 싶은 경우에는, 다음과 같이 다운캐스트해서 사용하면 된다.

```java
...
Command recent = ...;

if (recent instanceof ReversibleCommand) {
    ReversibleCommand downcasted = (ReversibleCommand) recent;
    downcasted.undo();
}
```

대게 instanceof를 사용하는 경우, 코드가 특정 클래스에 제한되므로 유연성이 떨어진다. 하지만 이 경우에는 인터페이스를 개선할 수 있었으므로, instanceof의 사용이 정당화된다. <br>
대체 인터페이스는 달갑지 않은 문제에 대한 달갑지 않은 해결책이다. 인터페이스를 사용하면 구현을 쉽게 바꿀 수 있지만, 인터페이스 자체는 바꾸기 매우 어렵다. <br>
대체 인터페이스를 사용하는 것은 자바를 올바르게 사용하는 것이라고 볼 수 없다. 하지만 인터페이스를 확장할 수밖에 없는 경우에 대비해 사용법을 알아두는 편이 좋다. <br>

### 값 객체

```java
class Transaction {
    int value;
    Transaction(int value, Account credit, Account debit) {
        this.value = value;
        credit.addCredit(this);
        debit.addDebit(this);
    }
}
```

위의 Transaction 객체의 경우, 일단 생성된 후에는 값을 변경하는 것이 불가능하다. 더구나 객체 생성 시 생성자는 2개의 Account 객체를 모두 업데이트 해준다. 따라서 이 코드를 보면
거래가 중간에 중단되는 경우나 거래 금액이 나중에 바뀌는 경우를 미리 걱정할 필요가 없다는 것을 알 수 있다. <br>

이렇게 값 스타일 객체(변화하는 상태를 지닌 객체가 아닌 정수와 같은 객체)를 구현하려면, 먼저 상태를 가지고 있는 세계와 값으로만 구성된 세계를 구별해야 한다.
위의 예에서 Transaction은 값 스타일 객체이고, Account는 변화하는 상태를 가진 객체다. 값 스타일 객체에서는 생성자에서만 모든 상태를 설정할 뿐, 다른 경로를 통해서는 필드 값을 변경하면 안된다.
값 스타일의 객체를 다루는 연산은 언제나 새로운 객체를 반환한다. 이러한 객체는 연산을 요청한 쪽에서 저장한다.

```java
bounds.translateBy(10, 20); // 변경 가능한 Rectangle 객체
bounds = bounds.translateBy(10, 20); // 값 스타일 Rectangle 객체
```

하지만 상태로 된 부분과 값으로 된 부분을 구별하기 쉽지 않다.

### 특화
연산 간의 유사점과 차이점을 부각시키는 방향으로 코드를 작성하면, 프로그램을 읽고 사용하고 수정하기 쉬워진다.
하지만 사실 로직과 데이터의 경계가 분명한 것도 아니다. flag는 boolean 데이터지만 제어 흐름에 영향을 미친다. 도우미 객체(HelperClass)는 필드에 저장될 수 있지만 연산에서 사용된다. <br>

하위 클래스, 인터페이스 분리, 내부 클래스, 조건문, 위임 등을 이용해 로직에서 유사점과 차이점을 효과적으로 표현하면 추후 코드 확장성이 한결 좋아진다.

## 상태
### 지역 변수
지역 변수는 현재 사용하는 컬렉션의 원소를 저장하기 위해 사용 될 때도 많다.

```java
for (Clock each : getClocks()) {
    each.setTime(now);
}
```

위의 코드와 같이 each는 현재 원소를 저장하는 명확하고 단순한 지역 변수 이름이다. 어떤 원소인지 궁금한 경우 간단히 for 문을 살펴보면 된다.
중첩 루프의 경우, 컬렉션 이름을 지역 변수 이름 뒤에 붙여서 구별하면 된다.

```java
for (Source eachSender : getSenders()) {
    for (Destination eachReceiver : getReceivers()) {
        ...
    }
}
```

### 수집 파라미터
여러 메서드 호출을 통한 결과를 모으려면 결과를 통합하는 과정이 필요하다. 한 가지 방법은 메서드의 결과값을 반환하는 것이다. 이 방법은 값이 정수처럼 단순한 경우 적합하다.

```java
int size() {
    int result = 1;
    for (Node each : getChildren()) {
        result += each.size();
    }
    return result;
}
```

하지만 단순 덧셈이 아니라 좀 더 복잡한 방식을 통해 결과를 통합해야 하는 경우라면, 파라미터를 전달해서 결과를 수집하는 편이 더 직관적이다.

```java
asList() {
    List results = new ArrayList();
    addTo(results);
    return results;
}

addTo(List elements) {
    elements.add(getValue());
    for (Node each : getChildren()) {
        each.addTo(elements);
    }
}
```

cf) 클린 코더스에서는 Innies not Outies(파라미터는 입력으로 작용하는 거지 output으로 작용하면 안된다) 라고 한다. 위의 예제 코드에 해당하는 말은 아니지만
헷갈릴만한 거라 관련 내용을 추가한다.

```java
private String toSimpleText(Parse table, StringBuffer returnText) {
    if (table.parts == null) {
        SimpleTextOfLeave(table, returnText);
        SimpleTextOfMore(table, returnText);

        retrun returnText.toString();
    }
}
```

위의 코드는 파라미터로 받은 returnText가 다른 메서드의 파라미터로 가서 변경된 후 리턴된다. input으로 들어와서 변경되서 return 되는 구조는 좋지 않다. 로컬 변수로 만들어서 거기에 담고 리턴하는게 좋다.

### 파라미터 객체
여러 개의 파라미터가 함께 여러 메서드로 전달된다면 이들을 묶어서 하나의 객체로 만드는 것을 고려할 수 있다. <br>
cf) 몇개 이상부터 객체로 할것인지 정하는 룰이 필요할 거 같다. <br>

파라미터 객체를 사용하면 코드가 짧아지고, 의도를 좀 더 분명히 전달할 수 있다. 파라미터 객체를 사용하는 주된 이유는 가독성을 높이기 위함이지만, 파라미터 객체는 로직에 있어서 중요한 역할을 하기도 한다.
여러 파라미터 리스트에 데이터가 함께 나타난다는 이야기는 그 데이터들이 밀접한 연관성을 지니고 있다는 명백한 증거다. 여러 필드를 갖고 있는 객체 역시 "이 데이터는 밀접히 연관되어 있음"을 나타낸다.

### 작명
때로 단어를 축약해서 변수 이름을 짓고 싶을 때가 있다. 이 경우, 가독성을 희생하면서 타이핑에서의 효율성을 택하는 것이다. 하지만 한번 작성된 코드는 여러 차례 읽히게 되므로 이는 옳지 못한 방법이다. <br>
저자는 변수 이름을 통해 변수의 역할을 전달한다. 변수에 관한 다른 중요한 정보(생명기간, 범위, 타입)는 일반적으로 문맥을 통해 전달할 수 있다.

## 행위
### 메시지
자바는 주로 메시지를 이용해서 로직을 표현한다. 절차적 언어는 프로시저 호출을 사용해서 정보를 숨긴다.

```java
compute() {
    input();
    process();
    output();
}
```

위의 코드는 "이 연산을 이해하기 위해 알아야 할 것은 3단계이며, 당장 자세한 내용을 알 필요는 없다"라는 뜻을 갖고 있다. 객체 지향 프로그래밍의 좋은 점 중 하나는 같은 프로시저를 사용해도 더 풍부한 내용을
전달할 수 있다는 것이다. 모든 메서드의 구조는 비슷하지만 세부 구현은 다른 메서드가 존재할 수 있다. <br>

메시지를 제어 흐름의 메커니즘으로 사용하면 프로그램에서는 상태의 변화가 중요해진다. 메시지 수신자가 메시지를 받으면, 발신자의 상태는 바뀌지 않지만 수신자의 상태는 바뀔 수 있다.
메시지를 이용하는 프로시저는 "당장 중요하지 않은 세부 구현이 존재한다"라고 하는 대신 "지금 입력에 관해 뭔가 재미있는 일이 벌어지고 있다. 세부 구현은 바뀔 수 있다"라는 의미를 전달해준다.
이러한 유연성을 지혜롭게 사용해서 로직을 명확하고 직접적으로 표현하고, 세부 구현을 적당히 미루는 것은 효과적인 커뮤니케이션을 하는 코드를 삭성하기 위한 중요한 기술이다.

### 선택 메시지
때로 저자는 절차적 언어의 case 문과 같은 목적으로 구현 선택을 위해 메시지를 보낸다.

```java
public void displayShape(Shape subject, Brush brush) {
    brush.display(subject);
}
```

위와 같은 방법으로 런타임에 선택이 일어남을 나타내기 위해 다형적 메시지를 사용한다. display() 메시지는 런타임에 Brush의 타입에 따라 구현을 선택한다.

```java
public interface Brush {
    display();
}

public class PostscriptBrush implements Brush {
    display() {
        ...
    }
}

public class ScreenBrush implements Brush {
    display() {
        ...
    }
}
```

선택 메시지를 사용하면 명시적 조건문의 사용을 크게 줄일 수 있으며 추후 확장이 쉽다. 프로그램에서 명시적 조건문을 사용한 부분은 나중에 전체 프로그램의 행위를 바꿀 때마다 명시적으로 수정해야 한다. <br>

선택 메시지를 많이 사용하는 프로그램을 이해하기 위해서는 약간의 기술이 필요하다. 코드 독자가 연산의 세부 구현을 이해하기 위해 여러 개의 클래스를 살펴봐야 할 수도 있다.
과도한 선택 메시지 사용은 좋지 않다. 당장 연산의 변형이 필요하지 않는 경우라면 미래 확장을 위해 굳이 선택 메시지를 사용할 필요는 없다. <br>

### 더블 디스패치
선택 메시지를 사용하면 일차원적 변형을 잘 표현할 수 있다. 위에서 살펴본 예에서는 Shape를 그릴 Brush를 변형할 수 있었다. 하지만 두 가지의 독립적인 차원에서의 변형을 표현하기 위해서는 2개의 선택 메시지를
직렬(cascade)로 연결해야 한다. <br>

예를 들어 PostscriptBrush로 출력할 타원과 ScreenBrush로 출력할 직사각형의 연산 과정 차이를 표현하고 싶다고 하자.
우선 어느 곳에서 연산을 해야 하는지 결정해야 한다. 기본 연산은 Brush에서 하는 것이 더 적당해 보이므로 선택 메시지를, 먼저 Shape에 보낸 후 Brush로 보내자. <br>

```java
displayShape(Shape subject, Brush brush) {
    subject.displayWith(brush);
}
```

이제 각 Shape는 다른 방식으로 displayWith()를 구현할 수 있다. 하지만 세부 구현을 하는 대신 자신의 타입을 Brush에 넘겨주자.

```java
Oval.displayWith(Brush brush) {
    brush.displayOval(this);
}

Rectangle.displayWith(Brush brush) {
    brush.displayRectangle(this);
}
```

이제 각 Brush는 경우에 따라 어떤 일을 해야 할지 알 수 있다.

```java
PostscriptBrush.displayRectangle(Rectangle subject) {
    writer print(subject.left() + " " + ... + " rect);
}
```

더블 디스패치를 사용하면 유연성을 조금 잃게 되며, 코드 중복이 발생한다. 우선 첫째 선택 메시지의 수신자 타입 이름이 둘째 선택 메시지의 수신자 메서드에 들어간다.
위 예의 경우, 새로운 Shape를 추가하기 위해서는 모든 종류의 Brush의 메서드를 수정해야 한다. 두 가지 요소 중 변경될 가능성이 높은 요소를 둘째 선택 메시지의 수신자로 하라.

### 더블 디스패치 (토비 강의)

```java
public class StaticDispatch {
	static class Service {
		void run() {
			System.out.println("run1");
		}

		void run(String msg) {
			System.out.println("run(" + msg + ")");
		}
	}

	public static void main(String[] args) {
		new Service().run();
		new Service().run("hello");
	}
}
```

스태틱 디스패치는 컴파일 타임에 어느 메서드가 호출될지 정확히 알고 있다. <br>

```java
public class DynamicDispatch {
	static abstract class Service {
		abstract void run();
	}

	static class MyService1 extends Service {
		@Override
		void run() {
			System.out.println("run1");
		}
	}

	static class MyService2 extends Service {
		@Override
		void run() {
			System.out.println("run2");
		}
	}

	public static void main(String[] args) {
	    Service svc = new MyService1(); // receiver parameter
	    svc.run();
	}
}
```

위의 코드는 컴파일 시점에 똑같은 이름으로 정의되어 있는 두개의 run 메서드 중 어느거를 실행할건지가 결정이 되어있지 않다(다이나믹 디스패치). <br>
svc 변수의 타입이 Service 추상 클래스이기 때문에 컴파일 시점에서는 결정할 수 없다. 런타임 시점에 svc에 할당되어 있는 오브젝트가 뭔가를 보고 걔에 의해 결정한다. <br>
메서드 호출과정에 첫번째로 들어가있는게 receiver parameter이다. this에 해당하는 오브젝트가 receiver parameter에 들어가 있다. 걔를 보고 런타임 시점에 결정한다. <br>

더블 디스패치는 다이나믹 디스패치를 두 번 하는 경우다. <br>

```java
public class DoubleDispatch {
	interface Post {
		void postOn(SNS s);
	}

	static class Text implements Post {
		@Override
		public void postOn(SNS s) {
			s.post(this);
		}
	}

	static class Picture implements Post {
		@Override
		public void postOn(SNS s) {
			s.post(this);
		}
	}

	interface SNS {
		void post(Text t);
		void post(Picture p);
	}

	static class Facebook implements SNS {

		@Override
		public void post(Text t) {
			System.out.println("text - facebook");
		}

		@Override
		public void post(Picture p) {
			System.out.println("picture - facebook");
		}
	}

	static class Twitter implements SNS {

		@Override
		public void post(Text t) {
			System.out.println("text - twitter");
		}

		@Override
		public void post(Picture p) {
			System.out.println("picture - twitter");
		}
	}

	public static void main(String[] args) {
		List<Post> posts = Arrays.asList(new Text(), new Picture());
		List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

		for (Post p : posts) {
			for (SNS s : sns) {
				p.postOn(s);
			}
		}

		// 람다 표현식
		posts.forEach(p -> sns.forEach((SNS s) -> p.postOn(s)));
	}
}
```

유튜브 영상 :  https://www.youtube.com/watch?v=s-tXAHub6vg

### 되돌림 메시지
대칭성을 이용하면 코드의 가독성을 높일 수 있다. 다음 코드를 보자.

```java
void compute() {
    input();
    helper.process(this);
    output();
}
```

3개의 메서드로 구성되는 위의 메서드는 대칭성이 떨어진다. 잠재적인 대칭성을 드러내는 도우미 메서드를 사용하면 가독성을 높일 수 있다.
그렇게 되면 compute() 메서드를 보면서 누가 메시지를 받는지(언제나 this에게 전달되지만) 고민할 필요가 없다.

```java
void process(Helper helper) {
    helper.process(this);
}

void compute() {
    input();
    process(helper);
    output();
}
```

이제 독자는 쉽게 compute() 메서드의 구조를 이해할 수 있다. <br>

때로는 되돌림 메시지를 통해 호출되는 도우미 메서드는 그 자체로 중요하다. 하지만 되돌림 메시지를 과도하게 사용되면 구현을 다른 곳으로 옮겨야 할 필요성이 희석되어 버린다.
다음 코드를 보자.

```java
void input(Helper helper) {
    helper.input(this);
}

void output(Helper helper) {
    helper.output(this);
}
```

위의 코드는 아래와 같이 compute() 메서드 전체를 Helper 클래스로 옮기는 것이 나을 수도 있다.

```java
compute() {
    new Helper(this).compute();
}

Helper.compute() {
    input();
    process();
    output();
}
```

때로는 대칭성과 같은 "미학적" 만족만을 위해 새로운 메서드를 만드는 것이 의미 없게 느껴질 때도 있다. 하지만 미학은 생각보다 중요한 것이다. 미학은 엄격한 선형적 논리 사고보다 더 많은 두뇌 활동을 요구한다.
여러분이 코드의 미학에 대한 자신의 식견을 높이게 되면, 자신의 코드에서 받는 미학적 느낌을 통해 코드의 품질을 평가할 수 있게 된다. 코드에서 받는 느낌은 이미 유용성이 알려져 있는 다른 패턴만큼이나 중요하다. <br>

### 설명 메시지
sw 개발에서 개발자의 의도와 구현을 구분하는 것은 언제나 중요하다. 독자는 이를 통해 먼저 연산의 핵심을 파악할 수 있고, 필요한 경우에만 세부 구현에 관심을 기울이면 된다.
메시지를 사용하면 먼저 풀려는 문제의 이름을 반영하는 메시지를 보낸 후, 문제 푸는 방식을 반영하는 메시지를 보내서 이 구분을 명시할 수 있다.

```java
highlight(Rectangle area) {
    reverse(area);
}
```

highlight()는 프로그래머의 의도를 전달하기 위한 메서드이다. 메서드의 이름에는 해결하려는 문제(스크린의 하이라이트할 부분을 나타내는것)를 반영했다.
그리고 그 메서드 안에서 문제를 푸는 세부 구현 메서드가 있다. <br>

한 줄로 된 코드에 주석을 붙이고 싶은 경우라면 설명 메시지 사용을 고려하라.

```java
flags |= LOADED_BIT; // 로드 비트를 설정
```

이 코드는 다음과 같이 바꾸는 편이 좋다.

```java
setLoadedFlag();

void setLoadedFlag() {
    flags |= LOADED_BIT;
}
```

setLoadedFlag()의 구현이 하는 일은 거의 없다. 하지만 이러한 한 줄 짜리 메서드는 커뮤니케이션을 돕는다. <br>

### 보호절
프로그램에는 주요 흐름이 있지만 때로는 주요 흐름에서 벗어나야 할 때가 있다. 보호절을 사용하면 간단한 지역적 예외 상황을 지역적인 변화만을 수반하며 표현할 수 있다. 다음 두 가지 코드를 비교해보자.
```java
void initialize() {
    if (!isInitialize()) {
        ...
    }
}

void initialize() {
    if(isInitialize()) {
        return;
    }

    ...
}
```
첫째 코드를 읽을 때는 then에 해당하는 코드를 보면서 else 조건을 찾게 된다. 독자는 마음 속에서 조건을 스택에 넣어 놓는다. 이런 모든 요소들이 then 내부의 코드를 읽을 때 방해 요소가 된다.
반면 둘째 코드의 경우 두줄만 읽으면 수신자가 초기화되지 않았다는 사실을 금방 알 수 있다. <br>

if-then-else는 동등한 중요성을 갖고 있는 제어 흐름을 표현한다. 반면 보호절은 한 쪽의 제어 흐름이 다른 쪽보다 중요한 경우 유용하다. <br>

보호절은 여러 개의 조건이 있을 경우 특히 유용하다.

```java
void compute() {
    Server server = getServer();
    if (server != null) {
        Client client = server.getClient();
        if (client != null) {
            Request current = client.getRequest();
            if (current != null) {
                processRequest(current);
            }
        }
    }
}
```

중첩된 조건을 사용하면 코드에 문제가 발생할 확률이 높다. 보호절을 사용하면 복잡한 제어 구조를 사용하지 않고도 같은 코드를 표현 할 수 있다.

```java
void compute() {
    Server server = getServer();
    if (server == null)
        return;

    Client client = server.getClient();
    if (client == null)
        return;

    Request current = client.getRequest();
    if (current == null)
        return;

    processRequest(current);
}
```

보호절의 다른 형태는 루프 내에서 continue를 사용하는 것이다. "이 경우는 신경 쓰지 말고 다른 경우를 보세요" 라는 의미다.

```java
while (line = reader.readline()) {
    if (line.startsWith('#') || line.isEmpty()) {
        continue;
    }

    // 보통 경우 처리
}
```

다시 말하지만 보호절의 포인트는 보통 경우와 예외 경우 처리의(지역적) 차이점을 부각시키는 것이다.

## 메서드

### 조합 메서드
실제 읽기 좋은 메서드의 길이는 5 ~ 15줄 사이다. <br>

독자는 메서드의 길이에 영향을 받는 다양한 문제를 해결해야 한다. 전체 구조를 읽을 때는 긴 메서드가 좋다. 하지만 코드 세부 사항을 이해하려 할 때, 긴 메서드는 오히려 방해가 된다. <br>

메서드를 구성할 때는 추측이 아닌 사실에 근거하라. 일단 동작하는 코드를 만들고 구성 방식을 결정하라. 코드 구성에 미리 시간을 들이게 되면, 구현 도중 새로 알게 되는 사실 때문에 했던 일들을 번복하는 경우가 많다. <br>

### 의도 제시형 이름
메서드 이름을 통해서는 메서드 의도를 전달하고 그 외 정보는 다른 방식으로 전달하는 것이 좋다.

```java
Customer.linearCustomerSearch(String id);
```

구현 전략은 메서드 이름에 가장 자주 들어가는 부가 정보다.

```java
Customer.find(String id);
```

하지만 메서드에 대한 정보를 전달한다는 점에서 위 코드가 더 낫다. 프로그래머가 갖고 있는 모든 정보를 당장 전달하는 것이 능사는 아니다. 구현 전략(선형탐색, 해쉬탐색 등)이 사용자에게 중요한 문제가 아니라면
메서드 이름에서 빼라. <br>

메서드의 이름을 지을 때는 그 메서드를 호출하는 입장에서 생각해보라. 왜 이 메서드를 사용해야 하는가? 메서드 이름에서 이 질문에 대한 답을 얻을 수 있어야 한다. 메서드 이름은 그 메서드를 호출하는 코드가 표현하려 하는 바에
도움을 줄 수 있도록 지어야 한다. <br>

### 메서드 객체
메서드 객체는 복잡하게 꼬여 있는 메서드를 읽기 쉽고 명확하면서도 세부 구현 전달이 쉽도록 바꿔준다. 이 패턴은 일단 동작하는 코드가 나온 후에 사용하는 것이 보통이며, 코드가 복잡할수록 효과가 크다.

먼저 메서드 이름을 따서 클래스 이름을 정한다.

```java
complexCalculation() {
    new ComplexCalculator().calculate();
}
```

그리고 메서드에서 사용하는 각 파라미터, 지역변수, 필드에 대해 새로운 객체상의 필드를 생성하고 클래스 생성자를 통해 주입받는다. <br>

메서드에 있던 로직들을 새로운 클래스의 메서드로 옮기고, 새로운 메서드를 호출한다. <br>

본래 메서드에서 필드를 설정하는 부분이 있었다면 다음과 같이 calculate()가 반환된 후에 설정해준다.

```java
complexCalculation() {
    ComplexCalculator calculator = new ComplexCalculator();
    calculator.calculate();
    mean = calculator.mean;
    variance = calculator.variance;
}
```

이제 본격적으로 재미있는 부분이 시작된다. 새로운 클래스의 코드는 리팩토링하기 쉽다. 메서드의 일부를 새로운 메서드로 분리하더라도 이제 모든 데이터가 필드에 저장되어 있으므로, 파라미터를 사용할 필요가 없기 때문이다. <br>

### 메서드 반환 타입
메서드 반환 타입은 메서드가 프로시저인지 특정 타입의 객체를 반환하는 함수인지 구별해준다. void 반환 타입을 통해 별도의 키워드 없이 프로시저를 함수와 구별할 수 있다. <br>

함수를 작성할 때는 의도를 나타내는 반환 타입을 사용하라. 가급적 메서드의 적용 범위를 넓히기 위해서 의도를 드러낼 수 있는 가장 추상적인 타입을 사용하라.
이렇게 하면 이후 구체적인 반환 클래스의 타입을 유연하게 변경할 수 있다.

### 공장 메서드
공장 메서드를 사용하면 의도가 담긴 별도의 이름을 가질 수 있고 추상 타입을 반환할 수 있다.
하지만 복잡성이 증가하므로, 공장 메서드는 이득이 있을 경우(객체를 캐쉬에 저장해 놓거나 런타임에 타입이 결정되는 하위클래스를 반환하는 등)에만 사용해야 한다. <br>

코드를 읽을 때, 공장 메서드를 볼 때마다 객체 생성 이외에 어떤 작업이 일어나는가 하는 의문을 갖게 된다. 따라서 코드 독자의 시간을 아껴주고 싶다면, 평범한 객체 생성을 하는 경우에는 생성자를 사용하라.
객체 생성 이외에 다른 의도가 있을 경우에만 공장 메서드를 사용하라. <br>

cf) 이펙티브에서는 정적 팩토리 메서드(static factory method)라 부르고 생성자 대신 사용할 수 없는지 먼저 고려해보라고 조언 한다. <br>

### 컬렉션 접근자 메서드
컬렉션을 갖고 있는 객체가 있다고 해보자. 컬렉션에 대한 접근을 어떻게 제공할 것인가?

```java
List<Book> getBooks() {
    return books;
}
```

이렇게 하면 사용자는 최대의 유연성을 얻게 되지만 여러 가지 문제가 발생한다. 사용자가 컬렉션을 직접 조작하게 되므로, 컬렉션 데이터에 의존적인 객체 내부 상태가 유효하지 않게 될 수 있다. <br>

한 가지 해법은 컬렉션을 반환하기 전에 수정할 수 없는 컬렉션 형태로 바꿔서 반환하는 것이다.

```java
List<Book> getBooks() {
    return Collections.unmodifiableList(books);
}
```

하지만 이 방법은 또 다른 문제를 낳는다. 누군가가 컬렉션을 수정하려고 하면 예외가 발생하는데, 이런 문제를 디버깅하는 데 비용이 많이 든다. <br>

이런 방법 대신 컬렉션에 대한 제한적이지만 의미 있는 접근을 제공하는 메서드를 사용하라. 사용자가 컬렉션 원소를 하나씩 접근해야 한다면, 다음과 같이 Iterator를 반환하는 메서드를 제공하라.

```java
Iterator getBooks() {
    final Iterator<Book> reader = books.iterator();
    return new Iterator<Book>() {
        public boolean hasNext() {
            return reader.hasNext();
        }

        public Book next() {
            return reader.next();
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }
    }
};
```

이렇게 되면 클라이언트에서 Iterator에 remove() 메서드를 호출하지만 않는다면, 컬렉션이 수정되는 것을 막을 수 있다.

## 발전하는 프레임워크
프레임워크를 개발하고 유지하는 데 있어 근본적인 딜레마는 프레임워크를 지속적으로 발전시켜야 하지만, 기존 클라이언트 코드는 계속해서 동작하도록 해야 한다는 것이다. <br>

하지만 호환성을 유지하는 업그레이드가 항상 가능한 것은 아니다. 호환성을 유지하기 위해서는 때로 프레임워크의 복잡도가 올라가기도 한다. 어떤 경우에는 완벽한 호환성을 유지하기 위한 비용이
클라이언트에 제공하는 가치보다 큰 경우도 있다. <br>

### 객체 (사용 스타일)
프레임워크를 객체로 나타낼 때 프레임워크는 인스턴스화(instantiation), 설정(configuration), 구현(implementation)의 세 가지 주요 스타일을 지원한다.
각 스타일은 사용의 편의성, 유연성, 안정성 면에서 서로 다른 모습을 보인다. <br>

가장 단순한 스타일은 인스턴스화이다. 서버 소켓을 사용하고 싶다면 `new ServerSocket()`을 사용하면 된다. 인스턴스가 생성되면 해당 객체에 대해 메서드를 호출하는 방식으로 프레임워크를 사용한다.
클라이언트가 로직의 변형을 필요로 하지 않고 데이터의 변형만을 필요로 하는 경우 인스턴스화를 사용하라. <br>

설정은 인스턴스화보다 복잡하지만 좀 더 유연하다. 클라이언트는 프레임워크 객체를 생성한 후 자신만의 객체를 프레임워크에 전달해서 프레임워크에서 사용되도록 한다. 예를 들어 TreeSet은 클라이언트에서 정의하는
Comparator를 사용해서 원소 사이의 정렬을 수행한다.

```java
Comparator<Author> byFirstName = new Comparator<Author>() {
    public int compare(Author book1, Author book2) {
        return book1.getFirstName().compareTo(book2.getFirstName());
    }
};

SortedSet<Author> sorted = new TreeSet<Author>(byFirstName);
```

설정은 데이터뿐 아니라 로직의 변형도 지원하므로 인스턴스화보다 유연하다. 그러나 프레임워크 입장에서는 계속해서 같은 인터페이스(Comparaotr를 받는 기능)를 사용해야 하므로 발전시키는 데 제한이 생기고 클라이언트 코드에 대한 호환성을 보장해주기 어려워진다.
또한 실질적으로 표현할 수 있는 변형의 종류에도 제한이 있다. 예를 들어 어떤 객체 설정에 2개 이상의 옵션이 들어가는 경우 클라이언트가 쉽게 사용할 수 없을 것이다.<br>

클라이언트가 설정에서 제공하는 것 이상으로 많은 종류의 로직의 변형을 필요로 하는 경우, 구현에 의한 사용을 제공할 수 있다. 구현은 미래에 있을 설계 변경을 가장 크게 제한한다.
호환성을 보장하기 위해서는 프레임워크에서 제공하는 상위클래스나 인터페이스를 모두 그대로 유지해야 하기 때문이다. 앞에서 살펴본 Comparator는 구현을 사용한 프레임워크의 간단한 예이다.
byFirstName은 컬렉션 프레임워크 추상화(이 경우는 클래스)의 구현이다. <br>

프레임워크 추상화에서 세부 사항을 노출하는 것은 양날의 검과 같다. 이를 통해 클라이언트는 자신의 코드를 사용할 수 있게 되지만, 프레임워크 개발자는 기존 클라이언트 코드와의 호환성을 포기하지 않는 이상 앞으로도 같은 인터페이스를 계속
사용해야 하기 때문이다.