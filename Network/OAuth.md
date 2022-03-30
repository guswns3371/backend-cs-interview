# OAuth

## OAuth

`OAuth(Open Authorization)` 이란 접근 위임을 위한 개방형 표준이다.

- 인터넷 사용자들이 비밀번호를 제공하지 않고 사용중인 서비스에 제3 서비스에 대한 사용자 정보에 접근할 수 있는 권한을 부여하는 수단이다.

> 인증(Authentication), 인가(Authorization)
> 
- `인증` : 사용자가 누구인지 확인하는 절차(아이디와 패스워드를 통한 인증)
- `인가` : 사용자에게 권한을 부여하는 것(OAuth)

> OAuth는 인증 프로토콜이 아니다.
> 

OAuth 프로토콜을 통해 얻은 AccessToken은 사용자를 확인하는 용도 즉, 인증의 용도로 사용할 수 없다. 대신 OAuth를 활용한 인증 표준에는 `OpenID Connect` 방식이 있다

- `OAuth` → 인가(Authorization) → `Access Token`
- `OpenID Connect` → 인증(Authentication) → `ID Token`

## OAuth2.0

OAuth1.0은 절차가 복잡하고 앱에서 사용하기 곤란하다는 단점이 있다. 결국 1.0의 단점을 개선한 OAuth2.0이 등장했다.

> OAuth 1.0 & 2.0
> 

| 비교 | OAuth1.0 | OAuth2.0 |
| --- | --- | --- |
| 역할 (역할 명칭 변경 및 세분화) | 이용자(User)<br/>소비자(Consumer)<br/>서비스 제공자(Service Provider) | 자원소유자(Resource Owner)<br/>클라이언트(Client)<br/>자원 서버(Resource Server)<br/>권한 서버(Authorization Server) |
| 토큰 | 요청 토큰 (Request Token)<br/>접근 토큰 (Access Token) | 접근 토큰 (Access Token)<br/>재발급 토큰 (Refresh Token) |
| API 호출 인증 및 보안 | 서명 | HTTPS(SSL/TLS) 기본<br/>서명 : 자원 서버가 별도 서명을 요구하는 경우 |
| 유효기간 | Access Token의 유효기간 없음 | Access Token 유효기간 부여<br/>만료 시 Refresh Token 이용 |
| 클라이언트 | 웹 | 웹, 앱 등 |

> OAuth 2.0 용어
> 

| 용어 | 설명 |
| --- | --- |
| Resource Server | OAuth2.0 서비스를 제공하고 자원을 관리하는 서버(구글,카카오, 네이버 등등) |
| Authorization Server | Client가 Resource Server의 서비스를 사용할 수 있게 토큰을 발급해주는 서버<br/>(구글, 카카오, 네이버 등) |
| Client | Resource Server의 API를 사용하여 데이터를 가져오려고 하는 사이트(현재 이용중인 서비스의 백엔드 서버) |
| Resource Owner | Resource Server의 계정을 소유하고 있는 사용자<br/>(서비스를 이용하는 사용자) |
| Access Token | Resource Server에 자원을 요청할 수 있는 토큰 |
| Refresh Token | Authorization Server에 Access Token 재발급 받기 위해 사용되는 토큰 |

> OAuth2.0 로직
> 

`AccessToken, RefreshToken 발급`

![0_PEQup7rNn8Ti00ky.png](OAuth%203ae5b/0_PEQup7rNn8Ti00ky.png)

1. OAuth 로그인을 위해 Authorization Sever에 자신의 서비스를 등록한다
2. Authorization Server는 해당 서비스에 Client Id, Client Secret을 부여하고, 서비스 개발자는 Redirect URL과 Scope를 설정한다
    - Client Secret : Client Id에 대한 비밀키(노출 절대안됨)
    - Redirect URL : Authorization Code를 전달받을 URL
    - Scope : Auth Server로부터 인가받을 권한의 범위
3. 사용자가 “연동 로그인 버튼”을 누르면 아래와 같은 주소(Auth Server가 제공하는 로그인 페이지)로 접속
    
    ```java
    https://kauth.kakao.com/oauth/authorize?
    client_id=test&
    response_type=code&
    scope=profile_nickname,account_email&
    redirect_uri=https://api/doctalk/callback
    ```
    
4. Auth Server가 제공하는 로그인 페이지에 Resource Owner가 로그인 정보를 입력
5. 사용자의 로그인 정보 + 클라이언트의 Client Id 정보를 전달받은 Auth Server는 Redirect URL이 서비스 개발자가 등록한 URL과 동일한지 확인 (다르거나 없다면 Error Page로 redirect됨)
6. Auth Server가 제공하는 Scope 권한 동의 페이지로 이동 후 동의
7. 사용자 인증이 완료되면 Auth Server는 Resource Owner에게 `Authorization Code` 를 response url에 담아 보낸다
8. 사용자가 Auth Server로부터 받은 response 속 URL로 redirect되면 = 백엔드서버(클라이언트)는 client id, client secret, authorization code, redirect url를 POST request에 담아 다시 Auth Server로 보낸다
9. Auth Server는 받은 정보가 유효하다면 백엔드서버(클라이언트)에게 AccessToken, RefreshToken을 발급해준다

`AccessToken 만료 시`

![0_KOqKaoLdAO7Iolw8.png](OAuth%203ae5b/0_KOqKaoLdAO7Iolw8.png)

1. 백엔드서버(클라이언트)가 만료된 AccessToken으로 Resource Server에 접근하면 401 에러가 발생한다.
2. 백엔드서버(클라이언트)는 보관중인 RefreshToken을 Auth Server로 전달한다
3. Auth Server는 RefreshToken의 유효성을 판단하고 AccessToken을 재발급해준다

## OAuth 2.0 인가 방식 종류

1. `Authorization Code Grant` : 권한부여 승인을 위해 Authorization Server가 클라이언트(백엔드 서버)에게 Authorization Code를 전달하는 인가 방식이다.
    - `Authorization Server가 Authorization Code를 전달하는 이유`
        - 사용자에게 Access Token을 노출시키지 않으면서 백엔드 서버로 AccessToken을 발급해주기 위함이다.
2. `Implicit Grant` : SPA(브라우저 기반의 앱)에 적합한 방식으로 Authorization Code없이 바로 AccessToken을 발급하는 인가 방식이다.
    - AccessToken이 URL로 전달된다는 단점이 있다.
    - RefreshToken 사용이 불가능한 방식이다
3. `Resource Owner Password Credentials Grant` : username, password로 AccessToken을 발급받는 간단한 인가 방식이다.
    - 사용자(Resource Owner)의 username, password가 클라이언트(백엔드서버)에 노출되기 때문에 클라이언트(백엔드서버)와 OAuth Provider(Resource Server)가 같은 도메인 내에 존재하여 신뢰할 수 있는 경우에 사용한다.
4. `Client Credentials Grant` : 클라이언트(백엔드서버)가 오직 Client Credential 정보만으로 AccessToken을 발급 받는 인가 방식이다.
    - 사용자(Resource Owner)에게 권한을 위임받는 목적이 아니라 클라이언트(백엔드서버)가 제3 서비스 자원에 접근하기 위한 방식이다.

## OpenID Connect(OIDC)

![Screen Shot 2020-12-14 at 10.20.24 PM.png](OAuth%203ae5b/Screen_Shot_2020-12-14_at_10.20.24_PM.png)

`OpenID Connect(OIDC)` 는 OAuth2.0 프로토콜의 인증(Authentication)을 담당하는 계층이다.

- OAuth의 인증서버로 이뤄지는 인증 시스템을 기반으로 클라이언트(백엔드서버)가 사용자(Resource Owner)에 대한 인증을 제공한다.
- 공식적인 OIDC Provider : Google, Line (Naver, Kakao, Facebook은 아니다)

> OpenID Connect의 flow
> 

![1_vo7DtARltqnys11DW8vboA.png](OAuth%203ae5b/1_vo7DtARltqnys11DW8vboA.png)

scope에 `openid` 를 설정하고, Authorization Server로부터 `AccessToken`과 함께 `ID Token`을 받는것 말고는 OAuth2.0 flow와 동일하다.

> ID Token과 AccessToken 차이점 ([ID Token vs AccessToken](https://auth0.com/blog/id-token-access-token-what-is-the-difference/))
> 

![Untitled](OAuth%203ae5b/Untitled.png)

![Untitled](OAuth%203ae5b/Untitled%201.png)

![Untitled](OAuth%203ae5b/Untitled%202.png)

<aside>
📎 ID Token은 사용자 인증용 토큰이고, AccessToken은 구글, 네이버의 자원에 접근하기 위해 사용되는 토큰이다. (인증 → ID Token, 인가 → AccessToken)

서버는 ID Token을 HTTPS를 사용하여 브라우저나 앱과 같은 클라이언트에게 전달하고, 클라이언트는 request의 Authorization 헤더에 ID Token을 담아 서버로 요청을 보낸다.

ID Token 유효성 검사를 한 뒤, 만약 만료되었다면 Refresh Token을 OpenId Provider에 보내어 ID Token을 재발급 받는다.

</aside>