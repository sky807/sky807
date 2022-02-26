# Entity Mapping

Entity Mapping 은 JPA를 사용하는데 가장 중요하고 기본이 되는 작업이라고 할 수 있습니다. 그러므로 매핑 어노테이션을 정확히 숙지하고 사용하는 것이 요구됩니다. JPA가 지원하는 매핑 어노테이션은 크게 4가지로 분류할 수 있습니다. 

| 객체와 테이블 매핑 |         @Entity, @Table |
| ------------------ | ----------------------: |
| 기본 키 매핑       |                     @Id |
| 필드와 컬럼 매핑   |                 @Column |
| 연관 관계 매핑     | @ManyToOne, @JoinColumn |

### 객체와 테이블 매핑 
---
#### @Entity
JPA를 사용해서 테이블과 매핑할 클래스에는 @Entity를 꼭 붙여야 합니다. 

 >주의
 >기본 생성자 필수 (파라미터가 없는 public, protected 생성자) → JPA를 구현해서 쓰는 라이브러리들이 다양한 기술을 사용해서 객체를 프록싱 할 때 필요하기 때문!
 >final 클래스, enum, interface, inner 클래스는 @Entity로 사용할 수 없음
 >DB에 저장할 field에 final 사용 불가

**속성**

- name

  - eg)  @Entity(name = “Member”)

  - JPA에서 사용할 엔티티 이름

  - 기본값은 클래스 이름을 그대로 사용

  - 같은 클래스 이름이 없으면 가급적 기본값을 사용하도록 합니다.

#### @Table
@Entity와 매핑할 테이블을 지정합니다. 

  **속성**

- name

  - eg) @Table(name = “tblMember”)

  - 매핑할 테이블 이름을 지정 → D

  - 기본값은 엔티티 이름을 사용

- catalog

  - catalog기능이 있는 DB에서 catalog를 매핑

- schema

  - schema기능이 있는 DB에서 schema를 매핑

- uniqueConstraints (DDL)

  - DDL생성 시 유니크 제약조건 생성

  - 2개 이상의 복합 유니크 제약 조건도 만들 수 있음 → 스키마 자동 생성 기능을 사용해 DDL을 만들 때만 사용됨

### DB 스키마 자동 생성
---
JPA는 데이터베이스 스키마를 애플리케이션 실행 시점에 자동 생성하는 기능을 지원합니다. 

데이터베이스 방언을 활용해서 DB에 맞는 적절한 DDL을 생성합니다. eg)oracle : varchar2, MySQL : varchar

하지만, 이렇게 생성된 DDL은 개발 장비에서만 사용되어야 합니다. 생성된 DDL은 운영서버에 사용하면 안되고, 적절히 다듬은 후에 사용해야합니다. 

####   hibernamte.hbm2ddl.auto 속성

~~~
<property name="hibernate.hbm2ddl.auto" value="속성" />
~~~

| 옵션        | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| create      | 기존 테이블 삭제 후 다시 생성 DROP + CREATE                  |
| create-drop | create 속성에 추가로 애플리케이션 종료 할 때 생성한 DDL을 제거 DROP + CREATE + DROP |
| update      | DB테이블과 앤티티 매핑정보를 비교해서 변경 사항만 수정 ALTER |
| validate    | DB테이블과 앤티티 매핑정보를 비교해서 차이가 있으면 경고를 날림 → DDL 수정 |
| none        | 자동 생성기능 사용                                           |

>주의
>- 운영 장비에는 create, create-drop, update 사용 금지 → 운영 중인 DB의 테이블이나 컬럼을 삭제 할 >수 있기 때문!
>- 개발 초기 단계(로컬 개발 서버) →   create or update 用
>- 테스트 서버에 (여러 명이 같이 사용하는 개발 서버) →  update or validate 用
>  - but 테스트 서버에도 validate이나 none을 쓰는 것이 좋음 → data가 많은 상태에서 alter을 했을 >때 시스템이 중단될 수도 있기 때문!
>  - 직접 스크립트를 짜서 적용해 보고 문제가 없으면 DBA에게 검수 받은 후, 운영 서버에 적용하기
>- 스테이징 & 운영 서버 → validate or none 用

#### DDL 생성기능
- 제약 조건 추가
~~~
@Column(name = "userID", nullable = false, length = 50) 
private String userID;
~~~
- 유니크 제약조건 추가
~~~
  @Table(uniqueConstraints={@UniqueConstraint(name = "NAME_AGE_UNIQUE", columnNames={"NAME", "AGE"})})
~~~

>DDL 생성기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않습니다. 
>DB에만 영향을 주는 것이지 애플리케이션에 영향을 주는 것이 아닙니다. 
>JPA의 실행 매커니즘에 영향을 주는 것이 아니라 alter table과 같은 스크립트가 실행되는 것을 말합니다. 

 
### 필드와 컬럼 매핑
---
#### @Column
컬럼 매핑시 사용하는 어노테이션입니다
~~~
@Column(속성 = 속성 정보)
~~~

#### 속성

- name

  - @Column (name =”tblMemberIdx”) private Long id;

  - 객체명과 DB컬럼을 다르게 하고 싶을 경우에 사용

  - 기본 값은 필드 이름으로, name에는 실제 DB의 컬럼 값을 적어야 함

- insertable

  - @Column (insertable = false)

  - 삽입 가능여부로 기본 값은 true

- updatable

  - @Column (updatable = true)

  - 수정 가능여부로 기본 값은 true

  - false 설정시 변경 되어도 DB에 반영 되지 않음

- nullable

  - @Column (nullable = true)

  - Not null 제약 조건, false로 설정 시 not null 제약조건이 걸림

- unique

  - @Table의 uniqueConstraints와 같지만 컬럼에 간단히 유니크 제 약조건을 걸 때 사용

  - but! 잘 사용하지 않음 → constraint UK_ewkrjwel239flskdfj01 unique (name)와 같이 이름을 랜덤으로 생성하기 때문

- columnDefinition

  - @Column(columnDefinition = ”varchar(100) default 'EMPTY'”)

  - 직접 컬럼 정보 설정

- length

  - @Column (length = 100)

  - 문자 길이 제약 조건

  - String 타입에만 적용

- precision, scale

  - @Column(name = "SALARY", precision = 10, scale = 2)

  - BigDecimal타입에 사용

  - precision → 소수점 포함 전체 자리 수

  - scale → 소수의 자리 수

  - double, float에 적용   → 아주 큰 숫자나 정밀한 숫자를 다룰 때 사용 

#### @Temporal
날짜 타입( java.util.Date, java.util.Calendar ) 매핑 시 사용합니다

~~~
@Temporal(TemporalType.속성)
~~~

 **속성**

- TemporalType.DATE

  - 날짜 정보만 저장 →  DB의 date 타입과 매핑

  - eg) 2021-03-11 

- TemporalType.TIME

  - 시간 정보만 저장 → DB의 time 타입과 매핑

  - eg) 22:52:29

- TemporalType.TIMESTAMP

  - 날짜와 시간정보 저장 → DB의 timestamp타입과 매핑

  - eg) 2021-03-11 22:52:29

>java 8 버전 이상에서는 생략이 가능합니다
>>java 8 의 LocalDate(date)와 LocalDateTime(timestamp)은 최신 하이버네이트를 지원하기 때문에 생략이 가능합니다. 

#### @Enumerated
JAVA enum 타입 매핑시 사용합니다

**  속성**
~~~
@Enumerated(EnumType.속성)
~~~

- EnumType.String

  - enum 이름을 DB에 저장

- EnumType.ORDINAL

  - enum 순서를 DB에 저장

  - 기본 값 

>주의
EnumType.ORDINAL은 사용하지 않습니다 → 요구 사항에 따라 새로운 Type이 추가 되어 순서가 변경 되면 혼돈이 생기기 때문에 data가 꼬이게 됩니다. 

#### @Lob
DB에서 varchar을 넘는 큰 내용을 넣고 싶은 경우 사용하는 애노테이션 입니다

- @Lob에는 지정할 수 있는 속성이 따로 없음 → 매핑하는 필드 타입에 따라 BLOB 과 CLOB이 매핑됨

  - CLOB : 필드 타입의 문자열 eg) String, char[], java.sql.CLOB

  - BLOB : 그 외 나머지 eg) byte[], java.sql.BLOB

#### @Transient
특정 필드를 컬럼에 매핑하지 않을 때 사용하는 애노테이션 입니다

- @Transient가 붙은 필드는 DB에 저장, 조회가 되지 않음

- DB에 관계없이 메모리에서만 사용할  필드에 사용 → 주로 메모리 상에서만 임시로 어떤 값을 보관하고 싶을 때 사용


### 기본 키 매핑
---

#### @Id
직접 할당 시 pk가 될 필드에 사용합니다

#### @GeneratedValue
자동 생성 시 pk가 될 필드에 사용합니다

**속성 (전략)**
~~~
@GeneatedValue(startegy = GenerationType.속성)
~~~

- IDENTITY

  - 기본 키 생성을 DB에 위임 → id 값을 null로 하면 DB가 AUTO_INCREMENT 처리

  - eg) MySQL, PostgreSQL, SQL Server, DB2 등

>IDENTITY 전략 : entityManager.persist() 시점에 즉시 INSERT 쿼리 실행 하고 DB에서 식별자 조회 하기
- JPA는 보통 트랜잭션 commit 시점에 insert쿼리 실행
- auto_increment는 DB에 insert 쿼리 실행 후에 id값을 알 수 있음
- 문제점 : 영속성 컨텍스트에서 특정 객체가 관리되려면 무조건 pK값이 있어야 함 즉, id를 알 수 있는 시점은 DB 에 값이 들어 간 순간임
- 해결책 : IDENTITY 전략에서만 예외적으로 entityManager.persist()가 호출되는 시점에 바로 DB에 insert 쿼리 날리기 → DB에서 식별자 조회하여 영속성 컨텍스트의 1차 캐시에 값을 넣음 (select 쿼리 호출 불필요)
- 단점: 모아서 insert 불가 but 하나의 트랜잭션안에서 여러 insert 쿼리가 실행한다고 해도 비약적인 차이가 나지 않음 → 크게 신경쓰지 않아도 됨

- SEQUENCE

  - DB에 시퀀스 오브젝트 사용 → 유일한 값을 순서대로 생성하는 특별한 DB 오브젝트

  - eg) Oracle, H2, PostgreSQL

  - @SequenceGenerator 필요 

    - eg)@SequenceGenerator( name = ”MEMBER_SEQ_GENERATOR, sequenceName= ”MEMBER_SEQ”, initialValue = 1, allocationSize = 1)

    - name : 식별자 생성기 이름 (필수)

    - sequenceName : DB에 등록되어 있는 시퀀스 이름 (hibernate_sequence)

    - initalValue : DDL 생성 시에만 사용됨. 시퀀스 DDL을 처음 생성할 때 처음 시작하는 수 지정

    - allocationSize : 시퀀스 한 번 호출에 증가하는 수 (50) → 성능 최적화 시 사용

      - DB에 시퀀스 값이 하나씩 증가하도록 설정되어있으면 이 값을 반드시 1로 설정해야 함

    - catalog, schema : DB의 catalog, schema이름

>SEQUENCE 전략
- id값을 설정하지 않고 generator에 매핑된 Sequence전략에서 id 값을 얻어온다 → DB가 관리하는 것이기 때문에 DB에서 id 값을 가져와야 함
- 문제점 1: IDENTITY 전략과 같음
- 해결책 1:
  - entityManager.persist()를 호출하기 전에 DB의 sequence에서 pk 값을 가져오기
    - hibernate: call next value for MEMBER_SEQ 수행
  - DB에 가져온 pk 값을 해당 객체의 id에 넣기
  - 이후엔 entityManager.persist() 를 통해 영속성 컨텍스트에 해당 객체 저장 → 이 시점엔 insert 쿼리 실행 되지 않고 영속성 컨텍스트에 쌓야였디가 트랜잭션이 commit하는 시점에 insert 쿼리 실행
- 문제점 2: sequence를 매번 DB에서 가지고 오는 과젱에서 성능 상의 저하를 가져올 수 있음
- 해결책 2:allocationSize 속성 사용 (성능 최적화)
  - next call을 할 때 미리 DB에 50개(기본 값)를 한번에 올려놓고 메모리 상에서 1개씩 사용
  - 50개를 모두 사용 후에 next call을 날려서 다시 50개를 올려 놓음

>이론적으론 allocationSize를 더 큰 수로 설정 할수록 성능은 좋아짐 but 웹서버를 내리는 순간 seq 구멍이 생김. 이는 큰 문제는 아니지만 낭비가 되므로 50~100사이로 설정하는 것이 좋음

- TABLE

  - 키 생성용 테이블을 만들어서 DB sequence를 흉내내는 전략

  - @TableGeneator필요 

    - eg)  @TableGeneator(name = ”MEMBER_SEQ_GENERATOR”, table = ”MY_SEQUNECES”, pkColumnValue = ”MEMBER_SEQ”, allocationSize =1)

    - name : 식별자 생성기 이름 (필수)

    - table : 키 생성 테이블 명 (hibernate_sequence)

    - pkColumnName : 시퀀스 컬럼명 (sequence_name)

    - valueColumnName : 시퀀스 값 컬럼명 (next_val)

    - pkColumnValue : 키로 사용할 값 이름 (엔티티 이름)

    - initialValue : 초기 값, 마지막으로 생성된 값이 기준 (0)

    - allocationSize : 시퀀스 한 번 호출에 증가하는 수 , 성능 최적화에 사용 (50)

    - catalog, schema : DB의 catalog, schema이름

    - uniqueConstraints(DDL): 유니크 제약조건 지정

   - 모든 DB에 적용 가능

   - but 최적화 되어있지 않은 테이블을 직접 사용하기 때문에 성능상 이슈가 있음

- AUTO

   - 방언에 따라 자동 값 지정

   - 기본 값

#### 권장 식별자 전략
#### 기본 키 제약 조건
>1. Not Null
2. 유일
3. 변하면 안된다

but 미래까지 이 조건을 만족하는 자연키는 찾기 어려우므로 대체키를 사용헤야 합니다 → eg) Geneate Value, 랜덤 값 등

>권장 식별자 구성 전략 :    (Long 형) + (대체키) + (적절한 키 생성 전략)   
~~~
