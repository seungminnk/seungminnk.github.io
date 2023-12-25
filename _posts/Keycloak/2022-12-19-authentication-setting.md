---
title: 웹사이트에서 Keycloak 인증 설정하기
date: 2022-12-19 17:34:00 +0900
categories: [Keycloak]
tags: [keycloak, authentication]
---

이번 포스트에서는 React로 구성된 웹사이트를 생성하고, 이 웹사이트와 앞서 생성했던 API 서버와 통신할 때, 로그인 등의 인증 과정을 Keycloak이 담당할 수 있도록 연동하는 방법에 대해 설명하고자 한다.

플로우를 먼저 설명하면 다음과 같다.
1. React로 구성된 웹사이트에 접속한다.
2. 인증이 필요한 엔드포인트에 접근하려고 하면, 에러 발생 또는 로그인 페이지로 리다이렉트 된다.
3. 로그인한다.
4. 2에서 접근하려고 시도했던 엔드포인트에 다시 요청을 보낸다.
5. 정상적으로 데이터가 리턴된다.
6. 특정 권한 _(`ex` admin)_ 을 가지고 있어야 접근 가능한 엔드포인트에 요청을 보낸다.<br>
    1. 해당 권한이 없으면 에러가 리턴된다.<br>
    2. 해당 권한을 가지고 있는 사용자라면 정상적으로 데이터가 리턴된다.
<br><br>

    
## React 웹사이트 구성 및 Keycloak 연동
자, 그러면 먼저 React 웹사이트를 구성해보자. 

> React 프로젝트를 새로 생성하고 화면을 만들기엔 시간도 들고 어려우니까!😙<br>
Keycloak 연동용으로 구성된 테스트 프로젝트를 깃허브에서 클론받아 실행시키자!<br>


아래 깃허브 레포 링크로 접속해 프로젝트를 클론받는다. _(zip 파일을 받아 적절한 위치에 압축을 풀어 프로젝트를 열어도 되고, 직접 git clone으로 프로젝트를 클론받아도 괜찮다.)_<br>
> <https://github.com/Qodestackr/keycloak-app>

Visual Studio Code로 클론받은 프로젝트를 연다. _(다른 IDE 툴도 괜찮다!)_


프로젝트는 일단 두고, React 애플리케이션(웹사이트)를 연동할 Keycloak Client를 생성하자.

Keycloak 관리자 콘솔에 접속하여 앞서 생성했던 myrealm 내에 아래와 같이 mywebsite라는 Client를 새로 생성한다.<br>
_(생성 시 별다른 설정은 수정하지 말고, Client ID만 입력한 후 생성한다.)_<br>
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_01.png){: width='550'}

그리고 생성한 Client(mywebsite)의 **Root URL**과 **Valid redirect URIs**를 다음과 같이 설정한다.
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_02.png){: width='550'}

이제 다시 React 프로젝트로 돌아와, 루트 디렉토리의 keycloak.json 파일을 열어 다음과 같이 수정한다.
~~~ json
{
  "realm": "myrealm", // (1)
  "auth-server-url": "http://localhost:8080",  // (2)
  "ssl-required": "external",  // (3)
  "resource": "mywebsite",  // (4)
  "public-client": true,  // (5)
  "confidential-port": 0 // (6)
}
~~~
- (1) : Client가 속한 Realm 이름
- (2) : Keycloak 서버 URL
- (3) : Keycloak 서버와 **HTTPS**로 통신하기 위해 **external**로 설정<br>_(production에서는 all로 설정할 것을 권장)_
- (4) : React 프로젝트와 연결할 Keycloak Client ID
- (5) : true로 설정할 경우 Keycloak에 credential을 보내지 않음 _(기본값은 false)_
- (6) : SSL/TLS 보안 연결을 위해 Keycloak 서버에서 사용하는 비밀 포트 _(기본값은 8443)_


프로젝트 루트 디렉토리에서 터미널을 열고, 아래 명령어를 차례로 실행시킨다. 그러면 React 애플리케이션이 `localhost:3000` 으로 실행된다.
~~~ shell
yarn install
npm run dev
~~~
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_03.png){: width='550'}

이 데모 프로젝트를 잠시 설명하자면, 우측 상단의 Login 혹은 Resource 메뉴를 클릭했을 때 아래 `Resource.jsx`에 정의된 코드가 실행된다.
~~~ jsx
import { useState, useEffect } from 'react';
import Keycloak from 'keycloak-js';

export default function Resources(){
  const [keycloak, setKeycloak] = useState(null)
  const [authenticated, setAuthenticated] = useState(false)

  useEffect(()=>{
    const keycloak = Keycloak('/keycloak.json');
    console.log("keycloak", keycloak)
    keycloak.init({ onLoad: 'login-required' }).then(authenticated => {
      console.log("authenticated", authenticated)
      setKeycloak(keycloak)
      setAuthenticated(authenticated)
    })
  }, [])

  if (keycloak) {
    if (authenticated) return (
      <div className='my-12 grid place-items-center'>
        <p>This is a Keycloak-secured component of your application. You shouldn't be able
          to see this unless you've authenticated with Keycloak.</p>
          <div>
          <img src="https://random.imagecdn.app/500/250"/> 
          </div>
      </div>
    ); 
    else return (<div className='my-12'>Unable to authenticate!</div>)
  }

  return(
    <>
      <div className='my-12'>Initializing Keycloak...</div>
    </>
  )
}
~~~

- 페이지가 처음 로딩될 때, useEffect()가 실행된다. 이 메소드에서는 **keycloak.json 파일의 설정 값을 바탕으로 Keycloak 객체를 초기화**하고, 이 객체가 인증되었는지 상태를 가지고 있는다.
- 만약 keycloak 변수가 **인증이 완료된 상태**라면, 특정 이미지를 띄워주도록 html을 리턴한다.
- 하지만 **인증이 완료되지 않았다면**, `Initializing Keycloak...` 이라는 문구가 나타나고, Keycloak **로그인 페이지로 리다이렉트**된다.


실행시켜둔 React 애플리케이션으로 다시 돌아와서, Login 메뉴를 눌러 해당 페이지에 접속한다. 페이지에 `Initializing Keycloak...` 문구가 잠시 떴다가, Keycloak 로그인 페이지로 넘어간다.
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_04.png){: width='600'}
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_05.png){: width='400'}

여기서 로그인을 하고 나면 인증이 완료된 상태가 되므로, 위에서 설명한 것처럼 이미지가 나타난 페이지를 확인할 수 있어야 한다.<br>
하지만, 이미지는 뜨지 않고 계속 Initializing Keycloak... 상태에서 변하지 않는다😠

개발자도구를 열어 에러를 확인해보자. 콘솔을 확인해보면 **CORS** 에러가 발생한 것을 알 수 있다.
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_06.png){: width='600'}

이 CORS 에러를 해결하기 위해, 다시 Keycloak 관리자 콘솔에 접속한다. mywebsite Client 설정으로 이동하고, Web origins 부분에 React 애플리케이션의 URL을 입력한다.
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_07.png){: width='550'}

이제 다시 웹사이트로 돌아와 해당 페이지에서 새로고침을 해주면, 다음과 같이 이미지가 나타난 페이지를 확인할 수 있다.
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_08.png){: width='550'}
<br><br>


## Role 별로 제한된 엔드포인트에 요청 날리기
앞서 만들어두었던 네 가지 엔드포인트를 기억에서 되살려보자.
- `/permit-all` 
: **인증 없이** 모두 허용<br>_(헤더에 액세스 토큰이 없거나 만료된 토큰이 들어있어도 접근 허용)_
- `/authenticated` 
: **인증된 사용자**만 접근 허용<br>_(헤더에 유효한 액세스 토큰이 들어있을 경우에만 접근 허용)_
- `/user-allowed` 
: **user role**을 가진 인증된 사용자만 접근 허용<br>_(헤더에 유효한 액세스 토큰이 들어있고, 해당 사용자가 user role을 가진 경우에만 접근 허용)_
- `/admin-allowed` 
: **admin role**을 가진 인증된 사용자만 접근 허용<br>_(헤더에 유효한 액세스 토큰이 들어있고, 해당 사용자가 admin role을 가진 경우에만 접근 허용)_


### React 프로젝트 코드 수정
웹사이트에서 위 네 가지 엔드포인트에 요청을 날리기 위해서는, React 프로젝트 코드 변경이 약간 필요하다😅

먼저 터미널을 열고, axios 패키지를 설치해준다.
~~~ shell
yarn add axios
~~~

그리고 `src/components` 디렉토리의 `Resource.jsx` 파일을 `RoleResource.jsx`로 같은 위치에 복사하여 생성한다.<br>
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_09.png){: width='200'}

`App.jsx` 파일을 열고, 새로 만든 `RoleResource.jsx`를 Route path로 추가한다.
~~~ jsx
import {BrowserRouter, Route, Routes} from 'react-router-dom';

// components
import NavBar from './components/NavBar';
import Footer from './components/Footer';
import Home from './components/Home';

import Resources from './components/Resources';
import RoleResources from './components/RoleResources';

export default function App() {
  return (
  <div>
    <BrowserRouter>
      <NavBar/>

      <Routes>
        <Route path='/' element={<Home/>}/>
        <Route path='/resource' element={<Resources/>}/>
        <Route path='/role-resource' element={<RoleResources/>}/>
      </Routes>

      <Footer/>
    </BrowserRouter>
  </div>
  )
}
~~~

다음으로 `src/components/NavBar.jsx` 파일을 열어, Resource 메뉴를 눌렀을 때 `RoleResources.jsx` 파일을 바라보도록 코드를 수정한다.
~~~ jsx
import {Link} from 'react-router-dom'

export default function NavBar() {
  return (
    <>
    <div className='flex justify-around items-center py-5 bg-[#234] text-white'>
      <h1 className='font-semibold font-2xl'>KeyCloak App</h1>
      <ul className='flex'>

        <li className='mx-1'>
          <Link to='/'>Home</Link>
        </li>
        <li className='mx-1'>
          <Link to='/resource'>Login</Link>
        </li>
        <li className='mx-1'>
          <Link to ='/role-resource'>Resource</Link>
        </li>
      </ul>
    </div>
    </>
  )
}
~~~

마지막으로, `RoleResources.jsx` 파일을 다음과 같이 수정한다. <br>
_(먼저 `/authenticated` 요청 & 헤더에 유효한 액세스 토큰이 있는 경우부터 테스트해보자!)_
~~~ jsx
import { useState, useEffect } from 'react';
import Keycloak from 'keycloak-js';
import axios from 'axios';  // (1)

export default function Resources(){
  const [keycloak, setKeycloak] = useState(null)
  const [authenticated, setAuthenticated] = useState(false)
  const [resultStr, setResultStr] = useState(null)

  useEffect(()=>{
    const keycloak = Keycloak('/keycloak.json');
    keycloak.init({ onLoad: 'login-required' })
    .then(authenticated => {
      setKeycloak(keycloak)
      setAuthenticated(authenticated)
    })
  }, [])

  if (keycloak) {
    if (authenticated) {
			// (2)
      axios.get("http://localhost:8081/authenticated", {
        headers: {
          'Authorization': 'Bearer ' + keycloak.token. // (3)
        }
      })
        .then(result => {  
					setResultStr(result.data);  // (4)
        })
        .catch(error => {
					// (5)
          if(error.response.status === 401) {
            setResultStr("인증이 만료되었습니다. 다시 로그인하세요!");
          } else if(error.response.status === 403) {
            setResultStr("접근 권한이 없습니다!");
          } else {
            setResultStr("네트워크 에러가 발생했어요!");
          }
        });
      
      return (
				// (6)
        <div className='my-12 grid place-items-center'>
          <p>{resultStr}</p>
        </div>
      ); 
    }
    else return (<div className='my-12'>Unable to authenticate!</div>)
  }

  return(
    <>
      <div className='my-12 grid place-items-center'>Initializing Keycloak...</div>
    </>
  )
}
~~~
- (1) : axios 라이브러리를 import 한다.
- (2) : 인증이 필요한 API에 request를 보낸다.
- (3) : keycloak 서버에서 리턴받은 access token을 넣어준다.
- (4) : 성공적으로 응답을 받은 경우, resultStr에 응답 받은 데이터를 넣는다.
- (5) : 토큰이 만료되었거나 _(401)_, 접근 권한이 없거나 _(403)_, 에러가 발생한 경우 resultStr에 에러 메시지를 넣는다.
- (6) : (4), (5)에서 넣어준 resultStr을 화면에 띄운다.


### 인증이 필요한 엔드포인트
`npm run dev` 명령어로 프로젝트를 실행시키고, 실행시킨 웹사이트로 접속한다.<br>
_(여기서는 <http://localhost:3000>)_

그리고 우측 상단의 Resource 메뉴를 누르면 `Intializing Keycloak...` 이 화면에 떴다가, Keycloak 로그인 창으로 이동한다.<br>
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_10.png){: width='550'}
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_11.png){: width='550'}
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_12.png){: width='350'}

해당 페이지에서는 `/authenticated` API를 호출하도록 설정해두었는데, 이 API는 인증된 사용자일 경우에 **_This is authenticated request!_** 라는 메시지를 리턴한다.

User Role을 가진 사용자 _(앞서 생성한 user1)_로 로그인한다. 인증된 사용자이므로, 페이지 중앙에 This is authenticated request! 라는 메시지가 나타난 것을 확인할 수 있다.<br>
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_13.png){: width='350'}
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_14.png){: width='550'}


### User Role이 필요한 엔드포인트
다음은 User Role을 가진 사용자만 접근을 허가하는 `/user-allowed` API를 호출해보자. 이 API는 User Role을 가진 사용자가 접근했을 때, **_This request for USER ROLE!_** 이라는 메시지를 리턴한다.

`RoleResource.jsx` 파일에 `/user-allowed` API를 호출하도록 코드를 수정하고, 웹페이지로 돌아와 새로고침하면 다음과 같이 메시지가 나타난다.<br>
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_15.png){: width='550'}


### Admin Role이 필요한 엔드포인트
마지막으로 Admin Role을 가진 사용자만 접근을 허가하는 `/admin-allowed` API를 호출해보자. 이 API는 Admin Role을 가진 사용자가 접근했을 때, **_This request for ADMIN ROLE!_** 이라는 메시지를 리턴한다.

마찬가지로 `RoleResource.jsx` 파일에 `/admin-allowed` API를 호출하도록 코드를 수정하고, 웹페이지로 돌아와 새로고침하면 다음과 같이 접근 권한이 없다는 메시지가 출력된 것을 확인할 수 있다.
![image](/assets/img/post/keycloak/221219_웹사이트에서-인증-설정하기/screenshot_16.png){: width='550'}

> This request for ADMIN ROLE! 메시지를 정상적으로 받기 위해서는, Admin Role을 가진 admin1 사용자로 로그인하여 해당 페이지에 다시 접근하면 된다.

<br>


## 참고사이트
- <https://medium.com/devops-dudes/secure-front-end-react-js-and-back-end-node-js-express-rest-api-with-keycloak-daf159f0a94e>
- <https://www.section.io/engineering-education/keycloak-react-app/>