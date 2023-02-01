---
title: Cloud Build를 이용한 GKE 자동 배포 설정하기 - 3
date: 2023-02-01 12:10:00 +0900
categories: [DevOps, Google Cloud Platform]
tags: [devops, gcp, gke, k8s, cloudbuild]
---

앞선 2편에서 GKE 배포 설정 파일(deployment.yaml)을 생성하여 각종 설정을 해주는 것 까지 진행해보았다.

이제 마지막으로 드디어! **Cloud Build로 GKE에 배포**하기 위한 여러 설정들을 해보자.


## 클러스터 생성
가장 먼저 Kubernetes 클러스터를 생성해주어야 한다.

구글 클라우드 콘솔 - Kubernetes Engine - 클러스터 페이지에 접속하여, 아래와 같이 만들기 버튼을 눌러 Autopilot 클러스터를 구성한다.<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_01.png){: width='600'}
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_02.png){: width='450'}

클러스터 이름을 입력하고, 네트워크 액세스는 비공개 클러스터로 선택한다.<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_03.png){: width='450'}
> **⚠️ 주의 ⚠️** <br>
네트워크 액세스를 비공개 클러스터로 설정하면, **pod에서 외부 네트워크로 접속이 되지 않는다.**
그래서 웬만하면 공개 클러스터로 설정할 것을 추천한다😅 <br>
_(세팅 과정에서 비공개 클러스터로 설정하면 보안 상 더 좋겠지! 싶어서 비공개로 설정했는데, gke에 띄우려는 애플리케이션에서 외부 api 요청하는 부분이 있었다.<br> 외부 api로 요청이 들어가지 않아 결국에는 공개 클러스터로 설정했다. 보안을 강화하고 싶다면 클러스터는 그냥 공개 클러스터로 설정하고, 방화벽 등의 다른 요소들을 활용하는 것이 운영하기에 덜 까다로울 것이다🥲)_
<br>

나머지 설정은 그대로 두고, 만들기 버튼을 눌러 클러스터를 생성한다.<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_04.png){: width='700'}


클러스터가 완전히 생성되는 데에는 시간이 조금 걸린다. 시간이 조금 지난 후에 아래와 같이 클러스터가 생성된 것을 확인할 수 있다.<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_05.png){: width='450'}

<br>


## 네임스페이스 생성
클러스터를 생성한 후, 네임스페이스를 생성해주어야 한다. _(네임스페이스를 생성하지 않고 배포를 진행하면, 네임스페이스를 찾을 수 없다는 에러가 발생한다.)_

터미널에서 아래 명령어를 실행하여 방금 생성한 클러스터의 접근 credential을 얻는다.
~~~ shell
gcloud container clusters get-credentials [CLUSTER_NAME] --zone [ZONE]
~~~

해당 클러스터의 네임스페이스 목록을 가져온다.
~~~ shell
kubectl get namespace
~~~
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_06.png){: width='550'}

deployment.yaml 파일에 명시해주었던 이름으로 네임스페이스를 생성한다.<br>
_(이전 글에서 deployment.yaml 파일에 네임스페이스 이름을 api로 명시해두었으므로 아래와 같이 api라는 이름으로 네임스페이스를 생성한다.)_
~~~ shell
kubectl create namespace api
~~~
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_07.png){: width='550'}

<br>


## Cloud Build에 GKE IAM 권한 추가
Cloud Build에서 GKE 배포를 진행하려면, Cloud Build에 권한 추가를 해주어야 한다.

**구글 클라우드 콘솔 - Cloud Build - 설정** 에 접속한다.

Kubernetes Engine 개발자 역할의 상태를 **사용 설정**으로 변경한다.<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_08.png){: width='400'}

<br>


## GKE 배포 Step 추가
이제 앞서 생성했던 Cloud Build 트리거에 GKE 배포 Step을 추가해보자.

**구글 클라우드 콘솔 - Cloud Build - 트리거 - 앞서 생성했던 트리거의 수정** 페이지 에 접속한다.

인라인 편집기를 열어 아래 내용을 가장 하단에 추가하고, 완료 및 저장 버튼을 눌러 트리거를 수정한다.<br>
*(프로젝트 루트에 위치한 development.yaml 파일을 읽어 [ZONE] 리전에 있는 [CLUSTER_NAME]에 배포하라는 의미)*
~~~ yaml
- name: gcr.io/cloud-builders/gke-deploy
    args:
      - run
      - '--filename=deployment.yaml'
      - '--location=[ZONE]'
      - '--cluster=[CLUSTER_NAME]'
~~~

<br>


## GKE 배포 확인
**구글 클라우드 콘솔 - Cloud Build - 기록** 에 접속하여, 최상단에서 실행 중인 빌드 기록으로 들어간다. _(빌드 과정을 실시간으로 확인할 수 있다!)_

해당 빌드가 성공적으로 완료되었다면, **구글 클라우드 콘솔 - Kubernetes Engine** 에서 아래 내용들을 확인해보자.
1. **작업 부하** 메뉴를 클릭하고, **워크로드**가 잘 생성되었는지 확인<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_09.png){: width='550'}
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_10.png){: width='650'}<br>

2. **서비스 및 수신 - 인그레스** 메뉴를 클릭하고, **인그레스 객체**가 잘 생성되었는지 확인<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_11.png){: width='700'}<br>

3. 생성된 인그레스를 눌러 **프론트엔드 및 백엔드 설정**이 잘 되어있는지 확인<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_12.png){: width='600'}<br>

4. **서비스 및 수신 - 서비스** 메뉴를 클릭하고, **서비스 객체**가 잘 생성되었는지 확인
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_13.png){: width='600'}
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_14.png){: width='600'}<br>

5. **(선택사항)** &nbsp; Filestore가 지정 폴더에 마운트되었는지 확인<br>
a. 현재 실행중인 파드에 접속
~~~ shell
kubectl exec -it -n [NAMESPACE] pod/[POD_NAME] -- bash
~~~
b. 지정한 폴더로 이동하여 파일이 제대로 마운트 되었는지 확인<br>

<br>


## 도메인 추가
인증서 프로비저닝을 위해 인그레스 IP와 ManagedCertificated 객체에 설정해 준 도메인을 매핑해주어야 한다.

인그레스 목록에서 인그레스 IP를 확인하여 복사해둔다.<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_15.png){: width='500'}

그리고 나서 사용중인 호스팅 서비스 사이트에 접속하여, 도메인과 IP주소를 매핑해주면 된다.

인증서가 프로비저닝 되는 데에 약간의 시간이 걸린다. 잠시 기다린 후, **부하분산 - 인증서** 목록에서 아래와 같이 프로비저닝이 완료된 것을 확인할 수 있다.
_(부하 분산 페이지에서 하단의 **부하분산 구성요소 뷰**를 클릭하면 인증서 탭에 접근할 수 있다.)_<br>
![image](/assets/img/post/devops/gcp/230201_cloudbuild를-이용한-gke-자동-배포-설정하기-3/screenshot_16.png){: width='700'}

이제 설정해 둔 도메인으로 접속하면, 정상적으로 서버가 실행되고 있음을 확인할 수 있다!

<br>


## 참고 사이트
- <https://cloud.google.com/kubernetes-engine/docs/concepts/ingress>
- <https://seongjin.me/kubernetes-service-types/>
- <https://cloud.google.com/filestore/docs/accessing-fileshares>
- <https://cloud.google.com/build/docs/deploying-builds/deploy-gke?hl=ko>