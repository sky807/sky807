# JPA 란?



### JPA 사용이유
- 기존 CRUD 무한반복, 지루한 코드

    - 테이블 추가시 insert, update, select, delete sql 모두 작성, JOIN 필요시 같은 이유로 전부 수정

    - 컬럼 추가시 insert, update, select, delete sql 모두 추가

- 과거 JDBC->MyBatis->JPA 순으로 개발 생산성, 유지보수 좋음

- 객체  vs 관계형 DB 패러다임의 불일치

>어플리케이션이 발전하면서 내부의 복잡성은 커짐.
>"객체지향 프로그래밍"은 추상화, 캡슐화, 정보은닉, 상속, 다형성 등 시스템의 복잡성을 제어할 수 있는 다양한 정치들을 제공.
>"관계형 데이타베이스"는 데이터 중심으로 구조화, 집합적인 사고 필요. 추상화, 상속 다형성 같은 개념이 없다.
>객체와 관계형 데이타베이스의 패러다임 불일치. 패러다임 불일치 문제를 해결하는데 시간과 코드 소비.

~~~
// 기존
class Member {
  String id;
  long teamId; // 객체를 테이블에 맞추어 모델링한 결과  
  String username;
}
// 객체 그대로 테이블 저장
insert into MEMBER ( member_id, team_id, username ) values ...

// JPA
class Member {
  String id;
  Team team; // 객체의 참조를 그대로 유지
  String username;
}
// 테이블에 맞춘 객체 저장
member.getTeam().getId();
insert into MEMBER ( member_id, team_id, username ) values ...
~~~



### 객체 vs 관계형 DB 차이

------

#### 1. 기존 방법은 조회시에 문제

~~~
select M.*, T.*
from MEMBER M
join TEAM T on M.TEAM_ID = T.TEAM_ID
~~~
보통 이전에는 superDTO 를 만들어 전부 때려넣음 ex) memberTeamDTO

**문제는 처음 실행하는 sql 에 따라 탐색 범위가 결정됨 **
-> sql 실행시 객체에 일부 값을 셋팅 안해준다면? - 엔티티 신뢰 문제 발생 
-> 모든 객체를 미리 로딩할 수 는 없다
-> 기존에는 필요한 데이터의 로딩 마다 DAO 를 만들어 호출 

**객체 모델링, 자바컬렉션에 관리한다면 간단함**
~~~
list.add(member);
Member member = list.get(memberId);
Team team = member.getTeam();
~~~
#### 2. 비교하기
~~~
// 기존
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);
다르다

자바컬렉션에서 조회한다면?
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);
같다
~~~

>객체답게 모델링 할수록 매핑 작업만 늘어난다
>객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수는 없을까? -> JPA

 

**JPA**
>Java Persistence API 
>
>자바 진영의 ORM 기술 표준 [ORM (Object-relational mapping)]
>객체는 객체대로 설계
>관계형 데이터베이스는 관계형 데이터베이스대로 설계
>차이를 ORM 프레임워크가 중간에서 매핑

**JPA는 애플리케이션과 JDBC 사이에서 동작**

![](https://images.velog.io/images/shn807/post/e4432149-02f6-475f-a134-6ad91cfa92ca/6598d810-70dc-4994-84b1-2e879a1dbdfc.png)

>핵심은 쿼리를 JPA가 만든다
>패러다임 불일치 해결!!!!!

 

### 생산성
- 저장 jpa.persis(member) 

- 조회 Member member = jpa.find(memberId)

- 수정 member.setName("edit")

- 삭제 jpa.remove(member)

- 컬럼 추가시 -> 객체 필드값 추가만 하면됨 jpa 따로 할것 없음

  

### JPA와 패러다임의 불일치 해결

------

#### 1 ) JPA와 상속

~~~
// 자식에 insert 필요시
jpa.persist ( 자식테이블 )

// jpa가 알아서 insert 쿼리 2번생성
insert into 부모테이블..
insert into 자식테이블..

// 자식 select 시 알아서 join 해서 가져옴
Member member = jpa.find(Member.class, memberId)
// 나머진 JPA가 처리
select T.* , A.*
form TEAM T
join MEMBER M on T.team_id = M.team_id
~~~

#### 2) JPA와 연관관계, 객체 그래프 탐색

~~~
// 자유로운 객체 그래프 탐색 -> 지연로딩
Member member = memberDAO.find(memberId);
member.getTeam();
member.getOrder().getDelivery();
~~~

#### 3) JPA와 비교하기

~~~
Member m1 = jpa.find(Member.class, memberId);
Member m2 = jpa.find(Member.class, memberId);
//같다
m1 == m2 
~~~



### JPA의 성능 최적화 기능

------

#### 1) 1차 캐시와 동일성(identity) 보장

~~~
// 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
Member m1 = jpa.find(Member.class, memberId); //sql 실행
Member m2 = jpa.find(Member.class, memberId); //캐시
~~~

#### 2) 트랜젝션을 지원하는 쓰기 지연(transactional write-befind)

~~~
// 트랜젝션을 커밋할때까지 insert sql 을 모음
transaction.begin(); 

em.persist(memberA);
em.persist(memberB)
em.persist(memberC);

transaction.commit(); // 커밋하는 순간 데이터베이스에 insert sql 을 모아서 보낸다.
~~~


#### 3) 지연 로딩 (lazy loading)

~~~
// 지연로딩 : 객체가 실제 사용될때 로딩
// 즉시로딩 : JOIN sql 로 한번에 연관된 객체까지 미리 조회

Member member = memberDAO.find(memberId);
Team team = member.getTeam();  // select 
String teamName = team.getName(); // select 

// 기본은 지연로딩 -> 최적화 필요할시 즉시로딩 튜닝
~~~



### Query 메소드 기능

------

메소드 이름만으로 쿼리를 생성 
인터페이스에 메소드만 선언하면 해당 메소드의 이름으로 적절한 JPQL 쿼리를 생성해서 실행

- spring data JPA 쿼리 생성기능 

  엔티티의 필드명이 변경되면 인터페이스에 정의한 메소드 이름도 꼭 함께 변경해 주어야함!

![img](https://t1.daumcdn.net/cfile/tistory/99A9074A5FE4394B28)

```
public interface MemberRepository extends JpaRepository<Member,Long> {
	List<Member> findById(Long id);
	
	//실제로 실행된 SQL
	SELECT * FROM MEMBER WHERE ID = (id)
}
```
