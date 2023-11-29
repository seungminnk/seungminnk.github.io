---
title: Cloud SQL Proxy로 비공개 DB 접속하기
date: 2023-03-29 12:06:00 +0900
categories: [Google Cloud Platform]
tags: [devops, gcp, cloudsqlproxy, cloudsql, mysql, cloudbuild]
---

## 들어가기 전 💬
현재 회사에서 두 서비스의 API 통합 작업을 진행하고 있는데, 통합 과정에서 몇 가지 문제점이 발생했다. 이 문제들을 해결하기 위해서 어떤 방법을 사용했고, 어떻게 해결했는지를 정리해보고자 한다 🤓
<br><br>

먼저 현재 두 서비스가 어떤 구조로 구성되어 있는지부터 설명해보자면 다음과 같다.
- A 서비스는 **Proj-A**에 모든 인프라가 위치해있다.
    - **Api-A** : A 서비스의 API, **GKE**로 운영 중이다.
    - **Db-A** : Api-A와 연결되어 있는 데이터베이스, Cloud SQL로 운영 중이다.
- B 서비스는 **Proj-B**에 모든 인프라가 위치해있다.
    - **Api-B** : B 서비스의 API, **GCE 인스턴스**로 운영 중이다.
    - **Db-B** : Api-B와 연결되어 있는 데이터베이스, Cloud SQL로 운영 중이다.
- **Api-A에 Api-B를 통합**하는 작업을 진행하려한다.

즉 정리하자면, 두 시스템이 **서로 다른 GCP 프로젝트에 각각 구성**되어 있어 **각 프로젝트의 리소스끼리의 접근에 제한이 있다**는 것이다.

이로 인해 다음과 같은 문제가 발생하였고, 이를 해결하기 위한 해결책이 필요한 상황이었다.
- Api-A에서 두 개의 데이터베이스 _(Db-A & Db-B)_ 로 동시에 JDBC Datasource 연결을 물고 있어야 한다. 그래서 **Api-A에서 Db-B로의 접속이 가능**해야 한다. <br>
- 하지만 **Proj-A의 Api-A에서 Proj-B의 Db-B에 접근할 수 없다.** <br>
_(다른 GCP 프로젝트끼리는 별도의 VPC로 구성되어 있으므로, 다른 프로젝트 리소스에 접근하기 위해서는 별도의 네트워크 설정이 필요하다.)_

### 본격적인 문제 해결 전, 마주친 또 다른 문제 상황 😭
위에서 언급했던 문제 상황을 본격적으로 해결하기 전에 Db-A를 살펴보다가 또 다른 문제 상황을 직면했다.

현재 Api-A에서는 빌드 시에 테스트 코드가 실행된다. 이 때 Db-A의 테스트 실행용 스키마에 접근하기 위해 Db-A의 외부 IP로 접근하는데, Db-A에 모든 IP를 허용(0.0.0.0/0)하도록 설정되어 있었다! 😨 <br>
모든 IP를 허용하는 것은 보안상 위험하다고 판단하여 해당 설정을 제거했는데, 이로 인해 테스트 코드가 실행될 때 Db-A에 접속할 수 없어 빌드 실패가 발생하였다.

Api-A의 빌드와 배포는 Cloud Build를 이용하는데 이 문제로 빌드 실패가 발생해 배포까지 이루어지지 못했고, 때문에 이 이슈를 먼저 해결한 후 앞서 언급했던 문제들을 해결하고자 했다.

### gradle test 성공을 위한 시도 방법들
그래서 gradle test 성공을 위해 다음과 같이 여러 방법을 시도해보았다.<br><br>


**1. Cloud Build 서비스 계정에 SQL 권한 부여하기**

처음에는 가장 먼저 접근 권한을 의심해보았다. 

같은 GCP 프로젝트 내에서는 기본적으로 모든 리소스가 같은 네트워크 상에 존재하기 때문에, 별도 네트워크 설정 없이 접근이 가능하다.

그래서 Cloud Build와 Db-A는 같은 Proj-A에 있어 각 리소스들끼리는 접근이 가능한데, Cloud Build의 서비스 계정이 SQL에 접근할 수 있는 권한이 없어 Db-A에 접속하지 못하는 것이라고 생각했다. 

GCP 콘솔 > IAM > 서비스 계정 메뉴에 접속하여 Cloud Build의 서비스 계정에 **SQL 클라이언트** 권한을 부여해주고, 다시 Cloud Build를 실행하여 빌드를 시도해보았다.

하지만 SQL 권한을 부여해주어도 여전히 Db-A에 접근할 수 없었다.
<br><br>

**2. Cloud Build의 IP 알아내기**

권한 문제는 아니고, 그렇다면 그 다음은 네트워크 문제를 의심해보았다.

Cloud Build에서 gradle test가 실행될 때 Db-A로의 접근이 불가능한 상황이니, **Cloud Build가 실행되는 곳의 IP가 Db-A에 허용되지 않아 접속하지 못하는 것일 수 있겠다**는 생각이 들었다.

그래서 Cloud Build에 ifconfig 명령어를 실행하여 출력하는 step을 추가하여 IP 주소를 알아내고, 그 IP 주소를 Db-A에 허용 IP로 추가해주었다.

하지만 이 방법도 실패했다 😥 Cloud Build는 **실행될 때마다 IP 주소가 변경**되었고, 이전 빌드에서 IP 주소를 알아내 Db-A에 설정해주어도 새로 빌드를 실행할 때는 다른 IP 주소를 가지고 실행되었기 때문에 이 방법으로는 해결할 수 없었다.
<br><br>

**3. Cloud Build의 IP 대역 알아내기**

실행할 때마다 Cloud Build의 IP가 바뀐다면, **IP가 할당되는 IP 대역을 알아내어 Db-A에 해당 대역을 추가하면 되겠다!**라는 생각이 들었다. IP가 바뀌더라도 해당 대역 내 IP라면 접속이 가능하겠지!라는 생각으로 Cloud Build가 실행되는 IP 대역을 알아내보기로 했다.

Cloud Build의 IP가 할당되는 IP 대역 정보를 구글에 검색해보기도 하고, Chat GPT를 이용해 알아보려고도 했지만 명확한 IP 대역을 알 수 없었다. 그래서 2에서 했던 방식대로 빌드를 여러 번 실행해 직접 IP를 알아내고, 알아낸 IP들로 IP 범위를 추려낼 수 있을 것 같아 시도해보았지만 역시나 범위를 특정지을 수 없이 **무작위로 IP가 할당**되는 것을 확인할 수 있었고, 결국 이 방법도 실패했다.
<br><br>

**4. Cloud SQL Proxy 이용하기 ⭐️**

앞서 시도했던 방법들이 모두 실패하고 난 후, 테스트 코드 실행 시에만 접근하는 DB를 생성하여 해당 DB는 모든 IP를 허용하도록 설정해두어야 하는 방법 밖에 없는걸까, 아니면 테스트 코드 실행할 때 실제 DB가 아닌 H2 같은 메모리 DB를 사용하도록 코드를 변경해야 할까 등 여러 생각을 하고 있었고, GCP 문서도 읽어보고 Chat GPT를 통해서도 해결 방법을 계속 찾아보고 있었다. 비공개 풀이나 VPC Connector를 사용해야 하는지도 고민하던 중에 **Cloud SQL Proxy**를 알게 되었고, 결론적으로 이를 적용해 테스트 코드 실행 시 Db-A에 접속하여 드디어! 테스트 코드 실행 후 빌드에 성공할 수 있었다! 😂

Cloud SQL Proxy를 적용하고도 처음에는 여전히 테스트 코드 실행에 실패했었는데, 그 이유와 함께 Cloud SQL Proxy를 이용해 해당 문제를 해결한 방법에 대해 아래에서 더 자세히 설명하도록 하겠다 💪
<br><br>


## Cloud SQL Proxy? 🤔
먼저 Cloud SQL Proxy에 대해 간단히 알아보고 들어가자!

Cloud SQL Proxy는 승인된 네트워크나 SSL 구성 없이도 인스턴스에 안전한 엑세스를 제공하는 Cloud SQL 커넥터이다.

Cloud SQL 인증 프록시를 사용하면 다음과 같은 이점이 있다.
- SSL 인증서 없이 연결이 가능하다.
- IAM 권한을 사용해 Cloud SQL 인스턴스에 연결할 수 있는 사용자와 대상을 제어하기 때문에, Cloud SQL에 허용할 IP를 특별히 설정해주지 않아도 된다.
- OAuth 2.0 액세스 토큰의 자동 새로고침을 지원하기 때문에, 필요에 따라 해당 기능을 사용할 수 있다.<br>_(자세한 내용은 [여기](https://cloud.google.com/sql/docs/mysql/authentication?hl=ko) 참고)_


## Cloud Build에서 비공개 DB에 접근하기
자, 이제 Cloud Build에서 비공개 DB인 Db-A에 접근해 테스트 코드 실행에 성공할 수 있었던 방법에 대해 소개해보겠다! 😉

앞서 언급했던 것처럼, DB에 Cloud Build가 실행되는 IP 주소나 IP 대역을 허용하도록 해보았지만, 규칙없이 무작위로 IP가 할당되어 실행되기 때문에 특정한 IP나 IP 대역을 설정해줄 수 없었다.

Cloud SQL Proxy를 사용하면 DB에 허용할 IP를 설정해주지 않아도 간편하게 접속이 가능하다고 했다. 이를 이용해 다음과 같이 설정하면 된다.
1. Cloud Build의 gradle test step 전에 Cloud SQL Proxy를 실행한다.
2. 테스트 코드가 실행될 때 해당 프록시를 통해 DB에 접속하도록 한다.

### cloudbuild.yml 수정
Cloud Build 트리거 구성을 수정해보자. 
> 여기에서는 이미 사용하던 Cloud Build 트리거 설정이 있다고 가정한다. 기존에 사용하고 있던 트리거가 없다면, 새로 트리거를 생성할 때 설정해주면 된다!

원래 Cloud Build 트리거는 다음과 같은 순서로 실행하도록 설정되어 있었다.
1. Secret Manager로부터 인증 키 등의 설정 값을 가져와 파일로 만들기
2. 테스트 코드 실행 & jib을 이용하여 컨테이너 이미지 만들기 _(Gradle)_
4. 3에서 만든 이미지를 통해 GKE에 배포

테스트 코드가 실행되는 step 전에, Cloud SQL Proxy를 설치하고 실행하는 step을 아래와 같이 추가해준다.
~~~ yaml
  (생략)
  - name: gcr.io/cloud-builders/wget
    args:
      - 'https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64'
      - '-O'
      - cloud_sql_proxy
  - name: gcr.io/cloud-builders/gcloud
    args:
      - '-c'
      - >
        chmod +x cloud_sql_proxy

        ./cloud_sql_proxy
        -instances=[INSTANCE_CONNECTION_NAME]=tcp:0.0.0.0:13306
        -credential_file=[CREDENTIAL_FILE] &
    entrypoint: bash
  (생략)
~~~
- Cloud SQL Proxy는 13306번 포트로 실행한다.
- Cloud SQL Proxy를 실행할 때, 가장 처음 step에서 만들어둔 인증 키(CREDENTIAL_FILE)를 credential_file로 설정하여 실행한다.

### jdbc datasource 설정 변경
테스트 코드가 실행될 때는 test 프로필로 실행되는데, 이 때 읽어오는 JDBC Datasource의 url을 수정해주어야 한다. 앞서 Cloud SQL Proxy를 13306번 포트로 실행하도록 했으니, JDBC Datasource url의 host와 port를 아래와 같이 수정한다.
- host : 127.0.0.1 _(또는 0.0.0.0, localhost)_
- port : 13306

### Cloud Build 다시 실행해보기
여기까지 설정을 마치고나서, 다시 Cloud Build로 돌아가 수정한 트리거를 실행해보면 무사히 성공!!...할 것 같지만 안타깝게도 여전히 DB 접속에 실패한다 😭

> 혹시 Cloud SQL Proxy가 제대로 실행되지 않아서 해당 포트로 접근하지 못해 DB에 접근하지 못하는걸까? 라는 생각에 13306 포트가 열려있는지 확인해보고자 했다.

Cloud Build에서 Cloud SQL Proxy를 실행한 후에 해당 포트가 열려있는지 확인하기 위해서, 아래와 같이 cloudbuild.yml에 추가하여 확인해보았다.
~~~ yaml
(생략)
  - name: gcr.io/cloud-builders/gcloud
    args:
      - '-c'
      - >
        chmod +x cloud_sql_proxy

        ./cloud_sql_proxy
        -instances=[INSTANCE_CONNECTION_NAME]=tcp:0.0.0.0:13306
        -credential_file=[CREDENTIAL_FILE] &
				
        apt-get update && apt-get install -y netstat				
        nc -vz localhost 13306 && echo "Port is open!" || echo "Port is closed!"
    entrypoint: bash
(생략)
~~~

Cloud Build 트리거를 다시 실행하여 확인해보았을 때, **13306번 포트는 잘 열려있었지만 여전히 DB 접속은 실패**했다! 😠 <br>
![image](/assets/img/post/gcp/230329_cloud-sql-proxy로-비공개-db-접속하기/screenshot_01.png){: width='400'}
![image](/assets/img/post/gcp/230329_cloud-sql-proxy로-비공개-db-접속하기/screenshot_02.png){: width='650'}

### Cloud SQL Proxy 잘 실행되고 있는데 DB 접속이 왜 안돼...?
아니, 포트는 잘 열려있는데 왜! 계속 DB 접속에 실패하는걸까? 😢 

**바로 Cloud Build가 ⭐️도커⭐️로 실행된다는 점에 주목해야 한다!**
> Cloud Build가 도커로 실행된다는 것이 갑자기 떠올랐고, Chat GPT에게 Cloud Build가 어떻게 동작하는지 답변을 받아, 포트는 정상적으로 열려있지만 DB 접속이 되지 않는 원인을 알 수 있었다!
![image](/assets/img/post/gcp/230329_cloud-sql-proxy로-비공개-db-접속하기/screenshot_03.jpeg){: width='600'}

<br>

자, 그래서 정리하자면 **Cloud Bulid의 각 단계(Step)는 독립된 도커 컨테이너로 실행**된다는 거고, 각 step마다 서로 다른 빌드 환경을 사용하여 작업할 수 있다는거다!

그래서 Cloud SQL Proxy를 실행하고 다음 step으로 넘어가면, Cloud SQL Proxy가 실행되지 않는 _(13306포트가 열려있지 않은)_ 독립된 도커 컨테이너에서 gradle test가 실행되기 때문에 해당 포트로 DB 접근이 되지 않던 것이다! 😂

### cloudbuild.yml 최종 수정 및 빌드 성공 확인! ✌️
그래서, 같은 step에서 Cloud SQL Proxy를 실행한 후 gradle test를 실행할 수 있도록 다음과 같이 cloudbuild.yml을 수정해주었다.
~~~ yaml
(생략)
  // (1)
  - name: gcr.io/cloud-builders/gcloud
    args:
      - '-c'
      - >
        chmod +x cloud_sql_proxy

        ./cloud_sql_proxy
        -instances=[INSTANCE_CONNECTION_NAME]=tcp:0.0.0.0:13306
        -credential_file=[CREDENTIAL_FILE] &

        ./gradlew clean && ./gradlew test && ./gradlew jacocoTestCoverageVerification
    entrypoint: bash
  // (2)
  - name: 'gradle:7.4-jdk11'
    entrypoint: gradle
    args:
      - jib
      - '-Djib.to.tags=$SHORT_SHA'
(생략)
~~~
- (1) : Cloud SQL Proxy 실행 및 gradle clean, test, jacocoTestCoverageVerification을 한 step에서 실행
- (2) : (1)이 끝난 후 jib을 이용해 컨테이너 이미지 만들기

이렇게 수정 후 Cloud Build 트리거를 다시 실행하여 빌드를 실행하면, 테스트 코드가 정상적으로 실행되고 빌드에 성공한다! 😮‍💨

## 다른 GCP 프로젝트의 DB에 접근하기
빌드까지 성공했으니, 이제 배포 실패에 대한 문제를 해결해보도록 하자! 🥲

맨 처음에 언급했다시피 Api-A에서 Db-B로 접근하도록 설정해야 하니, 앞서 Cloud Build에서 사용했던 Cloud SQL Proxy를 이 경우에서도 적용해 해결해보려고 한다.

Cloud SQL Proxy는 다음과 같이 적용하여 사용할 것이다.
1. Proj-A에 있는 한 Compute Engine 인스턴스에 Cloud SQL Proxy를 13306 포트로 실행한다.
2. 1에서 실행한 Cloud SQL Proxy를 통해 DB에 접근한다.

### Cloud SQL Proxy 설치 및 실행
Proj-A에 띄워놓은 Compute Engine 인스턴스 중에서, 적절한 것을 하나 골라 Cloud SQL Proxy를 실행시켜줄 것이다. 
> _여기에서는 기존에 사용중인 Compute Engine 인스턴스가 있고, 이를 통해 Cloud SQL Proxy를 실행하여 DB에 접속하는 방법을 사용하는 방법을 설명한다. 만약 사용중인 인스턴스가 없다면 새로 인스턴스를 생성하여 설정하면 된다!_

그 전에 먼저 Cloud SQL Proxy 실행에 필요한 인증 키를 발급받아야 한다.
1. **Proj-B** _(Db-b가 위치한 프로젝트)_ **> IAM > 서비스 계정** 메뉴에 접속한다.
2. Cloud SQL Proxy 실행에 사용할 용도의 서비스 계정을 하나 생성한다.
3. 2에서 생성한 서비스 계정의 인증 키를 발급한 후, 해당 키 내용을 복사해둔다.

이제 Engine-A에 접속하고, 발급하여 복사해두었던 키 값을 적절한 위치에 파일로 만들어둔다.
> 프록시를 실행시켜 둘 인스턴스를 Engine-A 라고 표현하겠다! 

그리고 나서 아래 명령어를 통해 Engine-A에 Cloud SQL Proxy를 설치하고, 13306 포트로 실행한다.
~~~ bash
$ curl -o cloud-sql-proxy https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.0.0/cloud-sql-proxy.linux.amd64

$ chmod +x cloud-sql-proxy

$ ./cloud-sql-proxy [INSTANCE_CONNECTION_NAME] --address=0.0.0.0 --port=13306 --credentials-file=[CREDENTIAL_FILE_PATH] &
~~~
- **INSTANCE_CONNECTION_NAME** : Db-b의 연결 이름 _(GCP 콘솔에서 확인 가능하다!)_
- **CREDENTIAL_FILE_PATH** : 서비스 계정의 인증 키

### Cloud SQL Proxy를 통한 DB 접속
자, 이제 Cloud SQL Proxy를 통해 Db-b에 접속해보자.

Cloud SQL Proxy가 정상적으로 실행되었고, 이를 통해 Db-b로 접근이 잘 되는지 로컬에서 먼저 확인해보자!

**[Engine-A의 외부 IP]:13306**로 접속했을 때 정상적으로 연결되는지를 확인해보면 된다. 나는 MySQL Workbench에서 테스트해보았고, 다음과 같이 연결에 성공했다! ✌️ <br>
![image](/assets/img/post/gcp/230329_cloud-sql-proxy로-비공개-db-접속하기/screenshot_04.png){: width='300'}

### GKE 배포 확인
마지막으로 Cloud Build를 이용해 빌드 후 GKE에 배포하여 성공하는지 확인해보자!

Cloud Build에서 트리거를 다시 실행시키면, **GKE에 배포 성공!...이 한 번에 될 것 같지만 이번에도 DB 접속에 실패하고, 서버가 뜨지 않아 GKE의 Pod가 실행되지 못해 배포에 실패한다** 😂

**이번엔 또 왜! DB에 접근이 안되는걸까!? 🤔** 원인 파악을 위해서는 두 가지를 확인해보면 된다.
1. Engine-A 인스턴스에서 Db-B로의 접속이 정상적으로 가능한지
2. Api-A에서 Engine-A 인스턴스로의 접근이 가능한지

1의 경우, 앞서 확인해본 결과 **Engine-A의 Cloud SQL Proxy를 통해 Db-B로 접속이 가능함을 확인**했으므로 문제의 원인은 아니다. 그렇다면 나머지 한 가지가 원인일 확률이 매우 높다!

Api-A에서 Engine-A로 접근하려면, **Engine-A에 설정된 방화벽에 Api-A(GKE)의 IP 대역 및 13306 포트가 허용**되어 있어야 한다. Engine-A에 설정된 방화벽을 확인해보니 역시나 해당 IP 대역과 13306 포트가 허용되어 있지 않았고, 다음 설정을 방화벽에 추가해주었다.
- Api-A가 배포될 GKE 클러스터의 Pod 주소 범위 및 서비스 범위 IP 대역 추가
    > GCP 콘솔 > Kubernetes Engine > 클러스터 > 클러스터 이름 선택, 네트워킹 부분의 **Pod 주소 범위 및 서비스 범위**에 명시되어 있는 IP 대역 값을 복사하여 추가한다!

![image](/assets/img/post/gcp/230329_cloud-sql-proxy로-비공개-db-접속하기/screenshot_05.png){: width='350'}
- 13306 포트 허용<br>
![image](/assets/img/post/gcp/230329_cloud-sql-proxy로-비공개-db-접속하기/screenshot_06.png){: width='110'}


방화벽 설정 후 다시 Cloud Build로 돌아가 트리거를 실행시키면, 빌드 완료 후에 GKE 클러스터에 배포가 성공적으로 완료된 것을 확인할 수 있다! 🥳🥳 <br>
![image](/assets/img/post/gcp/230329_cloud-sql-proxy로-비공개-db-접속하기/screenshot_07.png){: width='300'}