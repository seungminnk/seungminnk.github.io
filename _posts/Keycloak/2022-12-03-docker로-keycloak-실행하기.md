---
title: Docker로 Keycloak 실행하기
date: 2022-12-03 17:28:00 +0900
categories: [Keycloak]
tags: [keycloak, authentication]
---

[Keycloak 공식 문서](https://www.keycloak.org/guides)의 _Get Started_ 파트에는 Keycloak 서버를 실행할 수 있는 여러 방법을 소개하고 있는데, 이 글에서는 **Docker**를 이용해 Keycloak을 실행하는 방법에 대해 설명하려고 한다.

## Docker 이미지 생성하기
Keycloak에서 Docker 이미지를 제공하고 있기 때문에, 이 이미지를 가져와 Keycloak을 실행할 수 있다. 다음 명령어를 실행해보자.
~~~ shell
$ docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:20.0.1 start-dev
~~~
- **8080**번 포트로 Keycloak 서버 실행
- **KEYCLOAK_ADMIN** : 관리자 콘솔 로그인을 위한 마스터 계정 아이디
- **KEYCLOAK_ADMIN_PASSWORD** : 관리자 콘솔 로그인을 위한 마스터 계정 비밀번호
- **quay.io/keycloak/keycloak:20.0.1** : Keycloak에서 제공하는 Docker 이미지
- **start-dev** : 개발 환경으로 실행 _(운영 환경일 시, start)_




## 참고사이트
<https://www.keycloak.org/getting-started/getting-started-docker>