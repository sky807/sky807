# QueryDSL 기본문법 1

기본적으로 QueryDSL은 프로젝트 내의 <u>@Entity 어노테이션을 선언한 클래스를 탐색 후</u> 
<u>Q클래스를 생성</u>합니다. 



##### 기본 Q-Type 활용 

```
1. QMember qMember = new QMember("m"); //별칭 직접 지정사용
2. QMember qMember = QMember.member; //기본 인스턴스 사용
```

저는 기본인스턴스를 static import와 함께 사용했습니다.



> MemberRepositoryCustomImpl.java

```
import static kr.or.test.jpa.model.entity.@Member.member;

@RequiredArgsConstructor
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom {
	private final JPAQueryFactory jpaQueryFactory;
	
    @Override
    public List<Member> findByMember() {
        return jpaQueryFactory
        		.select(Projections.constructor(Member.class,
        			member.id,
        			memeber.userName,
        			member.phoneNumber,
        			member.age
                ))
                .from(member)
                .where(member.id.isNotNull())
                .fetch();
    }	
}
```

`@RequiredArgsConstructor`란?

자동 생성자 생성 Annotation

- final 필드에 대해 생성자를 만들어주는 lombok annotation
- Spring Framework의 DI(의존성주입) 중 Constructor Injection(생성자 주입)을 임의의 코드 없이 자동으로 설정 

`@RequiredArgsConstructor 적용 전`

```
@Component
public class LombokTest {

    private final MyService myservice;
    private final String id;

    @Autowired
    public LombokTest(MyService myservice, String id) {
        this.myservice = myservice;
        this.id = id;
    }
```

`@RequiredArgsConstructor 적용 후`

    @Component
    @RequiredArgsConstructor
    public class LombokTest {
    
        private final MyService myservice;
        private final String id;
    }


### SELECT 절에서 집합함수

---

- 멤버의 수를 구할때 - count() 함수
- 멤버의 나이의 합을 구할때 - sum() 함수
- 멤버의 나이의 평균을 구할때 - avg() 함수
- 멤버중 나이가 가장 많은사람을 구할때 - max() 함수
- 멤버중 나이가 가장 적은사람을 구할때 - min() 함수

```
@Override
    public List<MemberResponse> findByMemberGroupBy() {
        return jpaQueryFactory
        		.select(Projections.constructor(MemberResponse.class,
        			member.id(),
        			member.count(), //총 멤버수
        			memeber.age.sum(), //멤버 나이 합
        			member.age.avg(), //멤버 나이 평균
                    member.age.max(), //멤버 나이 max값
                    member.age.min() //멤버 나이 min값
                ))
                .from(member)
                .where(member.id.isNotNull())
                .groupBy(member.id)
                .having(member.age.avg().goe(28))
                .fetch();
    }	
```

그룹함수와 같이 사용할때는 `group by` 나 `having` 절은 필수이다. 
group by는 내가 보고싶은 select 에서 그룹함수를 쓰지않은 부분을 다 넣어주면되고,
having 절에서는 그룹함수의 것에 조건을 넣어줄때 사용하면된다.



여기서 나는 따로 MemberResponse 를 만들어 데이터를 받고있다. 

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class MemberResponse {
	
	private Long id;
	private Long count;
	private Integer sum;
	private Integer avg;
	private Integer max;
	private Integer min;
	
}
```



### WHERE 조건절

------

위 쿼리처럼 .select .from 을 사용하는 경우도 있지만 
select + from 을 합친 .selectFrom()으로 함께 쓰는것도 가능하다.

`.where(member.id.isNotNull().or(member.userName.eq('송하늘')))` 
검색조건에는 and(),or() 메소드를 사용하여 연결할수 있고, 
`.where(member.id.isNotNull(), member.userName.eq('송하늘'))` 
이렇게 AND 조건을 파라미터로 처리할수도있다. 



JPQL이 제공하는 모든 검색 조건 제공한다.

```
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'

member.username.isNotNull() //이름이 is not null

member.age.in(10, 20) // age in (10,20)
member.age.notIn(10,20) // age not in (10, 20)
member.age.between(10,30) // between 10, 230

member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30

member.username.like("member%") // like 검색
member.username.contains("member") // like %member% 검색
member.username.startswith("member") // like member% 검색
```



### Order By 정렬

---

정렬의 가장 기본적인 

- 회원 나이 올림차순 (asc) 
- 회원 나이 내림차순 (desc)

```
@Override
    public List<Member> findByMemberGroupBy() {
        return jpaQueryFactory
        		.select(Projections.constructor(Member.class,
        			member.id,
        			memeber.userName,
        			member.phoneNumber,
        			member.age
                ))
                .from(member)
                .where(member.id.isNotNull())
                .orderBy(member.age.desc(), member.userName.asc().nullLast())
                .fetch();
    }	
```

- order by를 통해서 정렬을 할 수 있다. 
- nullLast(), nullFirst()로 null데이터의 순서를 부여할 수 있다. 