# OAuth 정리
#스터디/oauth2

## OAuth란?
`OAuth`는 외부서비스에서 리소스를 사용하려할때, 계정과 암호의 노출없이 권한을 인증할 수 있는 표준입니다.(Open Standard Protocol)

`OAuth`는 2006년 트위터 개발자들과 Gnolia 개발자들의 논의를 시작으로 탄생했습니다.

`OAuth`가 탄생하기 전 회사들은 각각 인증방식을 구현했습니다.
이렇게 파편화된 인증방식들을 통합한 표준이라고 할 수 있습니다.

`OAuth 1.0a`와 `OAuth 2.0` 두 가지가 존재하고 최근에는 `OAuth 2.0`을 더 많이 사용합니다.

``` 
## 과거에 사용하던 인증방식 ##
- 구글의 AuthSub
- AOL의 OpenAuth
- 야후의 BBAuth
- 아마존의 웹서비스API
등등
```
```
## 참고 ##
OpenID와 OAuth는 인증이라는 면에서는 같습니다.
하지만, OpenID의 목적은 로그인 OAuth의 목적은 권한허가라는 점에서 차이점이 있습니다.
```

## OAuth 2.0의 Roles
`OAuth 2.0`의 설명에 앞서 몇가지 Role을 알아야합니다.
`OAuth 2.0`에는 총 4가지의 Role이 있습니다.

- `resource owner` - (리소스 소유자)
- 리소스 접근을 허가해주는 엔티티 (주로 사용자, 사람)
- `resource server` - (리소스 서버)
- 리소스를 제공해주는 API서버 (Like Facebook graph api)
- `client` - (클라이언트)
- `resource owner`의 승인을 얻어 `resource server`에 리소스 요청을 수행하는 응용프로그램
- 어플리케이션 구성의 client와는 상관없음
- `authorization server` - (인증 서버)
- 인증과 권한 취득에 성공했을때 토큰을 발급해주는 서버

## OAuth 2.0 Protocol Flow
`OAuth 2.0`의 Protocol은 3-legged 방식을 표현하고 있습니다.
자주 사용되지는 않지만 2-legged 방식도 `OAuth 2.0`에 포함됩니다.

![OAuth2 Protocol flow](protocol-flow.png)

(A). 클라이언트는 리소스 소유자에게 권한을 요청한다.
(B). 리소스 소유자는 클라이언트에 권한을 부여한다.
(C). 클라이언트는 부여된 권한을 인증 서버에 전달한다.
(D). 인증 서버는 클라이언트에 액세스토큰을 전달한다.
(E). 클라이언트는 리소스 서버에 액세스토큰을 전달하고 리소스를 요청한다.
(F). 리소스 서버는 클라이언트에 리소스를 전달한다.

## OAuth 2.0 Protocol Endpoint
`OAuth 2.0`은 인증 과정중에 총 3개의 Endpoint를 사용합니다.
인증 서버에 2개와 클라이언트에서 1개를 사용합니다.

-> 인증 서버
- `Authorization Endpoint`
- `Token Endpoint`

-> 클라이언트
- `Redirection Endpoint`

#### Authorization Endpoint
사용자의 신원을 확인하고 권한을 허가받기 위해 사용한다.
사용자의 신원을 확인하는 방법은 별도로 명시되어 있지않다. (주로 세션사용)

#### Token Endpoint
`Authorization Grant` 혹은 `Refresh Token`을 토큰으로 교환할때 사용한다.
`Token Endpoint` 클라이언트 인증을 진행해야한다.

#### Redirection Endpoint
`Authorization Endpoint`의 결과를 클라이언트로 전송할때 사용한다.
`Authorization Endpoint`로 넘긴 값 혹은 미리 등록해둔 값을 통해 Redirect한다.

## OAuth 2.0 Client
- `OAuth 2.0`는 클라이언트를 미리 등록해야 사용할 수 있습니다.
- 클라이언트 등록 방법에 대한 것은 별도로 명시되있지 않다.
- 클라이언트와 인증 서버간의 직접적인 상호 작용은 필요하지 않다.

클라이언트 등록시 아래 사항들을 만족하면 된다.
- 클라이언트 타입을 지정한다.
- Redirect URI를 지정한다.
- 인증서버가 요구하는 기타정보를 입력한다. (이름, web사이트, 설명, 이미지 등 Like Facebook Developer)

#### Client Types
`OAuth 2.0`에서는 두가지 타입의 클라이언트를 정의하고 있습니다.
- Confidential
- 인증의 기밀성을 유지할 수 있는 클라이언트 (Web Server Application)
- Public
- 인증의 기밀성을 유지할 수 없는 클라이언트 (Native App Application, Browser)

#### Client Identifier
- 클라이언트 등록시 인증 서버는 고유성을 가지는 식별자를 발행한다.
- 클라이언트 식별자는 외부에 노출되기 때문에 클라이언트 인증시 클라이언트 식별자 만으로 인증을 진행하면 안된다.

#### Client Password
- 클라이언트 인증시 사용하는 인증수단
- 클라이언트 암호를 발급하는 방식과 클라이언트 시크릿을 발급하는 방식 두가지가 있다.

-> 암호 발급방식
- Http Basic 인증 방식을 지원해야한다.
```
Authorization: Basic czZCaGRSa3F0Mzo3RmpmcDBaQnIxS3REUmJuZlZkbUl3
```

-> 클라이언트 시크릿을 발급하는 방식
- Http Basic 방식 대신 `client_id`와 `client_secret`을 request body에 포함한다.
```
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA
&client_id=s6BhdRkqt3&client_secret=7Fjfp0ZBr1KtDRbnfVdmIw
```

#### Client Authentication
- 기밀 클라이언트는 인증 서버에서 요구하는 클라이언트 인증을 설정한다.
- 기밀 클라이언트는 일반적으로 암호(Client Password) 혹은 Public/Private키쌍을 통해 클라이언트 인증을 진행합니다.

## OAuth 2.0의 권한 획득 방법
`OAuth 2.0`에서 권한 획득 방법을 `Authorization Grant`라고 명명하고 있습니다.
`Authorization Grant`는 굉장히 유연한 방식으로 위 프로토콜만 구현하면 (때로 일부분이 생략되더라도) 권한 획득 방법중 한가지로 사용할 수 있습니다.

[RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749) 문서에서는 4가지의 `Authorization Grant`를 정의하고 있습니다.

- `Authorization Code`
- `Implicit`
- `Resource Owner Password Credentials`
- `Client Credentials`

#### Authorization Code
- `Authorization Code`는 리소스 소유자와 클라이언트 사이의 중개자로서 인증서버를 통해 발급 받는다.
- 인증서버는 권한 코드를 발급하기 전, 리소스 소유자의 인증을 진행하고 권한을 얻는다.
- 리소스 소유자는 인증 서버를 통해서만 권한 허가를 진행하기 때문에 클라이언트와 권한정보를 공유하지 않는다.
- 인증 서버에서 사용자 에이전트를 통하지 않고 바로 클라이언트로 액세스 토큰을 전달할 수 있는 등 여러가지 면에서 보안적인 혜택이 존재한다.
- 기밀 클라이언트의 최적화되어있다.

![OAuth2 Authorization Code](authorization-code.png)

#### Implicit
- `Authorization Code`의 흐름을 단순화 한 것이며, 브라우저에서 실행되는 클라이언트에 최적화 되어있다.
- 클라이언트에서 인증코드 대신 액세스 토큰을 받는다.
- 클라이언트의 특성상 액세스 토큰이 다른 응용 프로그램에 노출될 가능성이 있다.
- `Authorization Code`과 비교했을때 보안성과 효율성(요청횟수가 적음)의 트레이드 오프관계이다. 

![OAuth2 Implicit](implicit.png)

#### Resource Owner Password Credentials
- 리소스 소유자와 클라이언트가 신뢰할 수 있을때만 사용할 수 있다.
- 타 인증수단을 사용할 수 없을때만 사용할것을 권장한다.
- 토큰 엔드포인트로 요청할때 `ID/Password`를 파라미터로 담아서 요청한다.

![OAuth2 Resource Owner Password Credentials](resource-owner-password-credentials.png)

#### Client Credentials
- 권한 범위가 클라이언트의 제어하에 있는 리소스로 제한되거나 인증서버와 함께 배치된 리소스로 제한된 경우에만 사용할 수 있다.
- 기밀 클라이언트인 경우에만 사용가능하다.

![OAuth2 Client Credentials](client-credentials.png)


## Refresh Token
- `OAuth 2.0`에서는 액세스 토큰 만료시 새로운 액세스 토큰을 발급할 수 있는 수단을 제공합니다.
- `OAuth 2.0`은 액세스 토큰의 유효주기를 짧게 유지합니다. (권장)
- 인증서버에서 `Refresh Token`을 제공하는 경우만 사용할 수 있다.
- `Refresh Token`의 제공은 필수가 아니다.
- Refresh Token도 하나의 `Authorization Grant`처럼 구현하여 처리한다.

![OAuth2 Refresh Token](refresh-token.png)

```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA
```

---
## 코멘트
- 챕터 넘어가기전에 질문의 시간이 있으면 좋겠다.
- 포맷이 줄글이다보니 지루했다. 
  - 부가설명할 애들이 있으면 좋을듯
- 발표자와 듣는 사람들과 상호작용이 있었으면 좋았을것 같다.
