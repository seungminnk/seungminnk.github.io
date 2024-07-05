---
title: 동작 파라미터화와 람다식
date: 2024-07-05 16:57:00 +0900
categories: [Java]
tags: [java, behavior-parameterization, lambda]
---

## 들어가면서
자바 역사 상 가장 큰 변화는 자바 8에서 일어났다고 해도 과언이 아니다. 자바 8에서 새롭게 추가된 개념과 기법들이 많은데, 자바 8 설계의 밑바탕을 이루는 세 가지 프로그래밍 개념은 다음과 같다.

1. 스트림 처리
2. 동작 파라미터화로 메서드에 코드 전달하기
3. 병렬성과 공유 가변 데이터

이 중 ‘**동작 파라미터화**’ 개념은 **람다식**이 탄생한 이유이기도 하며, 자바 8에 광범위하게 적용된 소프트웨어 개발 패턴이다. 

이 동작 파라미터화를 추가하면서, 쓸데 없는 코드가 늘어날 수밖에 없는데, 자바 8에서는 이 문제를 람다식으로 해결한다. 그래서 동작 파라미터화에 대해 먼저 살펴본 후, 람다식을 통해 어떻게 코드를 더 간결하게 만드는지에 대해 알아보도록 하자 😊

## 동작 파라미터화
어떤 상황에서 일을 하든 소비자 요구사항은 항상 바뀌고, 이는 소프트웨어 엔지니어링에서 피할 수 없는 숙명이다.

비용은 최소화되고, 새로 추가할 기능은 쉽게 구현되고, 유지보수가 쉽다면 시시각각 변하는 사용자 요구사항에 대응하기에 좋을 것이다.

동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. 어떻게 동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 유연하게 대응할 수 있다는걸까?

동작 파라미터화란 **아직은 어떻게 실행할 것인지 결정하지 않은 코드 블럭**을 의미한다. 이 코드 블록의 실행은 **나중으로 미뤄진다.**

예를 들면, 컬렉션을 처리할 때 **리스트의 모든 요소에 ‘어떤 동작’을 수행**한다던가, **리스트 관련 작업을 끝낸 후 ‘어떤 다른 동작’을 수행**하는 등의 기능을 구현할 수 있다.

### 예시로 살펴보기
농장 재고 목록 애플리케이션에 **리스트에서 초록색 사과만 필터링하는 기능**을 추가한다고 생각해보자. 
*간단한 요구사항이니 다음과 같이 간단히 코드를 구현할 수 있을 것이다!*

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (GREEN.eqauls(apple.getColor())) {  // 초록색 사과일 경우 결과 리스트에 추가
            result.add(apple);
        }		
    }
    
    return result;
}
```

리스트에서 사과 색깔을 확인해 초록색 사과인 경우에만 리스트에 추가해 해당 리스트를 반환하는 메소드를 구현했다.

그런데 이때 농부가 **‘초록색 사과 말고 빨간 사과도 필터링 하고싶어요’** 라고 요구한다면, 코드를 어떻게 고쳐야할까?

단순히 위 코드를 복사해서 filterRedApples 메소드를 만들고, if문 조건을 빨간색인지 확인하는 코드로 만들면 된다고 생각할 수 있다.

하지만 ‘**노란 사과도 필터링 하고싶어요**’, ‘빨간 사과 말고 **진한 빨간색 사과로 필터링 하고싶어요**’ 와 같이 새로운 요구사항이 있다면, 코드를 복사해 새로운 메소드를 계속 추가하게 되고, 비슷한 코드가 중복되는 문제가 생길 것이다.

그렇다면 어떻게 해야 할까? 색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응할 수 있지 않을까?

다음 코드를 살펴보자.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getColor().equals(color)) {
            result.add(apple);
        }		
    }
    
    return result;
}

public static void main(String[] args) {
    List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
    List<Apple> redApples = filterApplesByColor(inventory, RED);
}
```

사과 색깔과 파라미터로 받은 색이 일치하는 경우에만 리스트에 추가하고, 해당 리스트를 반환하도록 코드를 수정했다. 그리고 해당 메소드를 활용해 초록색 사과, 빨간 사과를 각각 필터링할 수 있다.

그런데 이제는 농부가 **‘색 말고, 150g 이상의 사과만 분류하고 싶어요.’** 라고 요구한다.

자, 그럼 어떻게 해야할까? 다음 코드를 살펴보자.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getWeight() > weight) {
            result.add(apple);
        }		
    }
    
    return result;
}
```

사과의 무게가 파라미터로 받은 무게 이상인 경우, 결과 리스트에 넣어 반환하도록 구현했다.

이렇게 구현한 코드도 좋은 해결책이라고 할 수 있지만, 위에서 구현했던 사과를 색으로 필터링 하는 메소드인 filterApplesByColor와 if문 조건을 제외하고는 코드가 모두 중복된다.

중복된 코드를 없애기 위해 다음과 같이 파라미터에 색과 무게 중 어떤 것으로 필터링할 지 결정하는 플래그를 추가하도록 수정했다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ((flag && apple.getColor().eqauls(color)) ||
            (!flag && apple.getWeight().eqauls(weight))) {
            result.add(apple);
        }		
    }
    
    return result;
}

public static void main(String[] args) {
    List<Apple> greenApples = filterApplesByColor(inventory, GREEN, 0, true);
    List<Apple> redApples = filterApplesByColor(inventory, null, 150, false);
}
```

이렇게 구현한 코드는 가독성도 떨어지고, 메소드를 호출할 때 파라미터로 넣는 값이 무엇을 의미하는 건지도 명확하지 않다.

무엇보다도 또 다른 요구사항에 대응하기에 너무나도 적절하지 않은 코드다. 초록색 사과 중 150g 이상의 사과를 필터링 하고 싶다면? 혹은 색과 무게 이외의 다른 조건이 추가된다면?

> **filterApple 메소드에 어떤 기준으로 사과를 필터링할 건지에 대한 내용 자체를 전달할 수 있다면 좋지 않을까?**
> 

바로 이런 경우에 **동작 파라미터화**를 적용하면 된다!

### 동작 파라미터화 적용하기
자, 그러면 이제 앞서 살펴보았던 예시에 동작 파라미터화를 적용해보자!

먼저 어떤 기준으로 사과를 필터링할 것인지를 정의해야 한다. 사과 선택 조건을 결정하는 인터페이스를 먼저 정의해보자. *(참고로, 참 / 거짓을 반환하는 함수를 Predicate 라고 한다.)*

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}

// 무게가 150g 이상인 사과 선택
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

// 초록색 사과 선택
public class AppleGreenColorPredicate implements ApplePredicate {
    public test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```

사과 선택 조건을 결정하는 인터페이스 ApplePredicate를 추가하고, 이를 implements하는 AppleHeavyWeightPredicate, AppleGreenColorPredicate 클래스도 추가했다.

AppleHeavyWeightPredicate 클래스의 test() 에서는 **150g 이상의 사과인 경우 참을 반환**하도록, AppleGreenColorPredicate 클래스의 test() 에서는 **초록색 사과인 경우 참을 반환**하도록 구현했다.

사과 선택 조건을 결정했으니, 이제는 사과를 필터링할 때 사과 조건을 파라미터로 받아 이에 맞게 검사할 수 있도록 수정해야 한다.

filterApples 메소드에 ApplePredicate를 파라미터로 받아 사과 조건을 검사할 수 있도록 해보자.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) {  // 사과 필터링 조건에 만족하면 결과 리스트에 추가
            result.add(apple);
        }		
    }
    
    return result;
}

public static void main(String[] args) {
    // 초록색 사과로 필터링
    List<Apple> greenApples = filterApplesByColor(inventory, new AppleGreenColorPredicate());
}
```

filterApples 메소드에서는 **단순히 ApplePredicate라는 사과 필터링 조건을 넘겨받아 해당 조건을 만족**하면 리스트에 추가하고, 결과 리스트를 반환해준다.

그리고 filterApples 메소드를 호출할 때 파라미터에 `new AppleGreenColorPredicate()`를 넘겨주어 초록색 사과로 필터링하도록 했다.

그럼 이제 동작 파라미터화를 적용하기 전 코드와 비교해보자.

**이전 코드에 비해 훨씬 가독성이 좋아졌고, 사용하기 쉬워졌다. 무엇보다도 새로운 요구사항에 유연하게 대응할 수 있게 되었다.**

예를 들면, ‘200g 이상의 빨간 사과로 필터링하고 싶어요’라는 요구사항에는 다음과 같이 조건에 맞는 Predicate 클래스를 구현하고, filterApples 메소드에 해당 클래스를 파라미터로 넘겨주기만 하면 된다.

```java
// 무게가 200g 이상이고 빨간색 사과 선택
public class AppleHeavyWeightAndRedColorPredicate implements ApplePredicate {
    public test(Apple apple) {
        return apple.getWeight() > 200 && RED.equals(apple.getColor());
    }
}

public static void main(String[] args) {
    List<Apple> redHeavyApples = filterApplesByColor(inventory, new AppleHeavyWeightAndRedColorPredicate());
}
```

이렇게 우리는 filterApples 메서드의 동작을 파라미터화했다! 💪 **컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다**는 것이 동작 파라미터화의 강점이다.

### 익명 클래스로 코드 간소화하기
동작 파라미터화를 적용하면서, 코드의 가독성이 올라갔고 유연성을 얻을 수 있었다. 

하지만 ApplePredicate 인터페이스나, 이를 구현한 여러 사과 조건 클래스들을 모두 만들어야 하는 것이 꽤나 귀찮고 불필요한 과정으로 느껴진다 😂

이는 ‘익명 클래스’를 통해 해결할 수 있는데, 이를 이용하면 코드의 양을 훨씬 줄일 수 있다.

익명 클래스는 말 그대로 ‘**이름이 없는 클래스**’다. 클래스의 선언과 인스턴스화를 동시에 할 수 있다. 즉, 즉석에서 필요한 구현을 만들어서 사용할 수 있다.

익명 클래스를 적용한 다음 코드를 살펴보자.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
});
```

익명 클래스를 이용해 필요한 사과 필터링 조건 구현하여 파라미터 자리에 넘겨주어 사용하도록 수정했다. 이로 인해, 불필요한 ApplePredicate, AppleRedColorPredicate 코드 구현을 줄일 수 있었다!

### 람다식 적용하기
익명 클래스를 사용해 불필요한 코드를 줄일 수 있었다. 하지만 이 방법도 단점이 있다.

익명 클래스를 사용하면서, 코드가 장황해져 **가독성이 떨어진다**. 그리고 익명 클래스를 구현해 전달하는 과정에서, 결국 **객체를 만들고 명시적으로 새로운 메서드를 구현**해야 한다.

이 문제는 람다식을 이용해 해결할 수 있다. 

일단은 람다식을 적용하면 코드가 얼마나 **간결하고 가독성이 높아지는 지**만 살펴보고 넘어가도록 하자 😊

```java
List<Apple> redApples = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor());
```


## 람다식
앞서 살펴본 것처럼 동작 파라미터화를 이용해 정의한 코드 블록을 다른 메서드로 전달하고, 이를 통해 유연하고 재사용가능한 코드를 만들 수 있었다. 그리고 이를 통해 변화하는 요구사항에 효과적으로 대응하는 코드를 구현할 수 있었다.

람다를 이용하면 동작 파라미터 형식의 코드를 보다 더 쉽게 구현할 수 있고, 코드가 간결하고 유연해 질 수 있다.

### 람다식이란?
**메서드로 전달할 수 있는 익명 함수를 단순화한 것**이라고 할 수 있다.

람다식은 파라미터, 화살표, 바디로 이루어진다.

```java
//                  < 화살표 >
   (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
// <--- 람다 파라미터 --->     <------------   람다 바디  -------------->
```

### 람다식 작성하기

int 타입 파라미터 a의 값을 콘솔에 출력하는 람다식을 작성하는 과정을 단계별로 살펴보자.

맨 먼저 아래와 같이 람다식을 작성할 수 있다.

```java
(int a) -> { System.out.println(a); }
```

파라미터 타입은 런타임 시 대입되는 값에 따라 자동으로 인식될 수 있기 때문에, 파라미터 타입은 다음과 같이 생략할 수 있다.

```java
(a) -> { System.out.println(a); }
```

하나의 파라미터만 있을 경우 괄호를 생략할 수 있고, 하나의 실행문만 있다면 중괄호도 생략 가능하다.

```java
a -> System.out.println(a)
```

이렇게 파라미터 a의 값을 콘솔에 출력하는 람다식은 위와 같이 작성할 수 있다!

만약 파라미터가 하나도 없는 경우라면 람다식에서 파라미터 자리가 사라지므로, 빈괄호를 반드시 포함해주어야 한다.

```java
() -> { 실행문; ... }
```

중괄호를 실행하고 결과 값을 리턴해야 한다면, 다음과 같이 return문으로 결과 값을 지정할 수 있다.

```java
(x, y) -> { return x + y; }
```

만약 위와 같이 중괄호 안에 return문만 있을 경우, 아래처럼 return을 사용하지 않고 작성하는 것이 정석이다.

```java
(x, y) -> x + y
```

### 그래서 람다식을 어디서 사용할 수 있다고?
다 알겠고, 람다식을 그래서 어디서 사용할 수 있다는걸까? 어디서든 람다식을 사용할 수 있는걸까? 🤔

일단, 람다식은 인터페이스 변수에 대입된다. 즉, **인터페이스의 익명 구현 객체를 생성**한다는 뜻이다.

이게 무슨말이냐, 자바에서 인터페이스는 직접 객체화할 수 없고, 이 인터페이스를 구현하는 구현 클래스가 필요하다. 람다식은 이 익명 구현 클래스를 생성하고 객체화한다는 뜻이다.

그리고 람다식은 대입될 인터페이스의 종류에 따라 작성 방법이 달라지는데, 이 람다식이 대입될 인터페이스를 **람다식의 타겟 타입**이라고 한다.

람다식은 인터페이스의 익명 구현 객체를 생성한다고 했다. **람다식은 하나의 메소드를 정의**하고 있기 때문에, 두 개 이상의 추상 메소드가 선언된 인터페이스는 람다식에 대입될 수 없다.

즉, 하나의 추상 메소드가 선언된 인터페이스만이 람다식의 타겟 타입이 될 수 있는데, 이 인터페이스를 함수형 인터페이스라고 한다.

자 그러면 다시 돌아와서 정리해보자면, **하나의 추상 메소드가 선언된 인터페이스, 즉 함수형 인터페이스 자리에 가 람다식을 사용할 수 있다**는 것이다.

---
💡 **@FunctionalInterface**

@FunctionalInterface 어노테이션을 붙이면 함수형 인터페이스를 작성할 때 두 개 이상의 추상 메소드가 선언되지 않도록 컴파일러가 체킹해준다.

*두 개 이상의 추상 메소드가 선언되면 컴파일 오류를 발생시킨다.*

---

### 함수형 인터페이스 별 람다식 작성 방법
1. **파라미터와 리턴값이 없는 람다식**
    
    다음과 같이 파라미터와 리턴값이 없는 추상 메소드를 가진 함수형 인터페이스가 있다.
    
    ```java
    @FunctionalInterface
    public interface MyInterface {
        public void method();
    }
    ```
    
    이 인터페이스를 타겟 타입으로 갖는 람다식은 다음과 같은 형식으로 작성할 수 있다.
    
    ```java
    MyInterface mi = () -> { ... }
    ```
    
    람다식이 대입된 인터페이스의 참조 변수는 다음과 같이 method()를 호출할 수 있고, 이는 람다식의 중괄호를 실행시킨다.
    
    ```java
    mi.method();
    ```
    
2. **파라미터가 있는 람다식**
    
    다음과 같이 파라미터가 있고 리턴값이 없는 추상 메소드를 가진 함수형 인터페이스가 있다.
    
    ```java
    @FunctionalInterface
    public interface MyInterface {
        public void method(int x);
    }
    ```
    
    이 인터페이스를 타겟 타입으로 갖는 람다식은 다음과 같은 형식으로 작성할 수 있다.
    
    ```java
    MyInterface mi = (x) -> { ... }
    
    또는
    
    MyInterface mi = x -> { ... }
    ```
    
    람다식이 대입된 인터페이스의 참조 변수는 다음과 같이 method()를 호출한다. 파라미터로 5를 넘기면, 람다식의 x에 5가 대입되고, 이는 중괄호 내 코드에서 사용된다.
    
    ```java
    mi.method(5);
    ```
    
3. **파라미터와 리턴값이 있는 람다식**
    
    다음과 같이 파라미터와 리턴값이 있는 추상 메소드를 가진 함수형 인터페이스가 있다.
    
    ```java
    @FunctionalInterface
    public interface MyInterface {
        public int method(int x, int y);
    }
    ```
    
    이 인터페이스를 타겟 타입으로 갖는 람다식은 다음과 같은 형식으로 작성할 수 있다.
    
    ```java
    MyInterface mi = (x, y) -> { ...; return x + y; }
    
    // 만약 중괄호에 return 문만 있는 경우 다음과 같이 작성할 수 있다.
    MyInterface mi = (x, y) -> x + y
    ```
    
    람다식이 대입된 인터페이스의 참조 변수는 다음과 같이 method()를 호출한다. 파라미터로 2와 5를 넘기면, 람다식의 x에 2, y에 5가 대입되고, 이는 중괄호 내 코드에서 사용된다.
    
    ```java
    mi.method(2, 5);
    ```
    

### 람다 활용 예시 : 실행 어라운드 패턴
동작 파라미터화와 람다를 통해 유연하고 간결한 코드를 구현한 예시를 하나 살펴보고 넘어가자.

자원을 열고, 처리하고, 자원을 닫는 순서로 이루어지는 형태로 이루어진 코드를 실행 어라운드 패턴이라고 한다. 쉽게 말하면 **실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태**를 가진 코드이다.

다음 예제에 람다를 적용해보자. 파일에서 한 행을 읽어오는 메소드를 구현하는 코드이다.

```java
public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();
    }
}
```

위 코드는 파일에서 한 줄만 읽어올 수 있다. 만약 한 번에 두 줄을 읽거나, 가장 자주 사용되는 단어를 반환하기 위해서는 어떻게 하면 좋을까?

설정, 정리 과정은 그대로 두고, 한 줄만 읽어오는 부분만 다른 동작을 수행하도록 하면 좋을 것 같다.

> 어, 뭔가 익숙한데? 싶으면 맞다! 😉 해당 부분을 파라미터로 받아 수행하도록 동적 파라미터화 하면 된다!

자, 그러면 processFile 메소드를 수정해보자! 

processFile 메소드가 BufferedReader를 이용해 다른 동작을 수행할 수 있도록 processFile에 동작을 전달하면 좋을 것 같다. 동작을 전달할 때는 람다를 이용해보자.

BufferedReader를 이용해 한 줄을 읽어오는 동작을 람다식으로 짜보면 아래와 같다.

```java
String result = (BufferedReader br) -> br.readLine();
```

이제 동작을 메소드에서 받을 수 있도록 해보자. 메소드에 동작을 넘기려면, 메소드의 **파라미터에 인터페이스 형식**으로 넘겨주면 된다. 

그리고 앞서 작성했던 람다식을 다시 생각해보면, BufferedReader를 이용해 특정 동작을 수행하는 형태이기 때문에 BufferedReader를 파라미터로 받아 String을 리턴하는 형식이다.

이와 동일한 형식의 추상 메소드를 갖는 함수형 인터페이스를 정의하면 된다.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```

이렇게 정의한 인터페이스를 processFile 메소드의 파라미터로 전달해 사용할 수 있다. 다음은 위에서 살펴봤던 코드를 동작 파라미터를 적용해 수정한 코드이다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}
```

람다는 함수형 인터페이스 자리에 사용할 수 있다고 했다. 다시 말하면, processFile 메소드를 호출할 때 파라미터 자리에 람다를 이용해 동작을 전달할 수 있다는 말이다.
***( BufferedReaderProcessor 인터페이스의 추상 메소드인 process의 구현을 람다식으로 구현해 전달하고, 해당 코드에서 process가 실행되면 전달된 람다식이 실행되는 것! )***

람다를 이용해 다음과 같이 processFile 메소드에 여러 동작을 전달할 수 있다.

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

### 더알아보기 : 지역 변수 사용
람다식에서 바깥 클래스의 필드나 메소드는 제한 없이 사용할 수 있지만, 메소드의 파라미터(매개변수) 혹은 지역 변수를 사용한다면 이 두 변수는 final 특성을 가져야 한다.

왜 final 특성을 가져야 하는걸까? 🤔

우선 람다식은 메소드 내부에서 주로 작성되기 때문에, 로컬 익명 구현 객체를 생성한다고 보면 된다.

메소드 내에서 생성된 익명 객체는 메소드 실행이 끝나도 힙 메모리에 존재하기 때문에, 계속 사용이 가능하다. 하지만 파라미터나 지역 변수는 스택 메모리에 저장되기 때문에 메소드 실행이 끝나면 메모리에서 사라지고, 이로 인해 익명 객체에서 사용할 수 없게 되는 것이다.

그래서 자바에서는 메소드의 파라미터나 지역 변수를 익명 객체 내부에 복사해두고 사용하도록 한다. 복사해둔 값을 사용하는데 만약 **값이 달라지면 문제가 일어날 수 있기 때문에, 해당 변수들은 final의 특성을 가져야만 하는 것**이다.

### 자바 API에서 제공하는 함수형 인터페이스들
앞선 예제에서는 필요한 함수형 인터페이스를 직접 정의하여 사용했는데, 자바 8부터는 자주 사용되는 함수형 인터페이스를 java.util.function 패키지로 제공한다.

이 패키지에서 제공하는 함수형 인터페이스들은 **메소드 또는 생성자의 매개 타입으로 사용되어 람다식을 대입할 수 있다.** 그래서 우리가 개발하려는 메소드에도 이 함수형 인터페이스들을 사용할 수 있다.

java.util.function 패키지의 함수형 인터페이스는 인터페이스에 선언된 추상 메소드의 파라미터와 리턴값 유무에 따라 다음과 같이 구분된다.

1. **Consumer**
    - 파라미터 O, 리턴값 X  
    *⇒ T를 받아 void를 리턴하는 추상 메소드 accept()*
    
    > 파라미터를 소비하기만 하고 리턴하는 값은 없으니 Consumer*(소비자)* 😎
    > 
2. **Supplier**
    - 파라미터 X, 리턴값 O
    
    > 파라미터는 없고, 호출한 곳으로 **값을 리턴**(공급)하기만 하기 때문에 Supplier(공급자*)* 😎
    > 
3. **Function**
    - 파라미터 O, 리턴값 O  
    *⇒ T를 받아 R을 리턴하는 추상 메소드 apply()*
    - 주로 파라미터를 리턴값으로 매핑 *(타입 변환)*
4. **Operator**
    - 파라미터 O, 리턴값 O
    - 파라미터 값을 연산해 결과를 리턴
5. **Predicate**
    - 파라미터 O, 리턴값 O
    *⇒ T를 받아 boolean을 반환하는 추상 메서드 test()*
    - 파라미터를 확인해서 true / false를 리턴

### 메소드 참조

메소드를 참조해서 파라미터의 정보 및 리턴 타입을 알아내어, 람다식에서 불필요한 파라미터를 제거하는 것이 목적이다.

람다식은 종종 기존 메소드를 단순히 호출만 하는 경우가 많다. 예를 들면, 다음과 같이 두 개의 값을 받아 큰 수를 리턴하는 람다식을 들 수 있다.

```java
(num1, num2) -> Math.max(num1, num2)
```

위와 같은 람다식은 두 개의 값을 단순히 Math.max()라는 메소드의 파라미터로 전달하는 역할만 하고 있다. 이럴 때 메소드 참조를 이용하면 코드를 깔끔하게 수정할 수 있다.

```java
Math::max
```

메소드 참조는 정적 또는 인스턴스 메소드를 참조하거나, 생성자 참조가 가능하다. 하나씩 살펴보자.

1. **정적 메소드 & 인스턴스 메소드 참조**
    - 정적 메소드를 참조할 경우, 클래스 이름 뒤 :: 기호를 붙이고 해당 메소드의 이름을 작성하면 된다.
    *( 클래스 :: 메소드 )*
    - 인스턴스 메소드를 참조할 경우, 먼저 객체 생성 후에 해당 참조변수 뒤 :: 기호를 붙이고 메소드 이름을 작성하면 된다.
    *( 참조변수 :: 메소드 )*
    
    ```java
    public class Calculator {
        public static int staticMethod(int x, int y) {
            return x + y;
        }
        
        public int instanceMethod(int x, int y) {
            return x + y;
        }
    }
    
    public class MethodReferencesExample {
        public static void main(String[] args) {
            Operator operator;
            
            // 정적 메소드 참조
            operator = Calculator::staticMethod;
            operator.apply(1, 2);
            
            // 인스턴스 메소드 참조
            Calculator cal = new Calculator();
            operator = cal::instanceMethod;
            operator.apply(3, 4);
        }
    }
    ```

2. **파라미터의 메소드 참조**
    
    b를 파라미터로 사용해 a의 메소드를 호출하는 람다식이 있을 때, 다음과 같이 표현할 수 있다.
    
    ```java
    // 람다식
    (a, b) -> { a.instanceMethod(b); }
    
    // 메소드 참조
    클래스::instanceMethod
    ```
    
    예시 코드로 살펴보자. 두 문자열이 대소문자 상관없이 동일한 알파벳으로 구성돼있는지 비교하는 코드이다.
    
    ```java
    public class ArgumentMethodReferencesExample {
        public static void main(String[] args) {
            Function<String, String> function;
            
            // function = (a, b) -> a.compareToIgnoreCase(b); 와 같음
            function = String::compareToIgnoreCase;
            function.apply("Java8", "JAVA8");
        }
    }
    ```
    
3. **생성자 참조**
    
    단순히 객체 생성 후 리턴만 하는 람다식이 있을 때, 생성자 참조로 표현하면 다음과 같다.
    
    ```java
    // 람다식
    (a, b) -> { return new 클래스(a, b); }
    
    // 생성자 참조
    클래스::new
    ```
    
    생성자 참조를 이용해 Member 객체를 생성하는 예시이다.
    
    ```java
    public class ConstructorReferencesExample {
        public static void main(String[] args) {
            Function<String, Member> function1 = Member::new;
            Member mem1 = function1.apply("홍길동");
            
            BiFunction<String, Integer, Member> function2 = Member::new;
            Member mem2 = function2.apply("홍길동", 21);
        }
    }
    
    public class Member {
        private String name;
        private int age;
        
        public Member() {}
        
        public Member(String name) {
            this.name = name;
        }
        
        public Member(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }
    ```
    

## 참고
- 모던 자바 인 액션
- 이것이 자바다 2권