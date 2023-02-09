# :children_crossing: CORS 

백엔드를 맡고 있어서 기능을 구현할 때까지는 swagger로 테스트를 했기 때문에 CORS를 설정하는 것에 대해 생각하지 못헀었다.

이후 프론트와 연결을 하면서 axios 통신을 하게 될 때 CORS 문제가 발생하는 상황들이 생겨서 해결했던 방법들을 정리하고자 한다.

<br>

## :seedling: CORS란


Cross Origin Resource Sharing

서로 다른 오리진 사이 자원을 공유할 수 있도록 허용할지를 결정하는 정책이다.

여기서 오리진(origin)이라는 것은 URL에서 Protocol, Host, Port 부분을 의미한다.

따라서 위 세 가지 중에서 하나라도 다르면 다른 오리진으로 판단하여 두 오리진 사이에 자원 공유를 허용해주어야 한다.

오리진을 판단하는 로직은 브라우저에서 한다.


### 그렇다면 CORS가 왜 필요할까?

원래는 SOP로 같은 오리진에서만 자원을 공유할 수 있도록 허용했다. XSS와 CSRF를 막기 위해서이다.

하지만 요즘같은 상황에서는 트래픽이 증가하면서 여러 서버가 필요해졌고 서로 다른 오리진에서 데이터를 공유할 일이 생겨났기 때문에 특정 상황에서 허용하는 것이 필요하다.

<br>

## :dizzy: 발생했던 CORS 문제

1. 처음에는 진짜 단순히 CORS 관련 설정을 안해주었기 때문에 발생한 문제이다..

   백엔드 팀에서 처음에 WebCofiguration을 설정해줬어야 했다. 앞으로 이런 부분들을 미리 생각해서 나중에 문제가 발생했을 때야 해결하지 않도록 해야겠다.
   
```
 @Configuration
 public class WebConfiguration implements WebMvcConfigurer {

   @Override
   public void addCorsMappings(CorsRegistry registry) {
     registry.addMapping("/**")
          .allowedOrigins("*")
          .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH")
          .maxAge(3000);
    }
}
```
이렇게 WebMvpConfigurer을 구현한 WebConfiguration을 작성해주었다.

일단 모든 경로와 메소드에 대해서 허용해주는 것으로 설정하였다.


2. 두 번째는 JWT 헤더를 붙이면서 발생한 에러이다.
   
   구글링하여 알아낸 결과는 preflight을 보낼 때는 OPTIONS라는 HTTP METHOD로 보내는데 해당 헤더에 대한 설정을 안해줬기 때문이었다.
   
   또한 preflight는 JWT 헤더를 붙여서 가지 않기 때문에 interceptor에서 OPTIONS 로 온 요청은 interceptor에서 무조건 true를 반환하도록 하였다.
   
   
3. 세 번째는 PATCH 로 보내지는 요청만 CORS 에러가 발생했다.

   분명 allowedMethod에 PATCH를 달아주었는데 왜 안되는지 답답했는데 이것 역시 구글링으로 해결할 수 있었다.
   
   CORS에러가 났던 프론트 팀원들의 노트북에 모두 Chrome CORS Extension을 사용하고 있어서였다.
   
   해당 Extension을 사용하면 자동으로 생성하는 헤더에 PATCH만 빠져있었다.
   
   따라서 Chrome Extension을 꺼두기만 하면 해결이 됐다.
   
   
4. 네 번째는 `registry.addAllowedOrigin("*")` 문제이다.
   
   해당 에러는 allowOriginPatterns를 사용하라고 친절하게 오류에서 나와있기 때문에 바꿔주면 된다.
   
   나의 경우는 addAllowedOrigin에 필요한 오리진만 적어두었다.
   


