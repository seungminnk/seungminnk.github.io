---
title: Keycloak 소셜로그인 설정하기
date: 2022-12-15 15:08:00 +0900
categories: [Keycloak]
tags: [keycloak, authentication, oauth]
---

이전 포스트에서는 관리자 콘솔에서 직접 사용자를 생성하고, 패스워드를 설정해서 액세스 토큰을 발급받았다.<br>
_(username과 password로 로그인하는 방식 이용)_

하지만 우리 서비스에는 아이디(username) 및 패스워드를 이용한 방식이 아닌 **소셜 로그인** 기능이 필요하다. 

Keycloak에서는 OIDC를 지원하는 다양한 소셜 로그인을 지원하고 있다. 기본적으로 Keycloak에서 제공하는 소셜 로그인 방식은 다음과 같다.<br>
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_01.png){: width='600'}

대부분 한국 서비스는 카카오와 네이버 로그인을 기본적으로 제공하는데, 지원하는 소셜 로그인 방식 중에서는 찾아볼 수 없다. 그래서 직접 Identity Provider를 이용하여 필요한 소셜 로그인 방식을 구현해주어야 한다.

이번 포스트에서는 카카오와 네이버, 애플 로그인 방식을 추가하는 방법에 대해 설명하고자 한다.

카카오는 22년 3월부터 OIDC 방식을 지원하고 있기 때문에, OpenID Connect v1.0 를 통해 간단히 추가할 수 있다.<br>
하지만 네이버와 애플는 아직 OAuth 방식만을 지원하고 있기 때문에, 직접 Identity Provider를 구현하여 jar 파일로 만들고, Keycloak 서버를 실행할 때 추가해주어야 한다.<br>
> [참고] 애플 로그인용 jar 파일은 오픈소스로 공개되어 있는 것이 있음!


## 카카오 로그인
먼저 간단하게 추가할 수 있는 카카오 로그인 설정부터 진행해보자.

관리자 콘솔에 접속한 후, 좌측 하단의 Identity Provider - User defined - OpenID Connect v1.0 을 클릭한다.
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_02.png){: width='600'}

Alias에는 kakao, Display name _(로그인 화면에서 보여질 이름)_ 에는 Kakao를 입력하고, Discovery endpoint에는 아래의 카카오 메타 정보를 입력해준다.
~~~ text
https://kauth.kakao.com/.well-known/openid-configuration
~~~
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_03.png){: width='600'}


그리고나서 카카오 개발자센터에 생성해둔 애플리케이션 상세페이지로 접속한다. _(카카오 개발자센터에 애플리케이션은 이미 생성해두었다고 가정)_

좌측의 **카카오 로그인** 메뉴를 클릭하고, OpenID Connect 활성화 설정을 **ON**으로 변경한다.
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_04.png){: width='600'}
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_05.png){: width='400'}

그리고 좌측 메뉴의 요약 정보 페이지에 접속한 후, 앱 키의 **REST API 키**를 복사한다.
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_06.png){: width='600'}

Keycloak 관리자 콘솔로 돌아와 Client ID 부분에 복사한 REST API 키를 붙여넣는다.
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_07.png){: width='650'}
> ⚠️&nbsp;&nbsp;**참고**<br>
_Client Secret은 카카오 로그인에서 필수 값이 아니기 때문에 복사한 키 값을 똑같이 넣어주어도 되고, 카카오 개발자센터 - 카카오 로그인 - 보안 에 접속하여 Client Secret 값을 새로 생성해 설정해주어도 된다._
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_08.png){: width='550'}

<br>

Add를 눌러 카카오 Identity Provider를 생성하고, Redirect URI를 복사해둔다.
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_09.png){: width='600'}

다시 카카오 개발자센터에 접속해서 카카오 로그인 페이지로 이동한 후, 하단의 Redirect URI 부분에 복사한 값을 붙여넣는다.<br>
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_10.png){: width='600'}
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_11.png){: width='500'}

드디어 카카오 로그인 설정이 완료되었다! 카카오 로그인이 정상적으로 동작하는지 확인해보자.

아래 주소로 접속 후 로그인 화면으로 이동하면 다음과 같이 카카오 로그인 버튼이 잘 생성되어 있는 것을 확인할 수 있다.
`[KEYCLOAK_SERVER_HOST]/realms/[REALM_NAME]/account`<br>
(ex. <http://localhost:8080/realms/myrealm/account>)
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_12.png){: width='600'}
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_13.png){: width='450'}

생성된 Kakao 버튼을 클릭해 카카오 로그인을 완료하면 다음과 같이 추가 정보를 입력하는 페이지가 나타나고 _(회원가입 절차)_, 입력 후 submit을 누르면 로그인이 정상적으로 완료된다.
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_14.png){: width='450'}
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_15.png){: width='600'}

> _개발자도구의 네트워크 탭에서 확인했을 때, 아래와 같이 액세스 토큰도 응답 값으로 잘 리턴되는 것을 확인할 수 있다!_
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_16.png){: width='650'}

관리자 콘솔에서도 방금 로그인(가입)한 사용자를 확인할 수 있다.
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_17.png){: width='550'}
<br>


## 네이버 로그인
### 네이버 Identity Provider jar 만들기
네이버는 카카오와는 달리 OAuth만을 지원하기 때문에, 직접 Identity Provider를 구현해서 Keycloak에 직접 추가해주어야 한다.

네이버 로그인을 위한 Identity Provider를 만들어보자.

jar 파일 생성을 위해 프로젝트를 생성해주어야 한다. 여기에서는 Gradle 프로젝트를 생성해주었다. 프로젝트 전체 구조는 다음과 같다.<br>
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_18.png){: width='400'}

먼저 필요한 Gradle Dependency를 추가해주어야 한다. build.gradle 파일의 dependencies 부분에 다음 내용을 추가해준다.
~~~ gradle
compileOnly 'org.keycloak:keycloak-services:20.0.1'
compileOnly 'org.keycloak:keycloak-server-spi:20.0.1'
compileOnly 'org.keycloak:keycloak-server-spi-private:20.0.1'
~~~

그 다음 **AbstractOAuth2IdentityProvider** 클래스를 상속하고, **SocialIdentityProvider** 인터페이스를 구현하는 **NaverIdentityProvider** 클래스를 생성한다.
~~~ java
import com.fasterxml.jackson.databind.JsonNode;
import org.keycloak.broker.oidc.AbstractOAuth2IdentityProvider;
import org.keycloak.broker.oidc.OAuth2IdentityProviderConfig;
import org.keycloak.broker.oidc.mappers.AbstractJsonUserAttributeMapper;
import org.keycloak.broker.provider.BrokeredIdentityContext;
import org.keycloak.broker.provider.IdentityBrokerException;
import org.keycloak.broker.provider.util.SimpleHttp;
import org.keycloak.broker.social.SocialIdentityProvider;
import org.keycloak.events.EventBuilder;
import org.keycloak.models.KeycloakSession;

public class NaverIdentityProvider extends AbstractOAuth2IdentityProvider implements SocialIdentityProvider {

  // 네이버 개발가이드(API 문서)에 정의되어 있는 URL 참고
  public static final String AUTH_URL = "https://nid.naver.com/oauth2.0/authorize";
  public static final String TOKEN_URL = "https://nid.naver.com/oauth2.0/token";
  public static final String PROFILE_URL = "https://openapi.naver.com/v1/nid/me";

  // 네이버 로그인 시에는 scope 값이 필요 없어 빈 문자열로 정의함	
  public static final String DEFAULT_SCOPE = "";

  public NaverIdentityProvider(KeycloakSession session, OAuth2IdentityProviderConfig config) {
    super(session, config);
    config.setAuthorizationUrl(AUTH_URL);
    config.setTokenUrl(TOKEN_URL);
    config.setUserInfoUrl(PROFILE_URL);
  }

  @Override
  protected boolean supportsExternalExchange() {
    return true;
  }

  // 네이버 profile endpoint 주소 반환
  @Override
  protected String getProfileEndpointForValidation(EventBuilder event) {
    return PROFILE_URL;
  }

  // 네이버에 인증 요청 후 토큰을 받아오는 역할 (토큰을 이용해 profile을 가져오는 역할 수행)
  @Override
  protected BrokeredIdentityContext doGetFederatedIdentity(String accessToken) {
    try {
      JsonNode profile = SimpleHttp.doGet(getConfig().getUserInfoUrl(), session)
          .header("Authorization", "Bearer " + accessToken).asJson();

      return extractIdentityFromProfile(null, profile);
    } catch (Exception e) {
      throw new IdentityBrokerException("Could not obtain user profile from naver.", e);
    }
  }

  // 네이버 profile 내용 반환
  @Override
  protected BrokeredIdentityContext extractIdentityFromProfile(EventBuilder event, JsonNode profile) {
    BrokeredIdentityContext user = new BrokeredIdentityContext(profile.get("response").get("id").asText());

    user.setUsername(profile.get("response").get("name").asText());
    user.setEmail(profile.get("response").get("email").asText());

    user.setIdpConfig(getConfig());
    user.setIdp(this);

    AbstractJsonUserAttributeMapper.storeUserProfileForMapper(user, profile, getConfig().getAlias());

    return user;
  }

  @Override
  protected String getDefaultScopes() {
    return DEFAULT_SCOPE;
  }
}
~~~

다음은 **NaverIdentityProviderFactory** 클래스이다.
~~~ java
import org.keycloak.broker.oidc.OAuth2IdentityProviderConfig;
import org.keycloak.broker.provider.AbstractIdentityProviderFactory;
import org.keycloak.broker.social.SocialIdentityProviderFactory;
import org.keycloak.models.IdentityProviderModel;
import org.keycloak.models.KeycloakSession;

public class NaverIdentityProviderFactory extends AbstractIdentityProviderFactory<NaverIdentityProvider>
    implements SocialIdentityProviderFactory<NaverIdentityProvider> {

  public static final String PROVIDER_ID = "naver";

  @Override
  public String getName() {
    return "Naver";
  }

  @Override
  public NaverIdentityProvider create(KeycloakSession session, IdentityProviderModel model) {
    return new NaverIdentityProvider(session, new OAuth2IdentityProviderConfig(model));
  }

  @Override
  public IdentityProviderModel createConfig() {
    return new OAuth2IdentityProviderConfig();
  }

  @Override
  public String getId() {
    return PROVIDER_ID;
  }
}
~~~

마지막으로 **NaverUserAttributeMapper** 클래스를 생성한다.
~~~ java
import org.keycloak.broker.oidc.mappers.AbstractJsonUserAttributeMapper;

public class NaverUserAttributeMapper extends AbstractJsonUserAttributeMapper {

  private static final String[] cp = new String[] { NaverIdentityProviderFactory.PROVIDER_ID };

  @Override
  public String[] getCompatibleProviders() {
    return cp;
  }

  @Override
  public String getId() {
    return "naver-user-attribute-mapper";
  }
}
~~~


### Keycloak에 Naver IdP 추가하기
우리가 추가할 Naver Identity Provider(IdP)를 Keycloak 서버가 인식할 수 있도록 설정해주어야 한다.

resources 디렉토리에 `META-INF/services` 디렉토리를 생성하고, 다음 두 파일을 생성해준다.<br>
각각의 파일에는 앞서 생성한 **NaverUserAttributeMapper와 NaverIdentityProviderFactory 클래스의 위치**를 작성한다.
- org.keycloak.broker.provider.IdentityProviderMapper
: ~~~ text
org.keycloak.social.naver.NaverUserAttributeMapper
~~~
- org.keycloak.broker.social.SocialIdentityProviderFactory
: ~~~ text
org.keycloak.social.naver.NaverIdentityProviderFactory
~~~

그리고 관리자 콘솔에서 Naver Identity Provider를 추가할 때 나타날 화면을 설정해줄 수 있다. 
> 예를 들면, Google IdP _(아래 스크린샷 참고)_ 처럼 기본적으로 설정해야 하는 Client Id나 Client Secret 값 외의 특정 데이터를 더 입력받아야 하는 경우를 들 수 있다.<br>
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_19.png){: width='400'}

`main/resources` 디렉토리에 `theme-resources.resources.partials` 디렉토리를 생성하고, 해당 디렉토리 내에 아래 html 파일을 추가해준다.
- realm-identity-provider-*.html
: _기본값(Redirect URI, Client ID, Client Secret, Display order) 입력하는 form을 정의_
- realm-identity-provider-*-ext.html
: _부가(추가) 정보를 입력하는 form 정의 ~~(정확하진 않음)~~_

네이버 로그인 시에는 일단 기본 값만 입력하면 되므로 realm-identify-provider-naver-ext.html 에는 아무 내용을 작성하지 않은 채로 생성하고, realm-identify-provider-naver.html 파일만 아래와 같이 작성하여 추가한다.
~~~ html
<div data-ng-include data-src="resourceUrl + '/partials/realm-identity-provider-social.html'"></div>
~~~
> 기본으로 정의되어 있는 html파일(템플릿)을 include하는 코드이다. 만약 직접 정의하고 싶다면, html로 작성해주면 된다. 
_( `참고` [Discord Identity Provider](<https://github.com/wadahiro/keycloak-discord>) )_

필요한 파일과 클래스를 모두 생성했으니, 이제 jar 파일로 빌드해보자.

터미널에 `gradle build` 명령어를 실행하여 프로젝트를 빌드한다. 빌드가 성공적으로 완료되면, `build/libs` 디렉토리에 빌드된 jar 파일이 생성된다.<br>
![image](/assets/img/post/keycloak/221216_소셜로그인-설정하기/screenshot_20.png){: width='400'}

이렇게 생성된 jar 파일은 Keycloak을 띄울 서버의 `/opt/keycloak/providers` 디렉토리에 추가해주어야 한다.

먼저, 앞서 Keycloak Docker 이미지를 빌드할 때 사용했던 Dockerfile이 위치한 폴더에 해당 jar 파일을 복사해둔다. 그리고 Dockerfile을 열어 다음과 같이 내용을 수정한 후 저장한다.
~~~ dockerfile
# (1)
FROM quay.io/keycloak/keycloak:latest as builder

# (2)
COPY ./keycloak-naver-1.0-SNAPSHOT.jar /opt/keycloak/providers

RUN /opt/keycloak/bin/kc.sh build

# (3)
FROM quay.io/keycloak/keycloak:latest

COPY --from=builder /opt/keycloak /opt/keycloak 

WORKDIR /opt/keycloak

ENV KC_DB=mysql
ENV KC_DB_URL=jdbc:mysql://host.docker.internal:3306/keycloak
ENV KC_DB_USERNAME=keycloak
ENV KC_DB_PASSWORD=keycloak
ENV KEYCLOAK_ADMIN=admin
ENV KEYCLOAK_ADMIN_PASSWORD=admin

ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
~~~
- **(1)** : build 명령을 실행하여 최적화된 이미지 생성을 위한 빌드 옵션 설정
: > 만약 사용자 지정 provider를 설치하려는 경우, `/opt/keycloak/providers` 디렉토리에 jar 파일을 복사해주어야 하는데, 이 부분에 정의해주면 된다!<br>
_(이후 단계인 소셜 로그인 설정(identity provider 추가)하는 부분에서 사용하게 될 예정!)_
- **(2)** : `/opt/keycloak/providers` 디렉토리에 생성한 naver identity provider jar 파일 복사
- **(3)** : (1)의 빌드 단계에서 생성된 파일이 새 이미지로 복사됨
<br>


## 애플 로그인
<br>


## 참고사이트
- <https://www.sad-waterdeer.com/keycloak/2022/08/05/Keycloak-%EC%B9%B4%EC%B9%B4%EC%98%A4%ED%86%A1-%EB%A1%9C%EA%B7%B8%EC%9D%B8.html>
- <https://subji.github.io/posts/2020/07/24/keycloak4>