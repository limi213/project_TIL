# :sparkles: JWT

프로젝트에서 인증과 인가를 위한 JWT를 사용한다.

JWT 토큰에 대해서 알아보고 JWT 토큰 발급, 인터셉터 설정 등에 대해서 어떻게 구현했는지 서술하고자 한다.

## :green_heart: JWT란

JSON Web Token

사용자의 정보나 서비스 사용 정보를 담아 토큰을 생성한다

REST API로 개발하지 않을 때는 백엔드에서 세션을 사용하여 식별했지만,
REST API를 통하면서 세션을 사용하지 못하게 되었고 HTTP 요청을 보낼 때 헤더에 토큰을 넣어 단순하게 데이터를 받을 수 있게 되었다.

<br> 

JWT는 Header, Payload, Signature로 구성되어 있다.

- Header - JWT에서 사용할 타입, 해시 알고리즘 종류

- Payload - 서버에서 첨부한 사용자 권한 정보와 데이터

- Signature - Header에 있는 해시 함수를 적용하여 개인키로 서명한 전자서명


<br>

### 순서

사용자가 로그인 요청을 하면

1. 백에서 사용자에 정보를 바탕으로 access token refresh token을 발급하여 클라이언트에게 넘긴다.

2. 클라이언트는 받은 토큰들을 잘 가지고 있고 모든 HTTP 요청 헤더에 access token을 넣어서 보낸다.

3. 백에서는 요청을 받을 때마다 헤더에 있는 access token이 유효한지를 먼저 검증하고 유효하다면 요청을 처리해서 클라이언트에 보내준다.

4. 만약 유효하지 않다면 401 UnAuthorized를 반환하여 클라이언트가 access token을 재발급하는 API를 보내도록 유도한다.

5. access token을 재발급하는 API를 호출할 때는 refresh token을 보내 refresh token이 유효한지를 파악한다.

6. refresh token이 유효하다면 access token을 재발급하고,

7. 유효하지 않다면 다시 401 UnAuthorized를 반환하여 클라이언트가 사용자를 로그아웃 처리할 수 있도록 유도한다.

<br>

## :rocket: Bearer 

JWT나 OAuth으로부터 얻은 access token을 사용하는 것이다 ( RFC 6750 )

HTTP Authorization request header 는 `Authorization : <type> <credentials> `형식으로 되어있는데,

이때 type 부분에 `Bearer`을 쓰면 된다.

<br>

## :pushpin: Jwt Intercpetor 

**Interceptor**는 컨트롤러에 들어오는 요청과 나가는 응답을 가로챈다.

Filter와 역할이 비슷하나 다른데 가장 큰 차이는 Filter는 DispatcherServlet이 실행되기 전에, 
Interceptor는 DispatcherServlet이 실행된 후에 호출된다는 것이다.

<br>

프로젝트에서는 발급한 accessToken을 인증하는 것을 컨트롤러에 들어오기 전인 Interceptor를 설정하여 확인하는 것으로 구성했다.


JWT token을 발급하는 JwtService를 구현한 후, HandlerInterceptor를 구현한 **JwtInterceptor**를 작성한다.

여기에서는 preHandle, postHandle 등의 메소드를 재정의할 수 있다.

요청을 가로채서 확인해야 하므로 preHandle만 작성했다.

```
<JwtInterceptor>
request에서 url를 확인하여 swagger 관련이라면 인증 토큰 확인 전에 true를 반환한다.

프로젝트에서 특정 url의 get 요청만 jwt token 확인이 필요가 없었기 때문에 uri와 method 모두 확인하여 제외했다.

나머지 요청들에 대해서는 헤더의 Authorization 에서 Bearer을 추출하여 accessToken을 꺼내어 인증 절차를 확인 후 true/false를 반환한다.
```

<br>

만든 Interceptor를 WebConfiguration에 등록하기 위해 addInterceptor 메소드를 재정의한다.

```
<WebConfiguration> 

addInterceptor로 작성해 둔 JwtIntercpetor를 등록하고

excludePathPatterns()를 이용해서 Jwt 인증 과정에서 제외해야 하는 uri를 설정했다.
```

<br>

## :alien: Swagger 설정


프로젝트에서 쉬운 테스트를 위해서 Swagger3을 사용했다. 

이제 인증이 추가되면서 Swagger에 등록되어 있는 인증이 필요한 모든 기능들을 테스트할 수 없게 된다...

따라서 Swagger에 헤더를 달아주는 작업을 해야 한다.

직접 참고한 글은 아니지만 더 자세한 내용인 것 같아서 참고문헌을 남긴다

[Swagger Token 설정](https://blog.jiniworld.me/113)


설정을 완료하면 Swagger를 사용할 때 로그인 과정을 통해 토큰을 발급받고 

Authorized 내 빈칸에 작성하면 다른 기능들을 사용할 수 있다.







