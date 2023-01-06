---
title: Spring Boot와 Keycloak 연동하기
date: 2022-12-15 11:58:00 +0900
categories: [Keycloak]
tags: [keycloak, authentication]
---

이번 포스트에서는 API 서버의 인증을 Keycloak에서 담당하고, **인증된 사용자의 요청만 허용**하도록 설정하는 방법에 대해 알아보자!<br>

>사용자가 특정 요청을 보냈을 때, API 서버로 해당 요청이 들어가기 전 Keycloak에서 인증된 사용자인지 판단한다. Keycloak에서 인증된 사용자라고 판단했다면, 각각의 API에 접근하여 원하는 결과를 받을 수 있다.<br>

API 서버는 Spring Boot로 구성하고, Spring Security를 이용하여 Keycloak과 연동한다.


## Client 생성하기
먼저 API 서버와 연결할 Client를 Realm에 생성해보자.<br>
좌측 메뉴의 Clients - Create client 버튼을 클릭한다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_01.png' width='650'>

Client ID를 입력하고 Next를 눌러 다음 페이지로 넘어간다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_02.png' width='650'>

인증된 사용자의 요청만 API 서버로 들어가도록 할 것이기 때문에, Client에는 인증 설정이 되어 있어야 한다.

OIDC 방식으로 인증하여 액세스할 수 있도록 설정하기 위해 Client authenticateion을 On으로 설정하고, 세부 권한 제어 활성화를 위해 Authorization을 On으로 변경한 후 Save 버튼을 눌러 Client를 생성한다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_03.png' width='650'>
<br><br>


## Spring Boot 프로젝트 생성하기
다음은 API 서버로 사용할 Spring Boot 프로젝트를 생성해야 한다.<br>
Spring Boot 프로젝트는 Spring Initializr 사이트 _(<https://start.spring.io>)_ 에서 쉽게 생성할 수 있다.

프로젝트 메타데이터를 입력하고 우측 Dependencies에 lombok, Spring web, OAuth2 Client를 추가한 후 Generate를 눌러 프로젝트 zip 파일을 다운받는다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_04.png' width='650'>

원하는 폴더에 다운받은 zip 파일을 풀고, 프로젝트를 연다.
<br><br>


## Keycloak과 Spring Boot 연동하기
이제 Keycloak과 Spring Boot를 연동하는 과정이다.

### Gradle Dependency 추가
먼저, Keycloak 연동을 위한 dependency를 추가해야 한다. build.gradle 파일을 열어 dependencies 부분에 아래 내용을 추가해준다.<br>
_(Spring Boot Starter, Spring Security Adapter 디펜던시 추가)_
~~~ gradle
implementation 'org.keycloak:keycloak-spring-boot-starter:19.0.2'
implementation 'org.keycloak:keycloak-spring-security-adapter:19.0.2'
~~~
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_05.png' width='600'>
<br>

### Spring Boot Adapter 설정
그리고 application.properties 파일에 Keycloak 관련 Spring Boot Adapter 설정을 해주어야 한다.

앞서 생성한 Client의 Client secret 값을 알아야 한다. Keycloak 관리자 콘솔에 접속하여 좌측 메뉴의 Client 탭을 누르고, 생성한 Client의 이름을 클릭한다.<br>
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_06.png' width='600'>

상단의 Crendentials 탭을 누르고, Client secret 값을 복사한다.<br>
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_07.png' width='600'>

다시 Spring Boot 프로젝트로 돌아와 application.properties 파일을 열어 다음 내용을 작성한다.
~~~ properties
server.port=8081

keycloak.enabled=true
keycloak.realm=myrealm   # (1)
keycloak.auth-server-url=http://localhost:8080   # (2)
keycloak.ssl-required=external   # (3)
keycloak.resource=myapi   # (4)
keycloak.credentials.secret=[CLIENT_SECRET]   # (5)
keycloak.use-resource-role-mappings=true   # (6)
keycloak.bearer-only=true   # (7)
~~~
- (1) : Keycloak에 생성한 Realm 이름
- (2) : Keycloak 서버 URL
- (3) : Keycloak 서버와의 통신 방식을 HTTPS로 설정하기 위해 **external**로 설정 _(production에서는 **all**로 설정할 것을 권장)_
- (4) : Keycloak에 생성한 Client 이름
- (5) : 앞서 복사해두었던 Client Secret 값
- (6) : true인 경우 서비스 Level의 Role(Client Role)을 적용하여 내부 토큰 값을 확인하고, false인 경우 Realm Level의 Role을 적용 _(기본 값은 false)_
- (7) : true로 설정할 경우, 사용자 인증은 거치지 않고 bearer token만 검증 _(기본 값은 false)_
<br><br>


### Spring Security Adapter 설정
그 다음으로는 Spring Security Adapter 설정이다.

Spring Security를 사용하면서 보안 구성을 원하는대로 변경하여 적용하기를 원할 때, **WebSecurityConfigurerAdapter**를 상속받아 구현할 수 있다.<br>
Keycloak에서는 이를 생성하기 위한 간편한 기본 클래스인 **KeycloakWebSecurityConfigurerAdapter**를 제공하고 있다.

이 클래스를 구현하는 KeycloakSecurityConfig 클래스를 만들어 보안 구성 설정을 해보자.<br>
`/permit-all` 은 인증되지 않은 사용자도 접근 가능하게, 그 외 나머지는 인증된 사용자만 접근하도록 구성했다.
~~~ java
@KeycloakConfiguration
@Import({KeycloakSpringBootConfigResolver.class})
public class KeycloakSecurityConfig extends KeycloakWebSecurityConfigurerAdapter {

  @Autowired
  public void configureGlobal(AuthenticationManagerBuilder auth) {
    // authentication manager에 keycloakAuthenticationProvider 등록
    auth.authenticationProvider(keycloakAuthenticationProvider());
  }

  @Override
  protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
    // session authentication strategy 정의
    return new RegisterSessionAuthenticationStrategy(new SessionRegistryImpl());
  }

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    super.configure(http);

    http.authorizeRequests()
        .antMatchers("/permit-all").permitAll()
        .anyRequest().authenticated();
  }
}
~~~
- `@KeycloakConfiguration`
: Spring Security에서 Keycloak 연동에 필요한 모든 어노테이션을 정의하고 있다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_08.png' width='550'>
- `@Import({KeycloakSpringBootConfigResolver.class})`
: Keycloak Spring Security Adapter는 기본적으로 keycloak.json 파일을 설정 파일로 인식한다.<br>
KeycloakSpringBootConfigResolver 빈을 추가해주면, Spring Boot의 yml 파일 _(application.properties)_ 을 설정 파일로 인식할 수 있다.

### API 테스트하기
API 테스트를 위해 컨트롤러 클래스를 작성한다.
~~~ java
@RestController
public class TestController {

  @GetMapping("/permit-all")
  public ResponseEntity permitAll() {
    return new ResponseEntity<>("This is allowed request!", HttpStatus.OK);
  }

  @GetMapping("/authenticated")
  public ResponseEntity authenticated() {
    return new ResponseEntity<>("This is authenticated request!", HttpStatus.OK);
  }

}
~~~

이제 Spring Boot 프로젝트를 실행시키고, Postman으로 요청 테스트를 해본다.

먼저 인증 없이 접근 가능한 `/permit-all` 엔드포인트 테스트이다.<br>헤더에 별도 인증 토큰을 넣어주지 않아도 정상적으로 응답이 리턴된다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_09.png' width='650'>

다음은 인증된 사용자만 접근 가능한 `/authenticated` 엔드포인트 테스트이다.<br>헤더에 별도 인증 토큰을 넣지 않고 요청을 날리면, 다음과 같이 **401**(Unauthorized)이 리턴되는 것을 확인할 수 있다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_10.png' width='650'>

> 위 엔드포인트에 대해 정상 값을 리턴받기 위해서는, 인증된 사용자의 액세스 토큰을 헤더에 넣어 요청해야 한다.<br>액세스 토큰을 발급받아 위 엔드포인트에 다시 요청을 날려보자!

앞서 생성했던 사용자 _(user1)_ 의 계정으로 로그인하면 액세스 토큰을 발급받을 수 있다.

`[KeycloakServer]/realms/[RealmName]/protocol/openid-connect/token` 으로 다음 값을 body에 넣어 요청한다.
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_11.png' width='650'>
- **client_id** : Client 생성 시 설정한 이름 _(myapi)_
- **username** : 로그인할 User 이름
- **password** : 로그인할 User 계정에 설정한 패스워드
- **grant_type** : password로 고정 _(이 경우는 username과 password로 로그인하여 토큰을 발급받는 경우임)_
- **client_secret** : Client의 client secret 값 _(application.properties 파일에 설정했던 값)_

정상적으로 응답을 받으면, 다음과 같이 json 형식으로 데이터가 리턴된다.
~~~ json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJtaEw0SmVnR1ViTVF4aDFabmdrWG5yLUpUVHliMDR1aXZ1NDRmakRDQ0w4In0.eyJleHAiOjE2NjgwNjI0OTEsImlhdCI6MTY2ODA2MjE5MSwianRpIjoiNzNiMmNiZGEtYzNlNS00MTgyLWIwYTItY2M1ZWNjN2ZiOGJiIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDg5L3JlYWxtcy9teXJlYWxtIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjIxMjcxN2RkLWYzNzctNGZlMi1iZjY2LTk3YTg3ZWZlYzgxMSIsInR5cCI6IkJlYXJlciIsImF6cCI6Im15YXBpIiwic2Vzc2lvbl9zdGF0ZSI6IjJmMmI2OGJmLTYyZDYtNGM2Yi1hMTI5LWExZmRlYTRmZjQ1YyIsImFjciI6IjEiLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsiZGVmYXVsdC1yb2xlcy1teXJlYWxtIiwib2ZmbGluZV9hY2Nlc3MiLCJ1bWFfYXV0aG9yaXphdGlvbiJdfSwicmVzb3VyY2VfYWNjZXNzIjp7ImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsInNpZCI6IjJmMmI2OGJmLTYyZDYtNGM2Yi1hMTI5LWExZmRlYTRmZjQ1YyIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwicHJlZmVycmVkX3VzZXJuYW1lIjoidXNlcjEiLCJnaXZlbl9uYW1lIjoiIiwiZmFtaWx5X25hbWUiOiIifQ.ALMdiDSKrIlXd1iUEyNyLgXvwl7HFjwZVgWpE4irBD2mtiNXfAHSIz-HE-JUJNK958l65ZrjLfnbl0zFJ_ISvy50SQdzl3avCK38xJ2xETiGWGOv5YyCvOyslIonCtyv9qKcaPnrRI2SkE1UZuudZEih_d69COcPWBDxtMxJDLn0lctWJxbNGqVDVMDREKMzzGQgUJ1ZJcBapVMO0LAWsm-X3OZVc5PU7FqbAhonmP-R8iXIUi52eJAgWWH9V1AXharku1bwTYL-fqc_uc_PHHMtZbpajlktvLaiBFgFE9kQ4ALH1zwoihvatkex-7wn5whHU29CD5kc_-SPlrIqzw",
    "expires_in": 300,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIxZTFiMTRjNy1kN2YzLTRiMjQtYWYwMS02M2UxMTUwMTg1MmQifQ.eyJleHAiOjE2NjgwNjM5OTEsImlhdCI6MTY2ODA2MjE5MSwianRpIjoiNzI1ODBjOTAtNWViNC00OTIwLTlkODktNDU0Zjc4YTE2MGU4IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDg5L3JlYWxtcy9teXJlYWxtIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MDg5L3JlYWxtcy9teXJlYWxtIiwic3ViIjoiMjEyNzE3ZGQtZjM3Ny00ZmUyLWJmNjYtOTdhODdlZmVjODExIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6Im15YXBpIiwic2Vzc2lvbl9zdGF0ZSI6IjJmMmI2OGJmLTYyZDYtNGM2Yi1hMTI5LWExZmRlYTRmZjQ1YyIsInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsInNpZCI6IjJmMmI2OGJmLTYyZDYtNGM2Yi1hMTI5LWExZmRlYTRmZjQ1YyJ9.BmWNwtjeCB9oFKPp_HEDy8DT-2VMzGgjW-BcTSgP52o",
    "token_type": "Bearer",
    "not-before-policy": 0,
    "session_state": "2f2b68bf-62d6-4c6b-a129-a1fdea4ff45c",
    "scope": "profile email"
}
~~~

리턴받은 값 중 access_token 값을 복사하고, 인증이 필요했던 엔드포인트 _(`/authenticated`)_ 에 다시 요청을 날려보자.

다시 Postman으로 돌아와 Authorization 탭에서 Type을 **Bearer Token**으로 선택하고, 복사해두었던 토큰 값을 붙여넣는다. 그리고나서 요청을 날리면 다음과 같이 정상적으로 결과가 리턴되는 것을 확인할 수 있다!
<img src='/assets/img/post/keycloak/221215_spring-boot와-keycloak-연동하기/screenshot_12.png' width='650'>
<br><br>


## 참고사이트
- <https://www.keycloak.org/docs/latest/securing_apps/#_spring_boot_adapter>
- <https://www.keycloak.org/docs/latest/securing_apps/#_spring_boot_adapter>
- <https://www.keycloak.org/docs/latest/securing_apps/#_spring_security_adapter>
