# :sparkles: JWT

프로젝트에서 인증과 인가를 위한 JWT를 사용한다.

JWT 토큰에 대해서 알아보고 JWT 토큰 발급, 인터셉터 설정 등에 대해서 어떻게 구현했는지 서술하고자 한다.

## :green_heart: JWT란

JSON Web Token

사용자의 정보나 서비스 사용 정보를 담아 토큰을 생성한다

REST API로 개발하지 않을 때는 백엔드에서 세션을 사용하여 식별했지만,
REST API를 통하면서 세션을 사용하지 못하게 되었고 HTTP 요청을 보낼 때 헤더에 토큰을 넣어 단순하게 데이터를 받을 수 있게 되었다.

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
