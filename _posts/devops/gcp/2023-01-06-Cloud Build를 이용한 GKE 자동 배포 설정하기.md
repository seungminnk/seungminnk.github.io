---
title: Cloud Build를 이용한 GKE 자동 배포 설정하기
date: 2023-01-06 16:26:00 +0900
categories: [DevOps, Google Cloud Platform]
tags: [devops, gcp, gke, k8s, cloudbuild]
---

## 들어가기 전 💬
우리 회사는 현재 클라우드 서비스인 GCP(Google Cloud Platform)를 활용해 운영 중이다.<br>

특히 우리 서비스 중 한 API 서버는 GCE(Google Compute Engine) 인스턴스를 이용하고, 부하 분산기를 붙여 오토 스케일링이 가능하도록 운영 중이었는데,
최근에 이 서버를 배포, 관리, 확장하기 쉽도록 이 API 서버에 GKE(Google Kubernetes Engine)를 도입해 운영하게 되었고, 이로 인해 내가 GKE 배포 환경 구성을 맡게 되었다.

기존 GCE 인스턴스를 이용할 때, 빌드와 배포는 각 명령어를 쉘 파일로 구성해두고 인스턴스에 접속해 해당 쉘 파일을 실행하여 개발자가 수동으로 배포하도록 구성되어 있었다.
이런 방식으로 배포를 진행하다보니 배포 과정에서 코드가 꼬이거나, 예상치 못한 에러가 발생하는 경우가 종종 발생했었다.<br>

그래서 수동으로 배포하며 발생하는 문제들을 해결하고, 보다 쉽게 배포를 진행할 수 있도록 GCP의 **Cloud Build** 서비스를 이용하여 환경을 구성해보았다.<br>

> 아무튼! 이번 글에서는 GKE로 전환하며 겪은 **Cloud Build를 활용한 GKE 자동 배포 환경 구축 과정**을 정리해보았다.😃
<br>

<br>

## Cloud Build 란?
먼저 Cloud Build에 대해 간단히 설명하고 들어가보자!<br>

Cloud Build는 **Google Cloud에서 빌드를 실행하는 서비스**인데, 다양한 저장소에서 소스코드를 가져와 사양에 맞게 빌드를 실행하고, Docker 컨테이너 또는 자바 아카이브와 같은 아티팩트를 생성할 수 있다.
<br><br>
자, 그러면 이제 Cloud Build를 이용해 애플리케이션을 빌드하고, GKE에 배포하는 방법을 알아보자!


Cloud Build 트리거는 다음과 같은 단계로 구성한다.
1. Docker 컨테이너 이미지 생성
2. Container Registry에 Docker 이미지 푸시
3. GKE 배포
<br><br>


## Dockerfile 작성
Docker 이미지 생성을 위해서는 **프로젝트 루트 폴더**에 Dockerfile을 생성해야 한다.

PHP 애플리케이션의 이미지를 생성할 것이기 때문에, 아래와 같은 형식으로 Dockerfile을 구성하였다.<br>
_(만약 다른 언어로 되어 있는 애플리케이션(프로젝트)을 빌드하려는 경우, 각각에 맞는 Dockerfile을 작성해주면 된다!)_
~~~ Dockerfile
FROM php:7.4-apache
RUN docker-php-ext-install mysqli

... (중략) ...

RUN chmod 0755 /root/start.sh
RUN /root/start.sh
~~~


## Docker 이미지 생성
### Cloud Build 트리거 생성
Docker 이미지 생성에 앞서, 이미지 생성과 배포를 진행할 Cloud Build 트리거부터 만들어보자.

구글 클라우드 콘솔 - Cloud Build - 트리거 페이지로 접속한 후, 상단의 **트리거 만들기**를 클릭한다.
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기/screenshot_01.png){: width='400'}

트리거 이름을 입력하고, 이 트리거를 어느 경우에 호출할 지 선택한다.<br>
이벤트 설정은 **수동 호출**로 선택한다. _(특정 브랜치 혹은 새 태그가 원격 저장소에 푸시되었을 때 트리거가 자동으로 호출되도록 할 수도 있다!)_ <br>
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기/screenshot_02.png){: width='450'}

수동 호출을 클릭하면, 바로 하단 부분(소스)이 아래와 같이 활성화되는데, 소스코드를 클론할 **원격 저장소**와 **기준 브랜치**를 설정한다.<br>
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기/screenshot_03.png){: width='450'}