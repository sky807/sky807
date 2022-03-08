# QueryDSL - Sub query

내가 실제로 사용한 코드를 예시로 서브쿼리 사용법을 정리해보려고 한다. 
실제 성능상 subquery는 속도가 느려지니 필요하다면 
Join문, 쿼리를 나눠서 실행할 순 없는지 back에서 처리할 순 없는지 충분히 고려한 뒤 사용해야할꺼같다.  

- select절 
- where절 

> From절에서는 subquery를 지원하지 않는다. 



#### 1) Select절에서 사용할때

`ExpressionUtils.as`를 사용합니다. 
ExpressionUtils는 Querydsl 내부에서 새로운 Expression을 할 수 있도록 지원합니다.
그리고 as를 통해 결과물을 alias 시킵니다.

```
@Override
public List<AttendanceResponse> findByMemberList(AttendanceRequest request) {
	return jpaQueryFactory.select(
                Projections.constructor(AttendanceResponse.class,
                        attendance.name,
                        ExpressionUtils.as(
                            JPAExpressions.select(member.phoneNumber)
                            .from(member)
                            .where(member.name.eq(attendance.name)),
                            "phoneNumber")
                ))
                .from(attendance)
                .fetch();
    }
```



#### 2) Where절에서 사용할때

select와는 달리 바로 `JPAExpressions` 만으로 생성할 수 있다. 

```
@Override
public List<Attendance> findByMemberList(AttendanceRequest request) {
	return jpaQueryFactory
				.selectFrom(attendance)
				.where(attendance.member.id.in(
                        JPAExpressions
                            .select(member.id)
                            .from(member)
                            .where(member.name.eq(request.memberName))
				))
                .fetch();
    }
```

이렇게 사용하면 where절에서도 사용할 수 있다. 

나는 여기서 in()안에 내용물을 method로 분리시켜 사용하였다. 
`attendance.name.in(memberNameEq(request.memberName))` 이렇게 바꾸어서 사용하였다. 
굳이 이렇게 밖으로 뺀이유는 동적 쿼리를 넣어주기 위함이었다. 

```
private List<Long> memberNameEq(String memberName) {
	A...(이부분)
  return jpaQueryFactory
                .select(member.id)
                .from(member)
                .where(builder) -> 여기를 builder로 바꾸었다.
                .fetch();
} 
```

위 동적쿼리를 설명하기 전에 알아야 할 것이 있다. 
`BooleanBuilder` `BooleanExpression`이다. 
쉽게 말하자면 쿼리의 조건 설정인 where뒤의 조건을 생성해주는것이라고 생각하면 된다. 

#### BooleanBuilder사용

> `import org.apache.commons.lang3.StringUtils;`  stringUtils import lang3추천
>
> isNotEmpty도 있다. 다른 라이브러리 보다 더 많은 기능을 가지고 있다. 

```
private List<Long> memberNameEq(String memberName, String memberAddress) {
	BooleanBuilder builder = new BooleanBuilder();
	
	if(StringUtils.isNotEmpty(memberName)) {
      builder.and(member.name.eq(memberName))
	}
	
	if(StringUtils.isNotEmpty(memberAddress)) {
      builder.and(member.adress.eq(memberAddress))
	}
	
  	return jpaQueryFactory
                .select(member.id)
                .from(member)
                .where(builder) -> 여기를 builder로 바꾸었다.
                .fetch();
} 
```

> 여기서 `builder.and`  `builder.or` 둘다 사용가능하다

if문으로 필요한 부분만 BooleandBuilder에 추가추가 하면서 쿼리를 만드는 형태이다. 



#### BooleanExpression 사용

```
private List<Long> memberNameEq(String memberName, String memberAddress) {
	return jpaQueryFactory
                .select(member.id)
                .from(member)
                .where(nameEq(memberName),
                	   addressEq(memberAddress))
                .fetch();
}

public BooleanExpression nameEq(String name){
        return StringUtills.isEmpty(name) ? null : member.name.eq(memberName);
    }
    
public BooleanExpression addressEq(String address){
        return StringUtills.isEmpty(address) ? null : member.address.eq(address);
    }
```

확실히 두 코드의 차이점을 살펴본다면 `BooleanExpression` 코드를 보기가 확실히 편하다 그래서 사용을 추천한다.  위에 내용들은 단순한 name이 있다 없다 address가 있다 없다의 간단한 구분이지만 이 안에서 충분히 응용할 수 있다. 날짜를 오늘날짜와 비교할 수 도있고, select해서 가져온 데이터를 가지고 가공할 수도있고 어떻게 활용할지 어떻게 접근할지 풀어낼지 고민하길 바란다. 

