---
title: 싱글톤 패턴 (Singleton Pattern)
date: 2024-07-02 20:45:00 +0900
categories: [DesignPattern]
tags: [design-pattern, gof, singleton]
---

## 싱글톤 패턴?

클래스의 인스턴스를 **단 하나만 생성하도록 보장**하는 패턴을 말한다. 어플리케이션에 특정 객체가 단 하나만 존재해야 하고, 이 객체에 전역으로 접근해야할 때 사용한다.

쉽게 말하면, **객체 딱 하나만 만들어두고서 인스턴스가 필요할 때 새로 만들지 않고 기존에 만들어두었던 것을 가져와 활용하는 기법**인 것!

### 오, 전역 변수랑 비슷하다!

똑같은 데이터를 메소드마다 지역 변수로 선언해 사용하면 낭비이기 때문에, 전역으로 하나만 데이터를 선언해두고 가져다가 사용하면 효율적이기 때문에 전역 변수를 사용하지 않는가?

이런 개념을 클래스에 대입한 게 싱글톤 패턴이라고 보면 된다 😀

아무튼, 그래서 보통 리소스를 많이 차지하는 무거운 클래스를 사용해야 할 때 싱글톤 패턴을 사용하곤 한다.

예를 들면, **데이터베이스 연결 모듈**을 들 수 있다.

데이터베이스에 접속하는 것은 리소스가 많이 드는 작업이다. 만약 데이터베이스에 요청을 보낼 때마다 이 모듈을 가지고 접속하는 작업을 수행한다면, 속도가 느린 것은 물론이고 리소스가 매우! 낭비될 것이다. 

그래서 이런 경우, 싱글톤 패턴을 적용해 유일한 객체로 만들어 사용하면 좋다.

## 싱글톤 패턴 구현하기

싱글톤 패턴은 단 하나의 객체만 만들어 사용하는 것이라고 했다. 그러면 어떻게 딱 하나의 객체(인스턴스)만을 만들어 전역에서 접근할 수 있을까?

간단히 이렇게 생각하면 된다.

1. 싱글톤으로 이용할 클래스를 다른 곳에서 생성해서 사용할 수 없도록 해당 **클래스의 생성자 메소드에 private 키워드를 붙여준다.** 
2. **`getInstance()` 메소드를 통해** 미리 생성한 객체를 가져와 사용한다.

싱글톤 패턴을 구현하는 방법에는 여러가지가 있다. 하나씩 살펴보자.

### 1. Eager Initialization

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {
        // 외부에서 생성할 수 없도록 생성자를 private으로 선언
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

- **클래스가 로드되는 시점에 인스턴스를 생성**한다. *싱글톤을 구현하는 가장 간단한 방법이다.*
- 클래스 로딩 시점에 인스턴스를 생성하므로, 멀티 스레드 환경에서도 안전하다.
- 객체를 사용하지 않더라도 무조건 생성되기 때문에, **리소스 낭비**가 있을 수 있고, **Exception 처리를 할 수 없다**는 단점이 있다.

### 2. Static Block Initialization

```java
public class Singleton {
    private static final Singleton instance;

    static {
        try {
            instance = new Singleton();
        } catch (Exception e) {
            throw new RuntimeException("싱글톤 객체 생성 실패");
        }
    }

    private Singleton() {
        // 외부에서 생성할 수 없도록 생성자를 private으로 선언
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

- static 블록을 사용해 클래스 로드 시점에 인스턴스를 생성하는 방식이다.
*( static 블록 : 클래스가 처음 로딩될 때 딱 한 번만 실행되는 블록 )*
- Eager Initialization 방식과 유사하고, Exception 처리가 가능하다는 장점이 있다.
- 하지만 마찬가지로 **리소스 낭비**가 있을 수 있다는 단점은 여전히 존재한다.

### 3. Lazy Initialization

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // 외부에서 생성할 수 없도록 생성자를 private으로 선언
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

- 만들어 둔 인스턴스가 **없으면 생성**하고, 만들어 둔 인스턴스가 **있으면 해당 인스턴스를 반환**하는 방법이다.
- 클래스가 처음 사용되기 전까지 인스턴스를 생성하지 않기 때문에, 메모리 사용이 최적화된다.
- **멀티스레드 환경에서 싱글톤이 보장되지 않는다**는 큰 문제가 있다.
    
    ---
    
    ![image](/assets/img/post/design-pattern/240702_싱글톤-패턴/screenshot_01.png){: width='350'}
    
    1. A라는 스레드가 `getInstance()`를 호출한다. 생성된 인스턴스가 없기 때문에, 인스턴스 생성을 위해 if문 안으로 들어가있는 상태이다.
    2. 이 때 B라는 스레드가 `getInstance()`를 호출한다. A 스레드가 아직 인스턴스를 생성하기 전이기 때문에, if문 안으로 들어간다.
    3. 이 과정을 거치면 A가 반환받은 인스턴스와 B가 반환받은 인스턴스는 **서로 다르다.**    
    *( = **싱글톤인데 객체가 두 개가 되어버린다.** A, B 모두 각각 인스턴스를 새로 생성했기 때문! )*
    
    ---
    
- **싱글 스레드 애플리케이션**이거나, **초기화 시점이 중요한 경우**에 적합한 방식이다.

### 4. Thread-Safe Initialization

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // 외부에서 생성할 수 없도록 생성자를 private으로 선언
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

- 멀티 스레드 환경에서도 안전할 수 있도록, **synchronized 키워드**를 이용해 `getInstance()` 메소드를 동기화하는 방법이다.
    > 💡 synchronized 키워드를 사용하게 되면, **메소드를 사용하고 있는 스레드를 제외한 나머지 스레드가 해당 메소드에 접근할 수 없도록 막아준다.** 그래서 멀티 스레드 환경에서 동시성 문제를 해결할 수 있다.
    
- `getInstance()` 메소드를 실행할 때마다 불필요한 락이 걸리기 때문에, 리소스 낭비가 발생하게 되는 문제가 있다. 
*(인스턴스 생성되고 나서는 synchronized 키워드가 사실상 필요 없음)*

### 5. Double-Checked Locking

```java
public class Singleton {
    private static **volatile** Singleton instance;

    private Singleton() {
        // 외부에서 생성할 수 없도록 생성자를 private으로 선언
    }

    public static Singleton getInstance() {
        if (instance == null) {
            **synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }**
        }
        return instance;
    }
}
```

- **volatile** 키워드를 이용하여 **리소스 낭비가 발생**하는 문제점을 해결하기 위한 방법이다.
    
    > 💡 **volatile 키워드**<br>
    Java에서는 스레드를 여러 개 사용하는 경우, 성능을 위해 각각의 스레드는 변수를 메인 메모리(RAM)으로부터 가져오는 것이 아니라 캐시 메모리에서 가져오게 된다.<br>
    이 때 **비동기로 변수 값을 캐시에 저장하다가, 각 스레드에 할당된 캐시 메모리의 변수 값이 일치하지 않을 수 있다는 문제**가 발생한다.<br>
    그래서 **volatile 키워드가 붙어있는 변수는 캐시에서 읽지 말고, 메인 메모리에서 읽어오도록 지정**해준다.
    
- 생성된 인스턴스가 있는 경우 synchronized 블럭이 실행되지 않고, 만들어져 있는 인스턴스를 곧장 반환한다.
- 다음과 같은 단점이 존재하기 때문에, 사용을 권장하지 않는다.
    - volatile 키워드가 JDK 1.5 이상에서만 동작한다.
    - JVM에 따라서 스레드 세이프 하지 않은 문제가 발생할 수 있다.
    - 자바의 메모리 관리 방법에 대한 이해가 필요하다.

### 6. Bill Pugh Singleton Design

```java
public class Singleton {
    private Singleton() {
        // 외부에서 생성할 수 없도록 생성자를 private으로 선언
    }

    private static class SingletonHelper {
        private static final Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHelper.instance;
    }
}
```

- Inner Static Helper Class를 이용하는 방법이다. *( **권장되는 방법 중 하나** )*
- 이 방식은 다음과 같이 동작한다.
    1. Helper 클래스는 static으로 선언했기 때문에, Singleton 클래스가 로딩될 때는 메모리에 로딩되지 않는다.
    2. 어딘가에서 `getInstance()` 메소드를 호출하면, Helper 클래스에 static으로 선언한 인스턴스를 가져와 리턴해주는데, 이때 클래스가 한 번만 초기화되며 싱글톤 객체를 생성한다.
    3. final로 선언해두었기 떄문에, 싱글톤 객체 선언 후 값이 변경되지 않는다.
- 클라이언트가 임의로 싱글톤을 깨트릴 수 있다는 단점이 있다. 
*(Reflection, 직렬화/역직렬화를 통해 가능)*

### 7. Enum Singleton

```java
public enum Singleton {
    INSTANCE;

    public void someMethod() {
        // method implementation
    }
}
```

- Enum을 이용해 싱글톤을 구현하는 방법이다. *( 권장되는 방법 중 하나 )*
- Enum 내에서 상수 뿐만 아니라, 변수나 메소드를 선언해 사용 가능하기 때문에, 이를 응용해 싱글톤 클래스처럼 사용하는 방법이다.
- Enum으로 구현한 싱글톤 클래스를 일반 클래스로 마이그레이션 해야 하는 경우라면, 처음부터 **코드를 다시 구현해야 하는 단점**이 있다. 또한, **Enum 외의 클래스를 상속할 수 없다**는 단점도 있다.
    

> 💡 **Enum과 싱글톤의 관계**<br>
Enum은 고정된 상수들의 집합이다. 그래서 런타임*(run-time)*이 아닌 컴파일 타임*(compile-time)*에 모든 값을 알고 있어야 하는 규칙이 있다.<br>
즉, **다른 패키지나 클래스에서 enum 타입에 접근해 변수처럼 동적으로 값을 할당하는 행위는 금지**된 것이다.<br>
이 때문에 **enum 객체의 생성자는 private**으로 설정해야 하고, 이렇게 하면 외부에서 생성자에 접근이 불가능하므로 enum은 실질적으로 final 클래스와 같아진다.<br>
**위와 같은 특성 덕분에 enum 타입은 싱글톤을 구현하는 하나의 방법으로 사용될 수 있는 것이다 😄**


## 싱글톤 깨트리기

위에서 잠깐 언급했던 것처럼, Bill Pugh 방식은 **Reflection, 직렬화/역직렬화**를 통해 클라이언트가 싱글톤을 깨트릴 수 있다고 했다.

그러면 각각의 방식으로 어떻게 싱글톤이 깨질 수 있는지 알아보자.

### Reflection 이용

---
<details>

<summary>&nbsp;&nbsp;<b>ℹ️&nbsp;Reflection</b></summary>

<div style='padding-left: 12px; padding-right: 12px; padding-top: 10px'>
    <p>
    Java에서는 Class 객체를 이용하면 클래스에 대한 모든 정보<i>(클래스에 정의된 멤버의 이름이나 갯수 등)</i>를 런타임에 코드 로직으로 얻을 수 있다.
    </p>
    <p>
        이러한 정보를 이용해 오로지 Class 객체만으로 본 클래스를 인스턴스화 하거나, 메서드를 호출하는 등의 보다 동적인 코드를 작성할 수 있게 된다.
    </p>
    <p>
        이렇게 <b>구체적인 클래스 타입을 알지 못해도, 그 클래스의 정보에 접근할 수 있게 해주는 자바 기법</b>을 Reflection API라고 한다.
    </p>
    <p>
        Reflection은 객체를 통해 클래스의 정보를 분석하여 런타임에 클래스의 동작을 검사하거나 조작하는 프로그램 기법이다. 클래스 파일의 위치나 이름만 있다면 해당 클래스의 정보를 얻고, 객체를 생성하는 것 또한 가능하기 때문에 유연한 프로그래밍을 할 수 있다.
    </p>
    <p>
        Reflection은 애플리케이션 개발보다는 <b>프레임워크, 라이브러리</b>에서 많이 사용된다. 
    </p>
    <p>
        프레임워크, 라이브러리는 사용하는 사람이 어떤 클래스와 멤버를 구성할지 모르는데, 이 사용자 클래스들을 <b>기존의 기능과 동적으로 연결시키기 위해</b> Reflection을 사용한다고 보면 된다 😊
    </p>
</div>

</details>
---

Reflection을 사용해서 **private 생성자에 접근하여 새로운 인스턴스를 생성**할 수 있고, 이로 인해 서로 다른 싱글톤 인스턴스를 생성할 수 있다.

다음 예시 코드를 살펴보자.

```java
public class SingletonReflection {
    public static void main(String[] args) {
        try {
		        // (1)
            Singleton instance1 = Singleton.getInstance();
            
            // (2)
            Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
            constructor.setAccessible(true);
            // (3)
            Singleton instance2 = constructor.newInstance();

						// (4)
            System.out.println("Instance 1 hash:" + instance1.hashCode());
            System.out.println("Instance 2 hash:" + instance2.hashCode());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- *(1) : `getInstance()` 메서드를 통해 정상적인 방법으로 싱글톤 인스턴스를 가져온다.*
- *(2) : Singleton 클래스의 생성자에 접근하기 위해 `setAccessible(true)`로 설정한다.*
- *(3) : 해당 생성자를 이용해 새로운 인스턴스를 생성한다.*
- *(4) : (1)에서 가져온 인스턴스의 hashCode와 (3)에서 생성한 인스턴스의 hashCode를 비교해보면 다음과 같이 값이 다른 것을 알 수 있다. 즉, **서로 다른 인스턴스가 만들어진 것**이다.*
    ![image](/assets/img/post/design-pattern/240702_싱글톤-패턴/screenshot_02.png){: width='250'}


### 직렬화/역직렬화 이용

---
<details>

<summary>&nbsp;&nbsp;<b>ℹ️&nbsp;직렬화<i>(Serializable)</i>와 역직렬화<i>(Deserializable)</i></b></summary>

<div style='padding-left: 12px; padding-right: 12px; padding-top: 10px'>
    <p>
        직렬화(Serializable)란 자바 시스템의 Object 또는 Data를 바이트 스트림 형태로 바꿔 외부 파일로 내보낼 수 있게 하는 기술이다.
    </p>
    <p>
        역직렬화(Deserializable)는 반대로 외부로 내보낸 직렬화 데이터를 다시 읽어들여 원래대로 자바 시스템의 Object 또는 Data로 변환하는 기술이다.
    </p>
    <p>
        직렬화해 내보낸 외부 파일은 데이터베이스에 저장되기도 하고, 네트워크를 통해 전송되기도 한다.
    </p>
    <p>
        직렬화를 적용하기 위해서는 해당 클래스에 Serializable 인터페이스를 implements 해주면 된다.
    </p>
</div>

</details>
---

먼저 예제를 살펴보기 전에, 아래와 같이 Singleton 클래스가 Serializable을 implements하도록 수정되었다고 가정한다.

```java
public class Singleton implements Serializable {
    private static final long serialVersionUID = 1L;

    private Singleton() {
        // private constructor to prevent instantiation
    }

    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

자 이제 다음 예제 코드를 살펴보자.

```java
public class SingletonSerialization {
    public static void main(String[] args) {
        try {
            // (1) 미리 생성한 싱글톤 인스턴스 가져오기
            Singleton instance1 = Singleton.getInstance();

            // (2) 인스턴스 직렬화
            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("singleton.ser"));
            out.writeObject(instance1);
            out.close();

            // (3) 인스턴스 역직렬화
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("singleton.ser"));
            Singleton instance2 = (Singleton) in.readObject();
            in.close();
            
            // (4) 각 싱글톤 인스턴스의 hashCode 값 비교
            System.out.println("Instance 1 hash:" + instance1.hashCode());
            System.out.println("Instance 2 hash:" + instance2.hashCode());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

역직렬화를 이용하면 **생성자 없이 바로 인스턴스화**하여 사용할 수 있다. 그래서 역직렬화 과정이 진행되며 새로운 인스턴스를 만들게 되고, **직렬화에 사용했던 인스턴스와는 전혀 다른 인스턴스가 된다.** 

이로 인해 싱글톤이 깨지는 문제가 발생한다. 클래스에 Serializable을 구현하면 더이상 해당 클래스는 싱글톤이 아니게 되고, 싱글톤의 장점을 더이상 얻을 수 없게 된다.

> 💡 **역직렬화 = 보이지 않는 생성자!?**<br>
자바에서 인스턴스는 생성자를 이용해 만드는 것이 기본이다. 하지만 역직렬화는 직렬화된 파일이나 데이터만 있다면, readObject 메소드를 통해 **생성자 없이 인스턴스를 만들 수 있다.**<br>
그래서 만약 어느 객체가 **생성자를 통해 인스턴스화 할 때 불변식이나 허가되지 않은 접근을 설정했더라도 이를 무시하고 생성될 수 있다**는 것이다.

### readResolve 메소드로 싱글톤 깨짐 막기 🤔

그렇다면 만약 Serializable을 반드시 implements 해야 하는데, 해당 클래스는 반드시 싱글톤으로 구현해야 한다면 어떻게 해야 할까?

바로 해당 **싱글톤 클래스에 readResolve 메소드를 직접 정의**하면 된다.

readResolve 메소드를 정의하게 되면 역직렬화 과정에서 새로 만들어진 인스턴스 대신, 기존에 만들어두었던 싱글톤 인스턴스를 반환하도록 할 수 있기 때문이다.

만약 역직렬화 과정에서 readObject 메소드가 자동으로 호출되더라도, readResolve 메소드에서 반환하는 인스턴스로 대체된다. *( + 참고로 readObject 메소드를 통해 만들어진 인스턴스는 GC의 대상이 된다 )*

```java
public class Singleton implements Serializable {
    private static final long serialVersionUID = 1L;

    private Singleton() {
        // private constructor to prevent instantiation
    }

    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }

    // 역직렬화한 객체는 무시하고, 클래스 초기화 때 만들어진 인스턴스 반환
    protected Object readResolve() {
        return getInstance();
    }
}
```

## 싱글톤 vs 정적 클래스

> *자바에는 ‘정적 클래스’라는 것이 따로 존재하지 않는다. 편의상 static 메소드로만 구성된 클래스를 ‘정적 클래스’라고 부르도록 하겠다!*
> 

### 공통점

1. 애플리케이션 내에서 **전역적으로 사용 가능**하다.
2. 인스턴스를 따로 생성하여 사용하는 것이 아니기 때문에, **유일성을 보장**받는다.

### 차이점

- 싱글톤은 **`getInstance()`** 메소드를 통해 **미리 생성한 인스턴스를 가져와 사용**한다.<br>
*( `ex` Singleton.getInstance() )*
- 정적 클래스는 인스턴스를 생성하여 사용하지 않고 **클래스를 이용해 접근**한다. <br>
*( `ex` Class.method() )*

### 그렇다면 싱글톤 대신 정적 클래스를 활용하면 안되는걸까? 🤔

싱글톤 혹은 정적 클래스를 무조건 사용해야 한다!에 대한 정답은 없다. 상황에 따라 더 유리한 방식을 선택하면 되는 것이다 😊

싱글톤은 상속이 가능하고, 메서드 파라미터로 사용이 가능하다. 

애플리케이션 내에서 **객체처럼 사용**하고 싶을 때, 혹은 인스턴스 생성 시 리소스가 많이 드는 경우 **Lazy loading이 필요할 때** 사용하기를 권장한다.

정적 클래스는 객체처럼 사용할 수는 없지만, 컴파일 시 정적 바인딩이 되기 때문에 보통 싱글톤보다 효율이 좋다. 

유틸 클래스처럼 클래스를 객체처럼 사용할 필요가 없을 때, 혹은 다형성이나 상속이 필요 없는 클래스에 사용하기를 권장한다.

## 싱글톤의 문제점

싱글톤은 **특정 클래스의 인스턴스를 하나만 생성하고 전역으로 접근할 수 있기 때문에 메모리 낭비를 방지할 수 있다**는 장점이 있지만, 물론 다음과 같은 여러 문제점도 가지고 있다.

1. **모듈 간 의존성이 높아진다.**
    
    싱글톤을 이용하게 되면 인터페이스가 아닌 클래스의 객체를 미리 생성하고, 정적 메소드를 이용하기 때문에 클래스 간 강한 의존성과 높은 결합이 생긴다. 
    
    여러 모듈에서 싱글톤 클래스 하나를 공유하고 있기 때문에, 싱글톤 클래스를 수정하면 이를 사용하는 모듈들 모두 수정이 필요하게 된다.
    
2. **S.O.L.I.D 원칙에 위배되는 경우가 많다.**
    - **단일 책임 원칙 *(SRP)* 위반** <br>
    싱글톤 자체가 인스턴스를 하나만 생성하기 때문에, 여러 책임을 가지게 되는 경우가 많다.
        
    - **개방-폐쇄 원칙 *(OCP)* 위배** <br>
    싱글톤이 혼자 너무 많은 일을 하거나 많은 데이터를 공유하게 되면, 클래스 간 결합도가 높아진다.
        
    - **의존 역전 원칙 *(DIP)* 위반** <br>
    클라이언트가 추상화 *(인터페이스)*에 의존하지 않고, 구체 클래스 *(싱글톤 클래스)*에 의존하게 된다.
        
    
    > 이러한 이유들로 싱글톤 패턴을 **‘객체 지향 프로그래밍의 안티 패턴’**이라고 부르기도 한다.
    

3. **테스트하기 어렵다.**

    단위 테스트 시에는 테스트가 서로 독립적이어야 하고, 어떤 순서로든 실행 가능해야 한다. 
    
    하지만 싱글톤은 자원을 공유하고 있기 때문에, 테스트가 문제없이 수행되려면 매번 인스턴스의 상태를 초기화해주어야만 한다. 그렇지 않으면 전역에서 상태를 공유하기 때문에 테스트가 정상적으로 수행되지 않을 수 있다.


## 참고
- <https://inpa.tistory.com/entry/GOF-💠-싱글톤Singleton-패턴-꼼꼼하게-알아보자>
- <https://youtu.be/5oUdqn7WeP0?si=qbmL0WuV4AFmLOfl>
- <https://sorjfkrh5078.tistory.com/108>
- <https://inpa.tistory.com/entry/JAVA-☕-열거형Enum-타입-문법-활용-정리#enum_과_싱글톤_관계>
- <https://inpa.tistory.com/entry/JAVA-☕-누구나-쉽게-배우는-Reflection-API-사용법>
- <https://inpa.tistory.com/entry/JAVA-☕-싱글톤-객체-깨뜨리는-방법-역직렬화-리플렉션>
- <https://madplay.github.io/post/what-is-readresolve-method-and-writereplace-method>