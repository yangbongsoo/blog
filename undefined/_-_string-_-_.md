# 왜 자꾸 String을 쓰지 말라는거야?

## StringBuffer 클래스와 StringBuilder 클래스

StringBuilder 클래스는 JDK 5.0에서 새로 추가되었다. StringBuffer 클래스가 제공하는 메서드와 동일하다. StringBuffer 클래스는 스레드에 안전하게\(ThreadSafe\) 설계되어 있으므로, 여러 개의 스레드에서 하나의 StringBuffer 객체를 처리해도 전혀 문제가 되지 않는다. 하지만 StringBuilder는 단일 스레드에서의 안전성만 보장한다. 그렇기 때문에 여러 개의 스레드에서 하나의 StringBuilder 객체를 처리하면 문제가 발생한다.

StringBuffer를 기준으로 생성자와 메서드를 확인하고 정리해 보자.

```java
// 생성자
    public StringBuffer() {
        super(16); //기본 용량은 16개의 char다
    }

    public StringBuffer(int capacity) {
        super(capacity);
    }

    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str); // str 값을 갖는 StringBuffer를 생성한다
    }

    // CharSequence를 매개변수로 받아 그 seq 값을 갖는 StringBuffer를 생성한다
    public StringBuffer(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }
```

cf\) CharSequence는 인터페이스다. 구현체로는 CharBuffer, String, StringBuffer, StringBuilder가 있으며 StringBuffer나 StringBuilder로 생성한 객체를 전달할 때 사용된다.

```java
    @Test
    public void stringTest() throws Exception {
        StringBuilder sb = new StringBuilder();
        sb.append("ABCDE");
        check(sb);
    }

    private void check(CharSequence cs) {
        StringBuffer sb = new StringBuffer(cs);
        System.out.println(sb.length()); // 5
    }
```

StringBuffer나 StringBuilder로 값을 만든 후 굳이 toString을 수행하여 필요 없는 객체를 만들어서 넘겨주기보다는 CharSequence로 받아서 처리하는 것이 메모리 효율에 더 좋다.

주로 사용하는 메서드는 append\(\)와 insert\(\)다.

```java
    @Test
    public void stringTest() throws Exception {
        StringBuffer sb = new StringBuffer();
        sb.append("ABC");
        sb.insert(1,"123");
        System.out.println(sb); // A123BC
    }
```

**많은 양의 문자열을 연산 할때 String을 쓰지 말아야 하는 이유**는 String은 immutable 객체\(생성 후 변경 불가한 객체\)이기 때문이다. 그래서 연산할 때마다 새로운 String 클래스 객체가 만들어지고 이전 객체는 필요 없는 쓰레기 값이 되어 GC 대상이 된다. 이런 작업이 반복 수행되면서 메모리를 많이 사용하게 되고, 응답속도에도 많은 영향을 미치게 된다. 반면에 StringBuffer나 StringBuilder는 새로운 객체를 생성하지 않고, 기존에 있는 객체의 크기를 증가시키면서 값을 더한다.

cf\) JDK 5.0 이상을 사용하면 아래와 같이 StringBuilder로 변환된다. 개발자의 실수를 어느 정도는 피할 수 있게 된 것이다.

```java
public class VersionTest {
    String str = "Here" + "is " + "a " + "sample.";
    public VersionTest() {
        int i = 1;
        String str2 = "Here " + "is " + i + " sample.";
    }
}

// 역 컴파일한 소스
public class VersionTest {
    public VersionTest() {
        str = "Here is a sample.";
        int i = 1;
        String str2 = (new StringBuilder("Here is "))
            .append(i).append(" sample.").toString();
    }
    String str;
}
```

