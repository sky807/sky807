# QueryDSL CASE WHEN 문

CASE WHEN 을 사용하고 싶을때는 CaseBuilder()를 사용하면 된다. 

```
@Override
public MemberResponse findByMemberList(MemberRequest request) {
	return jpaQueryFactory.select(
                Projections.constructor(MemberResponse.class,
                        member.id.count().as("totalResponse"),
                        new CaseBuilder()
                                .when(member.gender.eq('male'))
                                .then(1)
                                .otherwise((Expression<Integer>) null)
                                .count().as("maleCount"),
                        new CaseBuilder()
                                .when(member.gender.eq('female'))
                                .then(1)
                                .otherwise((Expression<Integer>) null)
                                .count().as("femaleCount")
                ))
                .from(member)
                .fetchOne();
    }
```

여기서 나는 when 조건에 맞지 않으면 count를 하지 않기 위해 otherwise에 null을 주었다.
저기에 0이라고 적어도되지만 가끔 join을 하거나 할때 0도 count가 되는 경우가 있었다. 그래서 항상 난 null로 주고있다.

위에 방식처럼 따로따로 CaseBuilder() 사용해도되지만 

```
@Override
public MemberResponse findByMemberList(MemberRequest request) {
	return jpaQueryFactory.select(
                Projections.constructor(MemberResponse.class,
                        member.id.count().as("totalResponse"),
                        new CaseBuilder()
                                .when(member.gender.eq('male')).then('남성')
                                .when(member.gender.eq('female')).then('여성')
                                .otherwise('성별없음')
                ))
                .from(member)
                .fetchOne();
    }
```

이렇게 같이 붙여서 사용할 수도있다. 
나는 위 첫번째 문법처럼 count할때는 아래처럼 붙여서 사용하지는 않았으나 count도 붙여서 사용할 수 있지않을까? 이건 한번 테스트를 해봐야할꺼같다. 

