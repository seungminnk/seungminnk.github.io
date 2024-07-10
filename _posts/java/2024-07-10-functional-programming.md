---
title: 함수형 프로그래밍 (Functional Programming)
date: 2024-07-10 14:50:00 +0900
categories: [Java]
tags: [java, functional-programming, imperative-programming, declarative-programming, programming-paradigm]
---

함수형 프로그래밍에 대해 살펴보기 전에, 프로그래밍 패러다임을 이루는 두 가지 개념인 명령형 프로그래밍과 선언형 프로그래밍에 대해 알아보자 🤗

프로그래밍 패러다임은 크게 명령형 프로그래밍과 선언형 프로그래밍으로 나뉜다. 이 두 가지 방식은 프로그램의 구조와 실행 방식을 다르게 정의한다.

## 명령형 프로그래밍

먼저, 명령형 프로그래밍은 **어떻게(HOW)** 프로그램을 실행할지 정의한다. 

명령형 프로그래밍은 **상태 변화**와 루프를 사용해 논리를 구현하고, 제어 흐름(조건문, 반복문 등)을 명확히 정의하는 것이 특징이다.

> 즉, **‘이 일을 먼저 하고, 그 다음에 저 값을 갱신하고, 그 다음에 …’** 와 같이 작업을 **어떻게 수행할 것인지**에 집중하는 방식이다.
> 

다음 예시를 살펴보자. 짝수를 필터링하는 예시이다.

```java
public class ImperativeExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        List<Integer> evenNumbers = new ArrayList<>();

        for (int number : numbers) {
            if (number % 2 == 0) {
                evenNumbers.add(number);
            }
        }

        System.out.println(evenNumbers);
    }
}
```

숫자 리스트가 주어졌고, 짝수를 저장할 리스트를 선언했다. for루프를 돌면서 짝수인 경우 해당 수를 짝수 리스트에 추가한다. 마지막으로 짝수 리스트를 콘솔에 출력한다.

‘어떻게’ 짝수 리스트를 필터링할지를 명확히 정의하고 있다. 이렇게 명령형 프로그래밍은 ‘***어떻게(HOW)***’에 집중하는 프로그래밍 방식이다.

### 장점

1. **제어 흐름이 명확히 드러나있**기 때문에, 프로그램이 어떻게 동작하는지 이해하기 쉽다.
2. 각 단계가 명시적으로 정의되어 있어 **디버깅이 쉽다**.
3. **세부적인 동작을 직접 제어**할 수 있어 복잡한 로직을 구현할 때 유용하다.
4. 많은 프로그래밍 언어가 명령형 프로그래밍 방식을 기반으로 하고 있어 대부분의 개발자에게 익숙하다.

### 단점

1. 상태 변화와 제어 흐름이 복잡해질 수 있기 때문에, 유지보수가 어렵다.
2. **상태 변화**를 일으키는 코드가 많아 **예측하지 못한 부작용이 발생**할 수 있다.
3. 코드의 특정 부분을 재사용하기 어렵고, 코드 중복이 발생할 가능성이 높다.

<aside>
💡 **부작용**이란?

**원하는 결과 외의 모든 변화**를 말한다. 메서드 외부 변수가 변경되거나, 예외가 발생하는 등을 예로 들 수 있다.

</aside>

## 선언형 프로그래밍

명령형 프로그래밍과는 달리, 선언형 프로그래밍은 무엇(WHAT)을 해야 하는지를 명확히 정의한다. 프로그램의 **상태 변화 없이** 연산을 기술하고, 논리를 명확히 표현한다.

선언형 프로그래밍은 **상태 변화가 거의 없고, 제어 흐름보다는 데이터의 흐름을 중시**하는 특징이 있다.

짝수를 필터링하는 코드를 선언형 프로그래밍 방식으로 작성한 예시를 살펴보자.

```java
public class DeclarativeExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        List<Integer> evenNumbers = numbers.stream()
                                           .filter(number -> number % 2 == 0)
                                           .collect(Collectors.toList());

        System.out.println(evenNumbers);
    }
}

```

앞서 살펴본 것처럼 동일하게 숫자 리스트가 주어졌지만, 스트림을 이용했다. filter를 이용해 필터링 조건을 선언하고, 데이터들을 흐름에 따라 처리한다. 동일하게 마지막으로 필터링한 짝수 리스트를 콘솔에 출력한다.

위 코드에서는 원하는 것이 명확히 나타나있다. 주어진 수 중에서 ‘짝수’를 필터링하는 것!

이처럼 선언형 프로그래밍은 코드에 **원하는 것이** ‘***무엇(WHAT)***’인지, ‘***무엇(WHAT)***’**을 해야 하는지**가 명확히 드러나는 프로그래밍 방식이다.

### 장점

1. 복잡한 로직을 간단히 표현할 수 있어 코드의 **가독성이 높아진다**.
2. **상태 변화를 최소화**하여 예측 가능한 코드를 작성할 수 있다.
3. 작은 함수 단위로 모듈화 하여 코드 재사용성이 높아진다.
4. 병렬 처리에 적합하다.

### 단점

1. 함수형 프로그래밍 개념과 선언형 프로그래밍 방식에 익숙하지 않은 개발자가 많다.
2. 람다식이나 고차 함수 등을 사용하기 떄문에 **디버깅이 어렵다**.
3. 내부적으로 더 많은 오버헤드를 가질 수 있어, **성능 최적화에 불리**하다.
4. 코드의 흐름이 명시적이지 않아 복잡한 로직에서는 **흐름 파악이 쉽지 않다**.

### 예상치 못한 변수값!?

많은 프로그래머가 유지보수 중 코드 크래시 디버깅 문제를 많이 겪는데, 코드 크래시는 예상치 못한 변수 값 때문에 발생할 수 있다. 그렇다면 **왜, 어떻게 변수값이 바뀐걸까?**

변수가 예상치 못한 값을 갖는 이유는 **여러 곳에서 공유된 가변 데이터를 읽고 수정**하기 때문이다. 어떠한 데이터도 바꾸지 않는다면, 예상치 못하게 데이터가 바뀔 일이 없으니 얼마나 좋을까?! 그래서 예상치 못한 값 변경을 막는 것이 유지보수에 용이하다. 

그렇다면 값 변경을 막기 위해선 어떻게 하면 좋을까? 🤔

불변 객체는 인스턴스화한 후 객체의 상태를 바꿀 수 없기 때문에 결코 예상치 못한 상태로 바뀌지 않는다. 따라서 불변 객체를 사용해 부작용을 없애는 방법도 있다.

**부작용 없는 시스템**의 개념은 함수형 프로그래밍에서 유래되었는데, 이 함수형 프로그래밍의 기반을 이루는 개념이 바로 선언형 프로그래밍이다 🙂

자, 그러면 이제 함수형 프로그래밍에 대해 알아보자!

## 함수형 프로그래밍

함수형 프로그래밍은 선언형 프로그래밍을 따르는 대표적인 방식이다. **상태 변화를 최소화하고,** 순수 함수를 이용해 **부작용 없이 데이터를 처리하는 방식을 지향한다.**

### 자바에서의 함수형 프로그래밍

자바에서는 자바 8부터 함수형 프로그래밍 개념이 도입되어, 람다식과 스트림을 이용해 함수형 프로그래밍 방식으로 쉽게 구현이 가능하다. 

자바 함수형 프로그래밍에서의 주요 키워드는 다음과 같다.

- **람다식**
    
    간결하게 익명 함수를 표현하여 코드를 명확하게 작성할 수 있다.
    
- **스트림**
    
    필터링, 매핑, 축소(reduce) 등의 데이터 연산을 선언적으로 처리한다.
    
- **불변성**
    
    상태 변화를 피하고, 변경할 수 없는 데이터를 사용해야 한다.
    
- **순수함수**
    
    동일한 입력에 대해 항상 동일한 출력을 반환하기 때문에 부작용이 없다.
    

### 함수형 자바

함수형 이라는 말은 ‘수학의 함수처럼 부작용이 없는’을 의미한다. 그리고 인수가 같다면 항상 같은 결과를 반환한다. 자바와 같은 언어에서는 바로 수학적인 함수냐 아니냐의 여부가 메소드와 함수를 구분하는 기준이 된다.

그렇다면 함수형 프로그래밍에서는 함수나 if-then-else와 같은 수학적 표현만 사용해야 할까? 시스템의 다른 부분에 영향을 미치지 않는다면 내부적으로는 함수형이 아닌 기능도 사용할 수 있을까?

그렇지 않다! 내부적으로 부작용이 발생했더라도 호출자가 이를 알지 못한다면*(알아차리지 못한다면)* 상관 없다 😉

따라서 **‘함수나 수학적 표현만 사용하는 방식’을 순수 함수형 프로그래밍**, **‘시스템에 다른 부분에 영향을 미치지 않는다면 내부적으로는 함수형이 아닌 기능도 사용하는 방식’을 함수형 프로그래밍**이라고 한다.

Scanner.nextLine()을 이용해 파일의 한 행을 읽어오는 경우를 생각해보자. 이 메소드를 두 번 호출하면 다른 결과가 반환될 수 있다. 즉, 하나의 함수를 호출 했을 때 항상 같은 결과가 반환되는 것이 아니라, **다른 결과가 반환**될 가능성이 있다는 것이다.

이처럼, 사실 **자바로는 완벽한 순수 함수형 프로그래밍을 구현하기는 어렵다**. 다만 시스템의 컴포넌트가 순수한 함수형인 것처럼 동작하도록 코드를 구현하는 것이다. 즉, 순수 함수형이 아니라 **함수형 프로그램을 구현한다**는 말이다! *실제로는 부작용이 있지만, 아무도 이를 알아차리지 못하게 함으로써 함수형을 달성할 수 있는 것이다 😉*

그러면 만약 진입 시 어떤 필드 값을 증가시켰다가 빠져나올 때 필드 값을 돌려놓는 메소드가 있다고 해보자. 싱글 스레드 환경에서는 이 메소드가 아무 문제를 일으키지 않기 때문에, 이 메소드는 함수형이라고 말할 수 있다.

하지만 멀티 스레드 환경에서 동시에 이 메소드를 호출하는 상황이 발생해 필드 값에 동시에 접근한다고 했을 때, 이 메소드는 함수형이 아니다. 

이 메소드에 락을 걸도록 수정해 문제를 해결하면 함수형이라고 할 수 있지만, 멀티코어 프로세서의 두 코어를 활용한다면 이 메소드를 병렬로 호출할 수 없게 된다. **프로그램 입장에서는 부작용이 사라졌지만, 프로그래머 관점에서는 프로그램의 성능이 저하된 것이다 😢**

### ‘함수형’ 이기 위해 만족해야 하는 조건들

그래서 ‘함수형’ 이라고 하려면, 다음과 같은 조건들을 만족해야 한다.

- 함수나 메소드는 **지역 변수만을 변경**해야 한다. 만약 참조하는 객체가 있다면, 그 객체는 **불변**이어야 한다.
- 예외가 발생하면 return으로 결과를 반환할 수 없기 때문에, **함수나 메소드가 **어떤 예외도 일으키지 않아야 한다.**
    
    ---
    💡 **예외를 사용하지 않고 함수를 표현하려면 어떻게 해야할까?**
    
    나눗셈 함수를 생각해보자. 입력 값에 따라 결과가 반환될수도, 예외가 발생할 수도 있다. 
    
    이런 경우에는 **`Optional<T>`**을 사용하면 된다. 
    
    `double sqrt(double num)` 대신 `Optional<Double> sqrt(double num)`을 사용하면, 예외 발생 없이 결과 값으로 연산이 성공했는지 아닌지를 확인할 수 있다.

    *( 정상적인 입력 값이 들어와 정상적인 연산 값이 반환된다면 해당 Optional 객체에 반환된 값이 존재할거고, 비정상 값이 들어와 예외가 발생했다면 해당 객체에 빈 값이 반환될 것이다. )*
    
    ---
    
- 비함수형 동작을 감출 수 있는 상황에서만 부작용을 포함하는 라이브러리 함수를 사용해야 한다. <br>
*( 발생할 수 있는 예외를 내부적으로 적절히 처리하여 호출자는 알 수 없도록 하는 것이다. 예를 들어, List를 인수로 받는 메소드에서 연산 전에 해당 리스트를 내부에서 복사해 사용하여 부작용을 감출 수 있다. )*


## 참고
- 모던 자바 인 액션 18장