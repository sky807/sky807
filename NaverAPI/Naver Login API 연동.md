Naver Login API 연동

---

1) Naver Login 화면 

ajax를 통해 네이버 url로 넘어가기전에 처리해야한 로직이 있으면 
url로 가서 한다음 결과값을 받아 팝업창 실행 



```
NaverController.class

@RequestMapping(value="/naverLogin") 
public String naverLogin (HttpServletRequest request, Model model) {
  //인증 요청문에 필요한 파라미터 만들기 
  NaverLoginVo naverLoginVo = naverService.naverLogin(request);
  
  model.addAttribute("login", naverLoginVo);
  return "login";
}
```



```
login.jsp

<div>
	<a onclick="showLoginPopup()">네이버로그인 </a>
</div>


<script>
	function showLoginPopup() {
      
      $.ajax({
        url : "/ajax/login",
        type : "POST",
        contentType : "application/json",
        async : false,
        dataType : "text",
        success : function(res) {
          var data = JSON.parse(res);
          var code = data.code;
          if (code === '0' || code === '0000') {
            //팝업창 가운데 정렬 
            var screenLeft = window.screenLeft != undefined ? window.screenLeft : screen.left;
            var screenTop = window.screenTop != undefined ? window.screenTop : screen.top;
            
            var width = window.innerWidth ? window.innerWidth : document.documentElement.clientWidth ? document.documentElement.clientWidth : screen.width;
            var height = window.innerHeight ? window.innerHeight : document.documentElement.clientHeight ? document.documentElement.clientHeight : screen.height;
            
            var leftValue = ((width - 617) / 2) + screenLeft;
            var topValue = ((height - 940) / 2) + screenTop;
            
            //화면으로 controller에서 정보보내기
            var naverClientId = ${login.clientId}; 
            var redirectUrl = ${login.redirectUrl}; 
            var state = ${login.state}; 
            
            var uri = 'https://nid.naver.com/oauth2.0/authorize?' +
            	'response_type = code' + 
            	'&client_id=' + naverClientId + 
            	'&state=' + state + 
            	'&redirect_uri=' + redirectUrl ;
            
            var naver = window.open(uri, '_blank', 'width=617, height=940, top=' + topValue + ',left' + leftValue);
            if (naver == null || naver == undefined) {
              $("#alertText").html("브라우저 팝업 차단 해지 후 이용해주세요.");
              popFancy("alert-text");
            }
            
          } else {
            $("#alertText").html("로그인 실패하였습니다. </br> 다시 시도해주시기 바랍니다.");
              popFancy("alert-text");
          }
        },
        error : function(request, status, error) {
          console.log(request.responseText);
          $("#alertText").html("로그인 실패하였습니다. </br> 다시 시도해주시기 바랍니다.");
          popFancy("alert-text");
        }
      });
	}
	
	function popFancy(name) {
      Fancybox.show([{src : name, type : 'inline'}])
	};
</script>
```

2) NaverCallback 설정 

네이버로그인 API등록시 NaverCallback 성공/실패 데이터 받을 URL 등록된 주소로 자동으로 데이터 보내준다. Controller 에서 그 URL을 만들어주고 데이터를 가공하여 화면에 보여줄 데이터만 보내준다. 

```
NaverController.class

@RequstMapping(value="/naverCallback")
public String naverCallback (@RequestParam Map<String,String> map, HttpSession session, Model model) {
  
  ResponseEntity result = naverService.naverCallback(map);
  if (result.getStatusCode().is2xxSuccessful()) {
    if (result.getStatusCode().value() == 204) {
      //통신은 성공했으나 결과값 실패일 때 분기처리 
      String resultStr = result.getBody().toString();
      
      model.addAttribute("resultMsg", false);
      model.addAttribute("errorMsg", resultStr.replace(". ", ". </br>"));
      
    } else {
      String resultStr = result.getBody().toString();
      //JsonObject data = (JsonObject) jsonParser.parse(resultStr);
      NaverProfileResponse naverProfileResponse = gson.fromJson(resultStr, NaverProfileResponse.class); //Json 데이터를 gson라이브러리를 통해 Object로 변환
      
      model.addAttribute("resultMsg", true);
      //화면에 보여질 정보담기 
      model.addAttribute("naverProfileResponse", naverProfileResponse); 
    }
  } else {
     model.addAttribute("resultMsg", false);
  }
  
  return "naverCallback";
  
}
```



```
NaverService.class

private static String REDIRECT_URL; //네이버 로그인 인증의 결과를 전달받을 콜백 URL

private static String CLIENT_ID; //애플리케이션 등록 후 발급받은 클라이언트 아이디

private static String CLIENT_SECRET; //애플리케이션 등록 후 발급받은 클라이언트 암호

private static String STATE; //애플리케이션이 생성한 상태토큰

private final Environment env;

public NaverLoginVO naverLogin(HttpServletRequest request) {
	REDIRECT_URL = request.getRequestURL().toString().replace(request.getServletPath(), "");
	REDIRECT_URL += "/naverCallback";
	
    CLIENT_ID = env.getProperty("naverClientId");
    CLIENT_SECRET = env.getProperty("naverClientSecret");
    
    STATE = 랜던값생성해서 넣기 
    
    NaverLoginVO naverLoginVO = NaverLoginVO.builder()
    			.redirectUrl(REDIRECT_URL)
    			.clientId(CLIENT_ID)
    			.clientSecret(CLIENT_SECRET)
    			.state(STATE)
    			.build();
    return naverLoginVO;
}

public ResponseEntity naverCallback(Map<String,String> map) {

  ResponseEntity tokenEntity = new ResultEntity();
  ResponseEntity profileEntity = new ResultEntity();

  //네이버 로그인 성공시에는 error가 없다 
  if (!StringUtills.isEmpty(map.get('error'))) {
  	//실패시
    String error = map.get('error');
    String errorMsg = map.get('error_description');
    
    return new ResponseEntity(ApiCode.NAVER_LOGIN_FAIL.getHttpStatus());
  } else {
    //성공시
    String code = map.get('code');
    String state = map.get('state');
    
    try {
      HttpEntity authCodeRequest = generateAuthCodeRequest(code,state);
      tokenEntity = requestAccessToken(authCodeRequest);
    } catch(Exception e) {
      return new ResponseEntity(ApiCode.NAVER_LOGIN_FAIL.getHttpStatus());
    }
    
    if (tokenEntity.getStatusCode().is2xxSuccessful()) {
      //네이버 토근 발급 성공시
      NaverTokenResponse naverTokenResponse = gson.fromJson(tokenEntity.getBody().toString(), NaverTokenResponse.class);
      
      try {
        //로그인 성공했으니, 프로필 조회 API 실행 
        HttpEntity profileRequest = generateProfileRequest(naverTokenResponse.getAccess_token());
        profileEntity = requestProfile(profileRequest);        
      } catch (Exception e) {
        return new ResponseEntity (ApiCode.Naver_PROFILE_FAIL.getHttpStatus());
      }

      if (profileEntity.getStatusCode().is2xxSuccessful()) {
      	JsonObject jsonObject = (JsonObject) jsonParser.parse(profileEntity.getBody().toString());
      	String resultCode = jsonObject.get("resultCode").getAsString();
      	if (resultCode.equals("00")) {
          //프로필 조회 API 성공시
          JsonElement jsonElement = jsonObject.get("response");
          return new ResponseEntity(jsonElement, HttpStatus.OK);          
      	} else {
                	return new ResponseEntity(ApiCode.NAVER_MEMBER_SEARCH_FAIL.getHttpStatus());
      	}
      } else {
      	//프로필 조회 API 실패시
      	return new ResponseEntity(ApiCode.NAVER_MEMBER_SEARCH_FAIL.getHttpStatus());
      }
    } else {
      //네이버 토근 발급 실패시
      return new ResponseEntity(ApiCode.NAVER_TOKEN_FAIL.getHttpStatus());
    }
  }
}


private HttpEntity<MultiValueMap<String,String>> generateAuthCodeRequest (String code, String state) {
  HttpHeaders headers = new HttpHeaders();
  headers.add("Content-type", "application/x-www-form-urlencoded; charset utf-8");
  
  MultiValueMap<String,String> params = new LikedMultiValueMap<>();
  params.add("grant_type", "authorization_code");
  params.add("client_id", CLIENT_ID);
  params.add("client_secret", CLIENT_SECRET);
  params.add("state", state);
  params.add("code", code);
  
  return new HttpEntity<>(params, headers);
}

private ResponseEntity<String> requestAccessToken(HttpEntity request) {
  RestTemplate restTemplate = new RestTemplate();
  
  return restTemplate.exchange("https://nid.naver.com/oauth2.0/token", HttpMethod.POST, request,String.class);
}

private HttpEntity<MultiValueMap<String,String>> generateProfileRequest (String accessToken) {
  HttpHeaders headers = new HttpHeaders();
  headers.add("Authorization", "Bearer" + accessToken);
  headers.add("Content-type", "application/x-www-form-urlencoded; charset utf-8");
  
  return new HttpEntity<>(headers);
}

private ResponseEntity<String> requestProfile(HttpEntity request) {
  RestTemplate restTemplate = new RestTemplate();
  
  return restTemplate.exchange("https://openapi.naver.com/v1/nid/me", HttpMethod.POST, request,String.class);
}
```



```
return "redirect:/화면위치";
RedirectAttributes flash
      flush.addFlashAttribute("resultMsg", false);

```

