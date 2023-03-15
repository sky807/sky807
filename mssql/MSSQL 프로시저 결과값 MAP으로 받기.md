### MSSQL 프로시저 결과값 MAP으로 받기 

---

mapper를 호출하기전에 service 단에서 

map을 선언하여 map안에 객체를 넣어준다. 

(객체안에 필드값 이름으로 DB에서 불려진다. 참고 할것)



```
@service
@RequiredArgsConstructor
public class MssqlService {

  private final ObjectMapper objectMapper;
  private final MssqlMapper mssqlMapper;
  
  public void getData(MssqlRequest request) {
    
    Map<String, String>map = new HashMap<>();
    //request 객체를 map에 넣기 DB에서 request 객체안에 필드값 이름으로 불러짐
    map.putAll(objectMapper.convertValue(request, Map.class));
    
    //프로시저에서 받은데이터 넣을 객체
    MssqlResponse response = new MssqlResponse();
    
    mssqlMapper.callMssql(map);
    response = objectMapper.convertValue(map, MssqlResponse.class);
    
  }
}
```



```
@Mapper
public interface MssqlMapper {
  void callMssql(Map<String, String> map);
}
```



```
<mapper namespace="co.kr.mapper.MssqlMapper">
	
	<select id="callMssql" statementType="CALLABLE" parameterType="hashmap">
	{
      call (DB에서 설정한 테이블이름) (
      	#{id}
      	,#{name}
      	,#{birth}
      	
      	,#{resultCode, mode=OUT, jdbcType=VARCHAR, javaType=String}
      	,#{resultMessage, mode=OUT, jdbcType=VARCHAR, javaType=String}

      )
	}
```



Service 단에서 호출한 mapper에서 void값으로 설정되어있지만 

프로시저에서 보내주는 값을 자동으로 map에 들어오게된다. 

받은 map을 objectMapper 를 호출하여 자동 변환 시켜서 사용할수있다. 

```
mssqlMapper.callMssql(map);
response = objectMapper.convertValue(map, MssqlResponse.class);
```

