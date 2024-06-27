---
title: Java의 final, static 키워드
date: 2024-06-27 20:48:00 +0900
categories: [Java]
tags: [java, final, static]
---

의존성 주입과 final 키워드의 관계를 알아보다가, final 키워드에 대한 내용을 찾아보게 되었다. final 키워드와 함께 static 키워드에 대해서도 알아보면서 알게 된 내용을 정리해 작성해보았다 😉

final 키워드, static 키워드에 대해 각각 알아보고 두 키워드를 함께 쓸 때의 이점도 함께 알아보자!

## final 키워드
final 키워드는 변수, 메서드, 클래스에 사용될 수 있고, **선언된 대상의 변경을 금지**한다. final 키워드를 사용하여 코드의 안정성과 예측 가능성을 높이고, 의도치 않은 변경으로부터 보호할 수 있다.

**1. final 변수**
- 해당 변수는 값이 초기화된 후 값을 변경할 수 없다. 즉, **상수가 되어 값의 변경이 불가능**해진다.
- 프로그램 실행 도중 예상치 못한 값의 변경을 막고, 코드 안정성을 높인다.

**2. final 메서드**
- 해당 메서드는 하위 클래스에서 재정의(override)할 수 없다. 즉, 메서드의 구현이 최종이 되어, 하위 클래스에서 변경할 수 없게 된다.
- 부모 클래스의 특정 변경 동작을 못하게 해 코드의 예측 가능성을 높이고, 설계 의도를 명확히 전달하는 데 도움을 준다.

**3. final 클래스**
- 해당 클래스는 다른 클래스에서 상속받을 수 없다.
- 클래스의 확장을 방지하고, 불변성을 유지하고, 보안과 성능을 향상시킨다.<br>
    *( `ex` 자바의 String 클래스는 final로 선언되어 이를 상속하거나 변경할 수 없다. )*
    
> **즉, 변경되지 않아야 할 요소에는 final 키워드를 사용하는 것이 좋다!**


## static 키워드
static 키워드는 클래스 멤버인 변수, 메서드에 적용될 수 있다. 클래스가 로드될 때 단 한 번만 메모리에 할당되고, 이후 생성되는 모든 인스턴스에서 공유된다. 그래서 상수값이나, 공유해야 할 자원을 관리할 때 사용하면 유용하다.

1. **static 변수**
    - 클래스 변수 라고도 부르며, **클래스의 인스턴스 간 데이터를 공유**할 때 사용한다.
    - 클래스가 처음 로드될 때 메모리에 할당되고, 클래스의 인스턴스가 생성되기 전에 이미 메모리에 존재하게 된다. 그래서 **인스턴스가 생성되기 전에도 접근이 가능**하며, 클래스명을 통해 접근할 수 있다. *(ex. MyClass.count)*
    - 하나의 인스턴스에서 static 변수 값을 변경하면, 해당 변수를 사용하는 여러 인스턴스에 영향을 미치기 때문에 주의해서 사용해야 한다. *(모든 인스턴스에서 해당 변수를 공유하기 때문에!)*

    ~~~java
    class MyClass {
        static int count = 0;
        // ...
    }
    
    // MyClass의 인스턴스를 생성하지 않고도 정적 변수 count에 접근 가능
    MyClass.count++;
    ~~~

2. **static 메서드**
    - 클래스가 로드될 때, 메모리의 메서드 영역에 적재된다. 그래서 객체의 인스턴스와 상관 없이 클래스 수준에서 메서드를 호출할 수 있다.
        <details>

        <summary>&nbsp;&nbsp;<i>메모리의 메서드 영역 (Method Area)</i></summary>
        <ul>
        <li>JVM이 시작될 때 생성되는 공간으로, 바이트 코드(.class)를 처음 메모리 공간에 올릴 때 초기화되는 대상을 저장하기 위한 메모리 공간이다.</li>
        <li>JVM이 동작하고 <b>클래스가 로드될 때 적재</b>되어 프로그램이 종료될 때까지 저장된다.</li>
        <li><b>모든 스레드가 공유하는 영역</b>이며, 정적 필드와 클래스 구조에 대한 정보를 가진다.</li>
        </ul>

        </details>
    - 객체의 상태를 변경하지 않고, 인자로 주어진 값만을 처리하는 경우에 주로 사용한다.
    - 클래스를 통해 직접 호출할 수 있어, **인스턴스 생성 없이 사용할 수 있다**.
    *(그래서 인스턴스 변수에 접근할 수 없고 static 변수나 다른 static 메소드에만 접근 가능하다.)*
    - 유틸리티 함수나 상태가 없는 메서드를 구현할 때 주로 사용한다.
    
    ~~~java
    class PrintUtil {
        static void printMessage(String message) {
            System.out.println(message);
        }
    }
    
    // 인스턴스를 생성하지 않고도 정적 메서드 호출 가능
    PrintUtil.printMessage("Hello, World!");
    ~~~
    
    <div class="promt-info">
        <h5>인스턴스 객체를 통해 static 메서드에 접근해도 괜찮을까?!</h5>
        <p>
            갑자기 든 의문점, static 메서드를 클래스명.메서드명()으로 접근하지 않고, <b>인스턴스를 생성해 해당 객체의 메서드를 호출하도록 하면 어떻게 될까?!</b><br>
            Java에서 static 메서드는 인스턴스 객체를 통해 호출하는 것도 가능하지만, 클래스명으로 접근하는 것을 권장한다고 한다.<br>
            다음 예시를 살펴보며 알아보자. <i>static 메서드를 클래스명과 인스턴스 객체를 통해 호출하는 예제이다.</i><br>
        <!-- </p> -->
        
        <pre><code>
        class MyClass {
            static void staticMethod() {
                System.out.println("Static method called.");
            }
        }

        public class Main {
            public static void main(String[] args) {
                // 클래스명을 통한 호출 (권장)
                MyClass.staticMethod();

                // 인스턴스 객체를 통한 호출 (비권장)
                MyClass obj = new MyClass();
                obj.staticMethod(); // 컴파일러 경고 없이 동작
            }
        }
        </code></pre>

        <!-- <p> -->
            static 메서드를 인스턴스 객체를 통해 호출해도 컴파일러에서는 아무런 에러를 내지 않는다. 실행 시에도 정상적으로 동작한다.<br>
            인스턴스 객체를 통해 호출하더라도, 실제로는 내부적으로 <b>클래스를 통해 호출하는 방식으로 동작</b>한다. <i>(obj.staticMethod()가 MyClass.staticMethod()와 동일하게 동작하는 것!)</i>
        </p>
    </div>
    
3. **static 블록**
    - **클래스가 처음 로드될 때 실행되는 코드 블록**이다.
    - 클래스 변수의 초기화나 복잡한 초기화 로직을 수행할 때 사용한다.
    
    ```java
    class MyClass {
        static {
            // 정적 초기화 블록
            System.out.println("MyClass is being loaded.");
            // 클래스 변수 초기화 등의 작업 수행
        }
    }
    ```


## 인스턴스 객체를 통해 static 메서드에 접근해도 괜찮을까?!

갑자기 든 생각! 만약 static 메서드를 클래스명.메서드명()으로 접근하지 않고, **인스턴스를 생성해 해당 객체의 메서드를 호출하도록 하면 어떻게 될까?!**

Java에서 static 메서드는 인스턴스 객체를 통해 호출하는 것도 가능하지만, 클래스명으로 접근하는 것을 권장한다고 한다.

~~~java
class MyClass {
    static void staticMethod() {
        System.out.println("Static method called.");
    }
}

public class Main {
    public static void main(String[] args) {
        // 클래스명을 통한 호출 (권장)
        MyClass.staticMethod();

        // 인스턴스 객체를 통한 호출 (권장하지 않음)
        MyClass obj = new MyClass();
        obj.staticMethod(); // 컴파일러 경고 없이 동작
    }
}
~~~

위 코드를 살펴보면, static 메서드를 인스턴스 객체를 통해 호출해도 컴파일러에서 아무런 경고를 주지 않는다. 실행 시에도 정상적으로 동작한다.

인스턴스 객체를 통해 호출하더라도, 실제로는 내부적으로 **클래스를 통해 호출하는 방식으로 동작**한다. *(obj.staticMethod()가 MyClass.staticMethod()와 동일하게 동작하는 것!)*


## static키워드와 final 키워드 함께 사용하기
static과 final 키워드는 함께 사용하면 더 큰 효과를 발휘하는데, 적절한 상황에서 활용한다면 코드의 안정성과 재사용성을 높이는데 도움이 된다.

**static final 변수**를 예로 들어보자.

클래스의 상수이고, 모든 인스턴스에서 공유되며 변경할 수 없는 값이다. 이는 프로그램 전체에서 공유되면서 일관된 값을 가져야할 때 유용하다. *(ex. 수학의 파이(π) 값이나, 환경 설정에서의 최대 파일 크기 등)*


## 참고
- <https://f-lab.kr/insight/understanding-java-static-final?gad_source=1>