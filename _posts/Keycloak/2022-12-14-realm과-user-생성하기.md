---
title: Realm과 User 생성하기
date: 2022-12-14 18:01:00 +0900
categories: [Keycloak]
tags: [keycloak, authentication]
---

관리자 콘솔에 접속할 수 있게 되었으니, 이제 Realm과 User를 생성해보고 사용자 콘솔에 로그인 해보도록 하자.

## Realm, Client, User
먼저 들어가기 전에 Keycloak의 Realm, Client, User에 대해 알아보자.
- Realm
: 사용자, 인증, 인가, 권한, 그룹이 **관리하는 범위**이다. 각각의 Realm은 독립적이고, 한 Realm 내의 Client들은 서로 SSO를 공유한다.
- Client
: 인증, 인가 업무를 Keycloak에게 요청할 수 있는 **어플리케이션** 또는 **서비스**를 뜻한다. Keycloak은 Client에게 보안이나 SSO를 제공한다.
- User
: **Client를 이용하는 사용자**를 뜻한다. 사용자는 Realm 단위로 구성되어 있다.<br>
  _(예를 들어, A라는 Realm에 가입된 X라는 사용자는 각각의 Client에서 X의 아이디 하나로 로그인이 가능하지만, B라는 Realm에는 가입되어 있지 않으므로 로그인이 불가능하다.)_
<br>


## Realm 생성하기
관리자 콘솔 _(<http://localhost:8080/admin>)_ 에 접속하고 로그인하면 다음과 같은 페이지가 나타난다.<br>
Realm 생성을 위해 좌측 상단의 Select box를 누르고, **Create Realm** 버튼을 누른다.
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_01.png">

Realm 이름을 입력한 후 Create를 누르면 Realm이 생성되고, 해당 Realm 관리 콘솔로 넘어간다. 
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_02.png">
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_03.png">
<br>


## User 생성하기
앞서 생성한 Realm에 로그인할 수 있는 User를 생성해보자.

좌측 메뉴의 Users 메뉴 클릭, Create new user 버튼을 클릭한다.
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_04.png">

생성할 User의 이름을 입력하고, Create를 눌러 user를 생성한다.
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_05.png">

그 다음으로, 로그인을 위해서는 해당 user 계정의 비밀번호를 설정해주어야 한다.<br>
앞서 생성한 user 상세 페이지에서 Credentials 탭 클릭, Set password 버튼을 누른다.
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_06.png">

설정할 비밀번호를 입력하고, 하단의 Temporary는 Off로 설정하여 Save 버튼을 눌러 저장한다.<br>
_(Temporary를 On으로 설정하면, 최초 로그인 이후 비밀번호를 재설정하도록 한다.)_
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_07.png" width="400">
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_08.png">

이제 생성한 User 계정으로 로그인을 시도해보자.<br>
사용자 콘솔 _(http://localhost:8080/realms/[생성한Realm이름]/account)_ 에 접속하고, 우측 상단의 Sign in 버튼을 눌러 로그인 페이지로 넘어간다.
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_09.png">
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_10.png" width="400">

설정해주었던 username과 password로 로그인 후, 다음과 같은 페이지로 넘어간다면 로그인이 성공적으로 완료된 것이다!
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_11.png">

<br>


## Keycloak 서버 재실행 시, 설정 값 유지되는지 확인해보기
> 이전 포스트에서 서버 재실행 시에 설정했던 값들이 초기화되지 않도록 외부 데이터베이스를 연결하여 Keycloak 서버를 실행하는 방법에 대해 설명했었다.<br> 
서버를 종료했다가 다시 재실행하면 앞서 생성했던 Realm과 User가 그대로 존재하는지 확인해보도록 하자 😃

먼저, 실행 중인 Keycloak 서버를 종료시키고, 다음 명령어로 다시 Keycloak 서버를 띄운다.
~~~ shell
docker run --name keycloak -p 8080:8080 keycloak-demo start-dev
~~~
관리자 콘솔에 접속하고 로그인한 후, 앞서 생성했던 Realm이 존재하는 것을 확인할 수 있다.
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_12.png" width="400">

좌측 상단 Select box를 눌러 생성했던 Realm을 선택하고, Users 탭에 접속한다. 생성했던 user가 존재하는 것을 확인할 수 있다.<br>
<img src="/assets/img/post/keycloak/221214_realm과-user-생성하기/screenshot_13.png" width="650">
<br>


## 참고사이트
- <https://soojae.tistory.com/47>
- <https://www.keycloak.org/getting-started/getting-started-docker>