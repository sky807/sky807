# QueryDSL 에서 정렬조건 동적으로 처리하기 

가끔 하다보면 정렬조건을 조건에 맞게 다르게 정렬이 필요한 경우가 있다. 
다른 동적인 처리처럼 `BooleanExpression`를 쓴다던지 하면 error가 발생한다. 
orderBy에는 단순히 `BooleanExpression`을 사용할 수 없다. 
아래의 코드를 살펴보자 

```
@Override
public Page<memberResponse> findBySearchRequest(MemberRequest request, Pageable page) {

		JPAQuery<memberResponse> query = jpaQueryFactory.select(
                        Projections.constructor(memberResponse.class,
                            member.id,
                            member.name
                ))
                .from(member)
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize());

    // 정렬 조건 적용.
	PathBuilder pathBuilder = new PathBuilder(member.getType(), member.getMetadata());
    if ("desc".equals(request.getDateSort())) {
    	query.orderBy(new OrderSpecifier( Order.DESC, pathBuilder.get("createdAt")));
    } else {
        query.orderBy(new OrderSpecifier( Order.ASC, pathBuilder.get("createdAt")));
    }

	QueryResults<memberResponse> results = query.fetchResults();
	List<memberResponse> content = results.getResults();
    Long total = results.getTotal();

    return new PageImpl<>(content, page, total);
```

위에 코드는 page처리랑 같이 사용하다보니 아래 page처리가 추가되었다. 
코드를 살펴보면 우린 항상 .fetch() , .fetchOne(), .fetchResults() 등 원하는 형태로 결과값을 받았다. 
하지만 위 코드는 정렬조건을 나중에 해줘야 했기 때문에 .fetch()를 나중에 사용하였다. 

정렬조건을 할때는 `pathBuilder`를 사용하였고 `member.getType(), member.getMetadata()` 여기에 `member`는 내가 정렬하고자 하는 entity의 이름을 넣어주면된다. 그리고 `pathBuilder.get("createdAt") get`안에는 정렬하고 자하는 entity column명을 넣어주면된다.

그리고 마지막 3줄은 

```
	QueryResults<memberResponse> results = query.fetchResults();
	List<memberResponse> content = results.getResults();
    Long total = results.getTotal();
```

페이징 처리를 하기위해 필요한것을 뽑기 위한 과정이다. 

> 혹시 페이징 처리를 할때, groupBy를 사용할때는 .fetchResults() 를 사용하면 error가 난다. 
> 때문에 그때는 다른 식으로 접근을 해야한다. 

```
	List<memberResponse> content = query.fetch();
    return new PageImpl<>(content, pageable, content.size());
```

여기서 핵심은 total대신 List size를 구하여 넣어줬다는 점과 `fetchResults()` 가아닌 `fetch()`를 사용하여 결과를 도출해냈다는 점이 다르다. 