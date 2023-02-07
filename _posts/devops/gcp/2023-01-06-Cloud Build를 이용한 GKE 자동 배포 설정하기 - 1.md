---
title: Cloud Build를 이용한 GKE 자동 배포 설정하기 - 1
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
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_01.png){: width='400'}

트리거 이름을 입력하고, 이 트리거를 어느 경우에 호출할 지 선택한다.<br>
이벤트 설정은 **수동 호출**로 선택한다. _(특정 브랜치 혹은 새 태그가 원격 저장소에 푸시되었을 때 트리거가 자동으로 호출되도록 할 수도 있다!)_ <br>
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_02.png){: width='450'}

수동 호출을 클릭하면, 바로 하단 부분(소스)이 아래와 같이 활성화되는데, 소스코드를 클론할 **원격 저장소**와 **기준 브랜치**를 설정한다.<br>
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_03.png){: width='450'}

다음은 구성 설정이다.<br>

Cloud Build 구성 파일을 선택하고, 해당 파일을 소스코드에서 불러올지 트리거 설정에 인라인으로 작성할지 설정한다.<br>
여기에서는 트리거에 인라인 yaml을 작성하도록 선택하자.<br>
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_04.png){: width='300'}

아래의 **편집기 열기** 버튼을 눌러 다음과 같이 작성해준다.<br>
*(snapshot, SHORT_SHA 태그를 달아 컨테이너를 빌드하여 이미지를 생성하라는 의미이다.)*
~~~ yaml
steps:
  - name: gcr.io/cloud-builders/docker
    args:
      - build
      - .
      - '-t'
      - 'gcr.io/$PROJECT_ID/[REGISTRY_NAME]:snapshot'
      - '-t'
      - 'gcr.io/$PROJECT_ID/[REGISTRY_NAME]:${SHORT_SHA}'
~~~

> `참고` **Java Jib**<br>
java의 경우, GCP에서 **Jib** 이라는 라이브러리를 제공한다.<br>
Dockerfile을 작성하고, Docker를 설치해 이미지를 빌드할 필요 없이 Jib을 이용해 컨테이너를 빌드할 수 있다.<br>
_(<https://cloud.google.com/java/getting-started/jib?hl=ko>)_
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_05.png){: width='600'}



## Container Registry에 이미지 푸시
앞서 Docker 이미지까지 생성했으니, 이제 Registry에 이미지를 푸시하는 방법에 대해 알아보자.

인라인 편집기를 열어 이전에 작성했던 내용 아래에 다음 내용을 작성한다.<br>
*(레지스트리에 모든 태그를 붙여 이미지를 푸시하라는 의미이다.)*
~~~ yaml
- name: gcr.io/cloud-builders/docker
    args:
      - push
      - gcr.io/$PROJECT_ID/[REGISTRY_NAME]
      - '--all-tags'
~~~

편집기 하단의 완료를 누른 후, 트리거 설정 페이지 하단의 저장 버튼을 눌러 저장한다.

생성된 트리거 우측의 실행을 누르고, 하단의 트리거 실행을 누르면 트리거가 실행된다.<br>
*(분기 부분에 다른 브랜치 이름을 입력해 실행하면, 해당 브랜치 기준으로 빌드를 실행할 수도 있다.)*<br>
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_06.png)
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_07.png){: width='300'}

트리거를 실행한 후, 좌측 메뉴의 **기록** 페이지로 접속하여 빌드 히스토리를 확인할 수 있다.

가장 상단에 실행 중인 빌드를 클릭하여 빌드 실행 상태를 실시간으로 확인가능하다.
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_08.png)
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_09.png)


빌드가 성공적으로 완료되었다면, Container Registry에 이미지가 잘 푸시되었는지 확인해보자.

**Container Registry - 이미지** 페이지로 접속하면 다음과 같이 저장소가 생성되어 있다.
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_10.png){: width='400'}

저장소 이름을 클릭하면 Docker 이미지가 생성되어 있는 것을 확인할 수 있다.
![image](/assets/img/post/devops/gcp/230106_cloudbuild를-이용한-gke-자동-배포-설정하기-1/screenshot_11.png){: width='600'}


<br>

### 다음 글 > [GKE 배포 설정 구성](<http://localhost:4000/posts/Cloud-Build%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-GKE-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-2/>)