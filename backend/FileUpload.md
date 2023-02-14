# :construction_worker: 회원 프로필 이미지

사이트에서 회원 정보의 기본이 되는 프로필 이미지 업로드를 처리했다.

이전까지는 프로젝트의 static 영역에 저장하는 식으로 구성했는데, 

이번 프로젝트에서는 서버를 연결했으므로 ec2 저장소 안에 저장하기로 했다!

프로젝트에서는 회원 가입할 때는 이미지를 따로 받지 않고, 회원 정보에 들어가 수정을 해야만 이미지를 설정할 수 있다.

<br>

## :bug: MultipartFile

스프링에서 제공하는 것으로 파일 업로드에 사용한다.

컨트롤러에서 요청을 받을 때 해당 부분을 MultipartFile로 받아주면 된다.

프론트에서는 multipart/form-data로 보내주면 된다.


여기서 고민한 부분은 @RequestPart, @RequestParam 중 어떤 것을 쓸지이다.

요청에서 file과 함께 사용자의 아이디, 닉네임을 함께 보내준다.

<br>

[RequestPart vs RequestParam](https://somuchthings.tistory.com/160?category=983431)

위 링크를 참고해서 고민하게 되었는데,

결론적으로 file은 @RequestParam, 아이디와 닉네임을 묶은 Dto는 @RequestPart로 받아주었다.

Dto의 경우는 json으로 받아서 매핑이 되어야 하므로 @RequestPart가 적절하였고,

이름만 잘 매핑하여 넘어오면 되는 file은 @RequestParam이 적절하였다.

![image](https://user-images.githubusercontent.com/81286029/218883863-bed55b1e-9958-4ab7-8b80-afb47351a446.png)

또한 이미지나 파일 둘 중 하나만 수정되는 경우도 있기 때문에 file의 경우 `required=false` 로 설정하였다.


<br>

## :art: 파일 업로드

이제 파일을 받았으니 서버에 올려야하고 회원의 프로필 이미지가 잘 적용되도록 적절한 url 을 프론트로 넘겨줘야 한다.

![image](https://user-images.githubusercontent.com/81286029/218884256-eb942e37-4bba-46f3-812c-8a18b53828b8.png)

서버에 해당 회원의 이미지를 넣을 때에는 `고유아이디_파일명` 과 같이 저장하였다.

그래서 새로운 이미지를 저장하기 전 해당 아이디로 저장된 파일이 있는지 디렉토리에서 먼저 찾는다.

만약 있다면 `delete()`를 사용하여 삭제한다.

그 다음 받은 이미지이름과 회원 아이디와 저장할 위치를 이용하여 저장 url을 작성한다.

file1.transferTo(file2); 와 같이 실행하면 받은 file1을 file2로 전환하는 작업을 거치면서 해당 위치의 이름으로 저장된다.




