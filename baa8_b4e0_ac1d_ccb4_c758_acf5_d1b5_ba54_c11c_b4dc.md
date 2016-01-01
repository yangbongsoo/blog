# 모든 객체의 공통 메서드
Object에 정의된 비-final 메서드(equals, hashCode, toString, clone, finalize)의 명시적인 일반 규약들에 대해서 알아보자.(종료자 finalize는 제외) 추가로 Object의 메서드는 아니지만 특성이 비슷한 Comparable.compareTo도 알아보자.

###규칙8 : equals를 재정의할 때는 일반 규약을 따르라
**equals를 재정의하지 않아도 되는 경우**
1. 각각의 객체가 고유하다.
    1. 값(value) 대신 활성 개체(active entity)를 나타내는 Thread 같은 클래스가 이 조건에 부합.
2. 클래스에 논리적 동일성 검사 방법이 있건 없건 상관없다. 
    1. Random 클래스는 equals 메서드가 큰 의미 없다.  
3. 상위 클래스에서 재정의한 equals가 하위 클래스에서 사용하기에도 적당하다. 
4. 클래스가 private 또는 package-private로 선언되었고, equals 메서드를 호출할 일이 없다. 
    1. 하지만 저자는 재정의해서 `throw new AssertionError();`를 선언하라고 한다. 



