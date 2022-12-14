---
title: Docker로 Keycloak 실행하기
date: 2022-12-03 17:28:00 +0900
categories: [Keycloak]
tags: [keycloak, authentication]
---

[Keycloak 공식 문서](https://www.keycloak.org/guides)의 _Get Started_ 파트에는 Keycloak 서버를 실행할 수 있는 여러 방법을 소개하고 있는데, 이 글에서는 **Docker**를 이용해 Keycloak을 실행하는 방법에 대해 설명하려고 한다.

## Docker 설치하기
먼저 Mac 환경에 Docker를 설치해보자. 간단히 [Docker Desktop](https://docs.docker.com/desktop/install/mac-install/)을 설치하기만 하면 된다.
Intel/M1 여부에 따라 dmg 파일을 선택하여 다운로드 받고, Docker Desktop을 설치한다.

설치 후에 Docker Desktop을 실행하면 이렇게 Docker Desktop starting...이 나타난다. 잠시 기다리면 로딩이 완료되어 Docker가 실행되어 있는 상태가 되는데, 상단 상태표시바에 아래와 같이 확인할 수 있다.
<img src="/assets/img/post/keycloak/221203_docker로-keycloak-실행하기/screenshot_01.png">
<img src="/assets/img/post/keycloak/221203_docker로-keycloak-실행하기/screenshot_02.png" width="300" height="450">


## 기존 이미지로 Keycloak 서버 실행하기
Keycloak에서 Docker 이미지를 제공하고 있기 때문에, 이 이미지를 가져와 Keycloak을 실행할 수 있다. 다음 명령어를 실행해보자.
~~~ shell
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:20.0.1 start-dev
~~~
- **8080**번 포트로 Keycloak 서버 실행
- **KEYCLOAK_ADMIN** : 관리자 콘솔 로그인을 위한 마스터 계정 아이디
- **KEYCLOAK_ADMIN_PASSWORD** : 관리자 콘솔 로그인을 위한 마스터 계정 비밀번호
- **quay.io/keycloak/keycloak:20.0.1** : Keycloak에서 제공하는 Docker 이미지
- **start-dev** : 개발 환경으로 실행 _(운영 환경일 시, start)_


위 명령어로 keycloak 서버를 실행시키면, 관리자 콘솔 _(<http://localhost:8080/admin>)_ 로 접속 가능하다.<br>
docker run 할 때 설정해주었던 마스터 계정 아이디/비밀번호로 로그인해보자!<br>
<img src="/assets/img/post/keycloak/221203_docker로-keycloak-실행하기/screenshot_03.png" width="450" height="350">
<img src="/assets/img/post/keycloak/221203_docker로-keycloak-실행하기/screenshot_04.png">


## 데이터베이스 커스텀 설정 추가하여 Docker 이미지 생성하기
Keycloak에서 제공하는 기본 이미지로 실행해 사용하면 문제가 하나 있는데, 이는 서버가 중단(종료)되는 경우이다.

Keycloak은 실행될 때 자체적으로 H2 데이터베이스를 사용하여 관련 데이터를 모두 저장한다. 만약 서버가 중단(종료)되었다가 다시 실행된다면, 기존에 사용하던 데이터베이스는 날아가고 **데이터베이스를 새로 생성**하게 된다. 이로 인해 기존에 설정했던 내용이나 저장했던 데이터가 모두 없어져 초기화된다.

이러한 이유로 Keycloak 자체에서 사용하는 H2 데이터베이스가 아닌 **외부 데이터베이스**에 연결하여 사용할 수 있도록 설정하는 것이 좋다.

Keycloak에서 제공하는 이미지를 기반으로 하되, 여기에 외부 데이터베이스를 연결하는 설정을 추가하여 새로운 Docker 이미지를 생성해보도록 한다.<br><br>

> 먼저 Keycloak에 연결할 데이터베이스를 생성해두고, url/username/password 값은 잘 기억해두자.

<br>
원하는 디렉토리로 이동하여 Dockerfile을 생성하고, 아래와 같이 작성 후 저장한다.
~~~ Dockerfile
FROM quay.io/keycloak/keycloak:latest

ENV KC_DB=mysql
ENV KC_DB_URL=jdbc:mysql://host.docker.internal:3306/keycloak
ENV KC_DB_USERNAME=keycloak
ENV KC_DB_PASSWORD=keycloak
ENV KEYCLOAK_ADMIN=admin
ENV KEYCLOAK_ADMIN_PASSWORD=admin

# ENTRYPOINT에서 kc.sh을 사용하면 모든 하위 배포 명령어 실행 가능
ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
~~~
- **KC_DB** 
  : 연결할 외부 데이터베이스 종류 _(mysql, mariadb, oracle, postgres...)_

- **KC_DB_URL** 
  : 데이터베이스 URL
    <details>
    <summary>&nbsp;<b>host.docker.internal</b></summary>
    <div markdown="1">
      localhost:3306~으로 설정하면 **Docker 내부***(게스트 PC)***의 localhost로 인식**하기 때문에, 내 PC 로컬(호스트 PC)의 데이터베이스로 연결되지   않는다.<br>
      localhost 대신 **host.docker.internal**로 설정해주면, Docker에서 **호스트 PC의 localhost로 인식**하여 정상적으로 데이터베이스에 연결할 수 있게 된다!
    </div>
    </details>

- **KC_DB_USERNAME** 
  : 데이터베이스 사용자

- **KC_DB_PASSWORD** 
  : 데이터베이스 사용자 패스워드

- **KEYCLOAK_ADMIN** 
  : 관리자 콘솔 로그인을 위한 마스터 계정 아이디<br>_(기존 이미지로 서버 실행 시 환경변수로 설정해주었던 KEYCLOAK_ADMIN과 동일)_

- **KEYCLOAK_ADMIN_PASSWORD** 
  : 관리자 콘솔 로그인을 위한 마스터 계정 비밀번호<br>_(기존 이미지로 서버 실행 시 환경변수로 설정해주었던 KEYCLOAK_ADMIN_PASSWORD와 동일)_ <br><br>


터미널을 켜고 생성한 Dockerfile이 위치한 디렉토리로 이동하고, 다음 명령어를 실행해 Docker 이미지를 만든다.<br>
_(keycloak-demo 라는 이름의 Docker 이미지가 생성된다.)_
~~~ shell
docker build . -t keycloak-demo
~~~
<br>


## 커스텀 Docker 이미지로 Keycloak 서버 실행하기
앞서 생성한 Docker 이미지를 실행하여, 외부 데이터베이스에 연결된 Keycloak 서버를 띄울 수 있다.

다음 명령어로 앞서 생성한 이미지를 docker로 실행한다.
_(**keycloak-demo** 이미지로 **8080**번 포트에서 **keycloak**이라는 이름으로 실행된다.)_
~~~ shell
docker run --name keycloak -p 8080:8080 keycloak-demo start-dev
~~~
<br>


## 관리자 콘솔 접속하기
외부 데이터베이스에 연결한 Keycloak 서버를 띄웠으니, 이제 관리자 콘솔에 접속하여 로그인이 잘 되는지 확인해보자.

앞서 했던 것처럼 다시 관리자 콘솔 _(<http://localhost:8080/admin>)_ 에 접속하고, Dockerfile에 설정했던 username/password로 로그인하면 정상적으로 콘솔에 접속할 수 있다.<br>
<img src="/assets/img/post/keycloak/221203_docker로-keycloak-실행하기/screenshot_05.png">

> Keycloak 서버를 종료한 후 다시 재실행했을 때, 설정했던 여러 값들이 여전히 유지되는지는 다음 포스트에서 **Realm과 User 설정 후 확인**해보자! 😃

<br>


## 참고사이트
- <https://docs.docker.com/desktop/install/mac-install/>
- <https://www.keycloak.org/getting-started/getting-started-docker>
- <https://www.keycloak.org/server/containers>