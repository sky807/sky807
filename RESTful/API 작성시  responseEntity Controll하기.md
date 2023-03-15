### API 작성시  responseEntity Controll하기 

---

API를 만들때 request/response 형태를 정하여 만들기 시작하는데 

response부분을 controll 하는 방법을 작성했다. 



일단 ResponseAdvice.class 를 작성

```
@Slf4j
@RestControllerAdvice
public class ResponseAdvice <T> implements ResponseBodyAdvice<T> {
	
}

```

@Override 

supports, beforeBodyWrite 메소드를 오버라이드 하게되는데 

여기서 beforeBodyWrite 메소드에 Exception Handler하기전 지나가는 메소드로 로그를 남겨준다 

(나는 실제로 받은 데이터를 로그로 남겨주었다.)



이제 실제로 Exception의 종류별로 분기처리하여 response를 가공해보자  

```
@ExceptionHandler({Exception.class}) //Exception 걸리면 이쪽으로 와서 실행된다
protected ResponseEntity<T> handleOutOfBounds(Exception e, WebRequest request) {
  
  ApiCode apicode = ApiCode.GLOBAL_ERROR; //미리 enum으로 오류상황별로 분기처리하여 code만듦
  
  //response data 초기선언
  Map<String, Object> map = new HashMap<>();
  Object body = map;
  
  //service method 이름 가져오기 (발생된시점 알기위해서)
  String[] uri = ((ServletWebRequest)request).getRequest().getRequestURI().split("/");
  String method = uri[uri.length-1];
  
  //API통신시 Header값 찾는 방법 
  String header = request.getHeader("찾으려는 헤더이름");
  
  //method별 분기처리 하기 (각 API별로 response가 다를수있기에)
  ApiResponse apiResponse = new ArrayList<>();
  body = apiResponse;
  
  return (ResponseEntity<T>)ResponseEntity
  		.status(apiCode.getHttpStatus())
  		.body(body);
}
```

중간에 ApiResponse안에 값들을 가공하여 body Object에 넣어주고 

마지막에 ResponseEntity body안에 넣어서 값을 보내준다. 



위에는 Exception을 사용했지만 

JsonProcessingException, NullPointException등등 내가 원하는 exception을 분기처리하여 

ApiCode로 어디서 어떤오류인지 좀더 세세하게 분류시킬수있다. 

또한 내가 ApiException으로 만들어서 가공할수도있다. 

`@ExceptionHandler({ApiException.class})` 이렇게 적어주면 된다. 



그리고 ApiException은  

`throw new ApiException();`이렇게 서비스나 컨트롤러에서 선언하여 불러올수있다. 