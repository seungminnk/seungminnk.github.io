---
title: final과 의존성 주입
date: 2024-06-27 18:04:00 +0900
categories: [Spring]
tags: [spring, di, final, java]
---

컨트롤러에 Service 클래스를 선언하고, @AllArgsConstructor를 클래스 상단에 선언하여 생성자 주입을 받을 수 있도록 했다. 그런데 다음과 같이 IntelliJ에서 해당 변수에 마크 표시가 되어 있었다.
<br>
![image](/assets/img/post/spring/240627_final과-의존성-주입/screenshot_01.png){: width='300'}


표시된 부분에 커서를 올려보니 해당 필드를 final로 선언할 수 있다는 메시지가 떴다. <br>
_( 좀 더 자세한 설명을 보기 위해 우측의 … 을 눌러 Show inspection description을 클릭해, 다음과 같은 메시지를 확인할 수 있었다. )_
<br>
![image](/assets/img/post/spring/240627_final과-의존성-주입/screenshot_02.png){: width='600'}

클래스의 필드를 final로 선언할 수 있는 조건에 대해 설명하고 있는데, 사용하고자 하는 필드는 non-static 필드이므로 해당 부분만 요약하자면 다음과 같다.

> 선언 시점에 초기화되거나, 하나의 비정적 클래스 초기화 블록에서 초기화되거나, 생성자에서 초기화되어야 해당 필드를 final로 만들 수 있다.

~~~ java
public class Example {
    // 선언 시점에 초기화
    private final int instanceFinalField = 10;

		// 인스턴스 초기화 블록에서 초기화
    {
        instanceFinalField = 10; 
    }

    // 생성자에서 초기화
    public Example() {
        this.instanceFinalField = 10; 
    }

		// 모든 필드를 파라미터로 받는 생성자에서 초기화
    public Example(int value) {
        this.instanceFinalField = value; 
    }
}
~~~

## 아니 그래서, final을 붙였을 때 왜 마크 표시가 사라지냐구! 🤔
@AllArgsConstructor 어노테이션*(모든 필드를 파라미터로 받는 생성자)*을 통해 해당 필드에 객체를 주입받는다. 

생성자를 통해 객체를 주입받기 때문에 생성자가 호출되는 최초 1번만 실행되고, 이는 해당 필드를 **final로 선언할 수 있는 조건에 해당되기 때문**에 마크가 사라진 것!
<br>

## 그럼 여기서 하나 더, final로 선언하면 뭐가 좋지?
*(해당 필드를 final로 선언할 수 있다는거지, 반드시 그래야 한다는건 아니잖아?)*

1. **생성자 주입을 사용해야 final 키워드를 사용해 불변으로 만들 수 있다.** 
    - 다른 방식들은 객체 생성*(생성자 호출)* 이후에 의존성이 주입되므로 final 키워드를 사용할 수 없다.
2. **컴파일 시점에 누락된 의존성을 확인할 수 있다.**
3. **final로 선언해서 변경될 수 없도록 만든다.** 
    - final로 선언하지 않으면 변경 가능해지고, 해당 변수에 다른 인스턴스를 넣어 변경할 수 있게 된다. 그래서 스프링 컨테이너에서 관리하는 객체(Bean)를 해당 필드에 주입하고, 그 이후 변경되지 못하도록 final을 선언하는 것이다.


## 참고
- <https://velog.io/@tjdtn4484/Spring에서의-DI-feat.-생성자-주입-final-영한님-짱>
- <https://velog.io/@tjdtn4484/스프링-DI의존성-주입와-필드-변수의-finalfeat.-생성자-주입>
- <https://mangkyu.tistory.com/125>
