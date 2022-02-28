# QueryDSL 기본문법 2 



### PAGING 처리

------

```
@Override
public Page<MemberSearchResponse> searchList(MemberSearchRequest request, Pageable pageable) {
    List<MemberSearchResponse> results = jpaQueryFactory
            .select(
                    Projections.constructor(MemberSearchResponse.class,
                            member.id,
                            memeber.userName,
                            member.phoneNumber,
                            member.age
            ))
            .from(member)
            .where(member.id.isNotNull())
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

    JPAQuery<Member> countQuery = jpaQueryFactory
            .selectFrom(member)
            .where(member.id.isNotNull());

    return PageableExecutionUtils.getPage(results, pageable, countQuery::fetchCount);
}
```

나 같은 경우에는 페이징 처리를 할때 내용을 뽑는 첫번째 쿼리는 List로 뽑고, countQuery를 두번째로 사용하여 
마지막에 두 쿼리를 합쳐서 사용하고 있다. 

return 값은 Page<MemberSearchResponse> 형태로 보내주고 
`MemberService.java`에서 아래의 방식으로 사용한다. 

```
@Service
@Transactional(readOnly = true)
@Slf4j
@RequiredArgsConstructor
public class EduContentService {

	private final MemberRepository memberRepository;

    public Pagination search(MemberSearchRequest request, Pageable pageable) {

        log.debug("회원 검색 조건에 맞춰 조회");
        return new Pagination(memberRepository.findBySearchList(request, pageable));
    }
    
}

```



이건 간단한 paging처리였고, QueryDSL에서 having절을 쓸때는 위에 방식으로 사용할 수 가없었다.
그래서 having절을 쓸때는 아래의 방식을 사용하여 paging처리를 하였다.

```
@Override
public Page<MemberSearchResponse> findByHaving(MemberSearchRequest request, Pageable pageable) {
    List<MemberSearchResponse> results = jpaQueryFactory
            .select(
                    Projections.constructor(MemberSearchResponse.class,
                            member.id,
                            memeber.userName,
                            member.phoneNumber,
                            member.age.sum()
                    ))
            .from(eduCourse)
            .where(member.id.isNotNull())
            .groupBy(member.id,
                    memeber.userName,
                    member.phoneNumber)
            .having(member.age.gt(member.age.sum()))
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .orderBy(eduCourse.id.desc())
            .fetch();

    List<Long> list = jpaQueryFactory
            .select(member.id)
            .from(member)
            .where(member.id.isNotNull())
            .groupBy(member.id,
                    memeber.userName,
                    member.phoneNumber)
            .having(member.age.gt(member.age.sum()))
            .fetch();

           return new PageImpl<>(results, pageable, list.size());
}
```

countQuery를 사용하는 것이 아니라 같이 List형태로 뽑고 나중에 list.size를 pageImpl에 넣어주었다. 
paging 처리는 더 많은 방법이 있기에 천천히 update를 해보겠다. 



### JOIN - 기본조인

---

- Join은 InnerJoin, leftJoin, rightJoin을 사용할 수 있다.
- Join 이후에 on을 넣어서 대상 지정을 넣을 수 있다.  (querydsl에서 on절은 JPA 2.1 이상부터 적용이 가능하다.)
- 연관관계가 없어도 조인을 할 수 있다. 

1) 연관관계 있을때

```
@Override
    public List<Member> findByTeamJoin() {
        return jpaQueryFactory
        		.select(Projections.constructor(Member.class,
        			member.id,
        			team.id
                ))
                .from(member)
                .leftJoin(member.team, team)
                .where(team.name.isNotNull())
                .fetch();
    }	
```



2) 연관관계 없을때 

Member 테이블의 name 과 Team 테이블의 name 이 같은걸 꺼내서 매칭시킨다.

```
@Override
    public List<Member> findByTeamJoin() {
        return jpaQueryFactory
        		.select(Projections.constructor(Member.class,
        			member.id,
        			team.id
                ))
                .from(member)
                .leftJoin(team).on(member.name.eq(team.name))
                .where(team.name.isNotNull())
                .fetch();
    }	
```



- fetchJoin

우리는 흔히 `select()`절에 있는 entity만 꺼내진다. 
`where()`조건절에서 쓰인다 해도  `select()`절에 없으면 따로 데이터를 뽑지않는다. 

하지만,
join된 테이블의 값을 같이 꺼내고 싶을 경우, `select()`절에 하나하나 쓰는 불편함을 해소 시켜주는 `fetchJoin` 을 사용하면 한번에 꺼내올수있다. 

```
  public List<Member> findByFetchJoin() {
      return selectFrom(member)
          .leftJoin(member.team, team)
          .fetchJoin()
          .where(team.name.isNotNull())
          .fetch();
  }
```



내가 생각했을때 join문에서 가장 중요한 부분은 조건을 어디에 걸어두냐일꺼같다. 
뭐 이 부분은 굳이 join문이 아니더라도 해당이 될 수 있지만 특히 join문에서 on절을 주의해야한다. 

> ON 절과 WHERE 절의 차이는 ON 절 같은 경우는 JOIN 할 데이터를 필터하기 위해서 사용하는 반면에 
> WHERE 절은 JOIN 을 하고나서 데이터를 필터하기 위해서 사용한다고 생각하면 된다. 
> 즉 ON 절이 WHERE 절보다 먼저 실행이 되고 이는 LEFT_OUTER_JOIN 을 하면 뚜렷히 드러난다.
>
> ON절을 활용하여 조인대상을 필터링 할 때는 InnerJoin을 이용할땐 where 과 결과적으로 동일하다. 
> 하지만 leftJoin에서는 ON절에 넣을때와 where 절에 넣을때는 결과가 달라지므로 주의해야한다. 



### 결과조회 

---

- fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환

- fetchOne() : 단 건 조회

   결과가 없으면 : null

   결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException

- fetchFirst() : limit(1).fetchOne()

- fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행

- fetchCount() : count 쿼리로 변경해서 count 수 조회

