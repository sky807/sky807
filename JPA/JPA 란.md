# JPA 란?

JPA (Java Persistence API)는  자바 진영의 **ORM** 기술 표준으로 인터페이스의 모음이다. 

> ORM (Object-Relational Mapping) 이란 객체관계매핑이라는 뜻으로 
>객체는 객체대로 설계하고, 관계형 데어티베이스는 관계형 데이터베이스대로 설계한다는 것을 의미한다.  
> 
>###### 즉 객체와 관계형 데이터베이스를 설계하면 그 데이터를 자동으로 맵핑(연결) 해준다는 의미이다. 



![img](https://media.vlpt.us/images/tmdgh0221/post/58f9b2f0-1521-4e7b-a317-45bc2d88313c/jpa_architecture.PNG)

위 그림처럼 JPA는 Java애플리케이션과 JDBC API 사이에서 동작합니다.

![img](https://blog.kakaocdn.net/dn/rb3RD/btqALtmJ7pi/FGHk84V9lP9ulbApk5GT4k/img.png)

하지만 JPA는 실제로 동작하는것은 아니다. 
JPA 인터페이스를 구현한 대표적인 오픈소스로는 Hibernate가 있다. 



### 왜 JPA를 사용해야 하는가?

------

#### 1.연관관계 매핑 

JPA가 등장하기 전에는 SQL 중심적인 개발이 이루어졌다. 
개발자가 객체지향 프로그래밍과 관계형 데이터베이스의 중간에서 반복적인 SQL을 작성하고 변환해야했다. 더불어 객체가 수정되면 그에 맞춰 SQL 모두 수정해야하는 번거로움이 있었다. 
하지만 JPA의 등장으로 ORM 프레임워크는 객체와 관계형 데이터베이스를 매핑해주고 SQL을 대신 작성하게 해주면서 번거로움이 줄어들게되었다.

#### 2. 영속성 컨텍스트 

JPA를 사용하면 객체를 조회하고 객체의 데이터를 변경만 해도 변경 내용이 데이터베이스에 자동으로 반영된다. 때문에 편리하게 데이터베이스를 관리할 수 있다.



### Spring Data JPA

------

 spring framework에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트

- CRUD 처리를 위한 공통 인터페이스 제공 
- repository 개발 시 인터페이스만 작성하면 실행 시점에서 구현객체를 동적으로 생성해서 주입 
- 공통 메소드는 spring data JPA 가 제공org.springframework.date.jpa.repository.JpaRepository  참고



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
