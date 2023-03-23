#### Object > Json String 변환

---

```
public String parseObjectToJson(Object obj) {
  
  try {
    return new ObjectMapper().writeValueAsString(obj);
  } catch (JsonProcessingException e) {
    e.toString();
    return obj.toString();
  }
}
```



#### Json String > Object 변환

---



ObjectMapper 라이브러리 임포트하여 

readValue 메소드를 사용하여 String 값을 Object로 변환할 수 있다. 

Json 형태의 String 안에 이름이 Object 필드값의 이름과 같으면 들어가진다. 

```
import com.fasterxml.jackson.databind.ObjectMapper;

private final ObjectMapper objectMapper; //ObjectMapper 라이브러리를 임포트하여 사용

TestObj test = objectMapper.readValue(String, TestObj.class);
```

