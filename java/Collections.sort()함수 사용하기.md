# Collections.sort()함수 사용하기

데이터베이스로 정렬하지 않고 후에 처리를 해야하는경우가있다.

배열을 정렬할 땐 Arrays.sort(), 컬렉션을 정렬할 땐 Collections.sort()를 사용한다. 
난 여기서 Collections.sort()를 알아보겠다.

나는 JPA를 사용하면서 평균률을 쿼리로 실행하지 않고 결과값을 가공하여 넣었다. 
근데 보여지는 화면에서 테이블안에 오름차순,내림차순의 기능이 평균률에 있었다. 
때문에 받은 결과데이터를 가지고 정렬을 생각하다가 collections.sort()함수를 알게되었다. 

먼저 내가 쿼리를 통해 받은 List데이터에 아래의 코드를 추가해준다. 

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class memberResponse implements Comparable<memberResponse>{
	private Long id;
	
	private String name;
	
	private String sortType; //asc인지 desc인지 체크하기
	
	@Override
    public int compareTo(@NotNull memberResponse o) {
        
    }
}
```

`implements Comparable<memberResponse>` 인터페이스 Comparable <>을 상속받는다. 
상속받으면 부모의 메소드인 `compareTo`라는 메소드를 오버라이딩 하게된다.

그럼 거기에 아래의 코드를 넣어주면 끝이다. 
id 부분에 내가 정렬할 대상을 넣어주면된다. 

```
if (StringUtils.isNotEmpty(this.sortType)) {
            if (this.id > o.id) {
                    return 1;
                } else if (this.id < o.id) {
                    return -1;
                } else {
                    return 0;
                }
        }
```

여기까지 하면 셋팅은 끝이났다. 
`MemberService.java` 로 가서 

```
public Page<memberResponse> search(MemberRequest request, Pageable pageable){

        log.debug("회원 검색조건에 맞춰 조회");
        List<memberResponse> list = memberRepository.findBySearchList(request);

        if (request.sortType().equals("asc")) {
        	log.debug("회원 아이디 오름차순 조회");
            Collections.sort(list);
        } else {
            log.debug("회원 아이디 내림차순 조회");
            Collections.sort(list, Collections.reverseOrder());
        }
       
        return new PageImpl<>(list,pageable,list.size());
}
```

나는 여기서 페이징처리를 해야하기때문에 return 값을 Page로 감싸서 처리했다. 
일단 내림차순은 `reverseOrder()`를 추가해주면된다.

이렇게 코드를 완성하면 정렬이 진행된다. 

추가적으로 정렬을 할때 LocalDateTime의 형태를 정렬해야할때가 있다. 
그럴때는 코드가 조금 바뀐다.  Service에서는 동일하지만 
`MemberResponse`에서 코드를 바꿔줘야한다.

```
@Override
public int compareTo(@NotNull memberResponse o) {
       if (this.sortDateType.equals("asc")) {
          return o.startDate.compareTo(this.startDate); //오름차순
      } else {
          return this.startDate.compareTo(o.startDate); //내림차순
	  } 
}
```

compartTo()함수를 사용하여 날짜를 비교해준다. 
오름차순과 내림차순일때 비교대상이 바뀌는걸 주의하자!

자바스크립트에서도 정렬을 사용할 수 있는데 그 코드는 나중에 올리겠다!