# :page_facing_up: 페이지네이션

데이터를 리스트로 가져오게 되면 반드시 페이지네이션 처리가 필요하다.

이번 프로젝트에서도 목록을 가져오는 경우가 많았다.

특히 가장 중요한 것은 대화할 방 목록을 가져올 때이다.

핵심 기능인만큼 이 부분에서 페이지 처리를 고민하면서 `무한스크롤`로 구현하기로 했다.

무한스크롤을 선택한 이유는 대화 목록은 상태에 따라 계속 바뀌고 생기게 되고 방이 많고 초대기능이 있다보니 페이지로 방을 이동해서 찾는게 의미 없다고 판단했기 때문이다.

<br>

사실 백엔드에서는 무한스크롤이나 페이지보기나 처리를 똑같이 해주면된다.

하지만 실제로 프론트에서 무한스크롤 관련해서 이슈가 많아서 구현에 성공하진 못했다.(라이브러리를 쓰면 쉽게 된다고 하는데 우리는 복잡한 필터가 많고 데이터가 많아서 사용하기 좋지 않다고 한다..)

아래에서는 백엔드에서 Pageable을 사용한 방법에 대해 정리하고자 한다.

<br>

## :triangular_flag_on_post: Pageable

Spring Data Jpa 에서는 Pagination을 처리하기 편리한 Pageable을 제공해준다.

Pageable은 Pagination을 처리하기 위한 정보를 넘기는데, page, size(offset), sort 와 같은 것이 있다.


페이지 처리가 필요한 api를 호출할 때 프론트에서는 `/growth/exp/list/{userId}?page={page}&size={size}&baseTime={baseTime}`
이런식으로 page, size와 같은 내용을 query string으로 넘겨주게되면

<span style='background-color:#fff5b1'>컨트롤러</span>
에서는 

```    
@GetMapping("/exp/list/{userId}")
public ResponseEntity<?> listExp(@PathVariable int userId,
                                   @PageableDefault(sort = "expLogSq", direction = Sort.Direction.DESC) Pageable pageable,
                                     @RequestParam String baseTime) {
 ```
 
 각각의 변수를 따로 받을 필요 없이 Pageable를 사용하면 알아서 매칭이 된다.
 
 <br>
 <br>
 
 이제 해당 목록을 가져오는데 페이지 처리를 하는 메소드를 `Repository`에 작성한다.
 
 ```
Slice<ExpLog> findByUserIdAndRegTmLessThanEqual(@Param("userId") int userId, @Param("regTm"), LocalDateTime regTm, Pageable pageable);
```

위에서 말했듯이 우리는 **무한스크롤 처리**를 하고자 했으므로 `Page`가 아닌 `Slice`를 사용했다.

`Slice`는 `Page`와는 다르게 전체 데이터 수를 가져오지 않고, 다음에 데이터가 있는지 알려주는 `hasNext()`가 존재한다.

사실 `Page`를 써도 크게 상관없지만 성능 상에 차이가 있다.

`Page`의 경우는 조회하는 쿼리를 실행한 후 전체 데이터 수를 조회하는 쿼리를 다시 한 번 실행한다.

`Slice`는 limit(size)+1 된 값을 가져오기 때문에 데이터 양이 많고 총 데이터 개수가 필요없는 경우에 유리하다.


<br>
<br>

Repository에서 가져온 데이터를 이제 `Service`에서 `Dto로 변환`해주어 반환해야 한다.

```
Slice<ExpLog> slice = expLogRepository.findByUserIdAndRegTmLessThanEqual(userId, localDateTime, pageable);

List<ExpLogResponseDto> users = new ArrayList<>();
for(ExpLog expLog : slice.getContent())
    users.add(ExpLogResponseDto.fromEntity(expLog));

return PageableResponseDto.builder().list(users).hasNext(slice.hasNext()).build();
```
<br>

`PageableResponseDto`는 프로젝트에서 데이터 목록을 조회하는 경우가 많아서 Response를 통일하기 위해 만든 Dto이다.

```
@Getter
@Setter
@Builder
@AllArgsConstructor
@ToString
@ApiModel(value="페이징 응답 정보")
public class PageableResponseDto {

    private boolean hasNext;

    private List<?> list;

}
```

클래스 내부를 보면 `List`와 boolean 타입의 `hasNext`를 필드로 갖는다.

다시 위의 코드를 살펴보면 `PageableResponseDto`의 `list`에는 아까 Slice로 받아와서 처리한 list를 넣어주고,

`hasNext`에는 Slice가 가지고 있는 hasNext 값으로 세팅해주었다. 


<br>

Swagger를 통해 테스트를 해보자!

![image](https://user-images.githubusercontent.com/81286029/219510901-f617b7f7-4c6d-4134-ae93-576951011e65.png)

이런식으로 page, size를 포함해서 넘겨주게 되면

![image](https://user-images.githubusercontent.com/81286029/219510975-61a88603-d80b-4b11-b820-2b5a1c8c13de.png)

다음과 같이 `list`와 `hasNext`값이 잘 넘어온다.
