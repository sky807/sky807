# QueryDSL Result Handling 

쿼리를 짜다보면 결과값을 이리저리 요리하고 싶은 순간이있다. 
예를들어 나같은 경우에는 한 객체로 받고 그안에 List<String> 객체가 들어가야한다던지 
결과값을 가지고 또 한번 가공해서 쿼리를 날리고 거기서 나온 값들을 하나의 response로 관리를 하고싶다던지 이런경우에 이 방법을 사용했다. 

#### 1) Map 형태로 결과값 가공하기 

---

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class programResponse {

	private Long id; 

    private String title; 
    
    private Map<Long, Long> totalParticipantsMap; //출석한 총 학생 Map
    
    private Long totalParticipants; //출석한 학생 수
    
    public AttendanceSearchResponse(Long id, String title) {
        this.id = id;
        this.title = title;
    }
    
    (1번추가내용)
    
}

```

`@AllArgsConstructor`annotation을 사용해서 QueryDSL을 실행하면 자동적으로 순서에 맞게 들어가게된다. 그래서 사용하지않는거는 AttendanceSearchResponse() 생성자에서 빼주면 에러가 생기지않는다. 

위 처럼 선언을 한 뒤 내가 여기서 포인트를 줘야하는 부분은 `totalParticipantsMap`이부분을 주의깊게 살펴보자 

```
    /**
     * 프로그램별 출석한 총 학생 배정
     * 1번 위치에 추가해준다.
     */
    public void setTotalParticipantsMap(Map<Long, Long> totalParticipantsMap) {
        this.totalParticipantsMap = totalParticipantsMap;
        this.totalParticipants = totalParticipantsMap.get(id);
    }
```

```

   	@Override
    public List<ProgramResponse> findBySearchList() {
		
		//프로그램 조회
        List<ProgramResponse> results = jpaQueryFactory 
        		.select(Projections.constructor(ProgramResponse.class,
                                program.id,
                                program.title))
                .from(program)
                .fetch();

        for (ProgramResponse response : results) {
			
			Long programId = response.getId();
			//프로그램 마다 출석한 학생수를 구하기위함
            Map<Long,Long> totalParticipants = jpaQueryFactory
                    .select(participant.program.id.as("programId"),
                            participant.count().coalesce(Long.valueOf(0))
                            .as("totalParticipants")) //1개의 프로그램에 출석한 학생수 count
                    .from(participant)
                    .innerJoin(program).on(program.id.eq(participant.program.id))
                    .where(program.id.eq(programId)
                    		,participant.status.eq('Y')) //출석한 학생수
                    .groupBy(participant.program.id)
                    //여기 매우중요★
                    .transform(
                    	groupBy(participant.program.id)
                        .as(Projections.constructor(Long.class,participant.count()
                        .coalesce(Long.valueOf(0))))
                        );

            if (totalParticipants.size() > 0) {
                attendanceSearchResponse.setTotalParticipantsMap(totalParticipants);
            }
        }
        return results;
    }

```

위에 코드를 보면 알겠지만 처음에 results로 program List를 받는다.
그 결과값 rsults를 가지고 program id값으로 출석한 학생수를 조회한다. 
그리고 조회된 결과를 

```
.transform(
      groupBy(participant.program.id)
      .as(Projections.constructor(Long.class,participant.count()
      .coalesce(Long.valueOf(0))))
);
```

transform으로 map형태에 넣어준다. 
Map<Long, Long> 안에 들어간건  `setTotalParticipantsMap` 호출 시켜 mapping 해준다. 
그러면 하나의 response안에 두개의 쿼리를 통해 불러온 데이터를 mapping 시켜줄수있다.