# 개발자의 숙명 Naming 의 중요성 



![img](https://blog.kakaocdn.net/dn/uszjY/btqz7yPMDNZ/ax2KFfIqC0tuQnamkaBvp0/img.jpg)



프로그래밍 영역에서는 영어를 사용하고있다. 때문에 영어로 이름을 짓는것이 꽤나 어려웠다. 
영어 실력도 실력이지만 문맥에 맞게 사용해야 할것같다. 
내가 배운 Naming의 방식들이 여러개 있다. 오늘은 그 방법을 간단하게 소개해보려고한다. 



### 1. 절대 축약하지마라 

나는 처음에 개발을 할때 축약하는것이 여러모로 좋은줄알았다. 뭔가 길게 구구절절 써서 단어가 길어진다거나 메소드명,엔티티명 같이 계속 선언을 해야하는 명들이 길어지거나 두줄보다는 한줄이 10단어보다는 5단어가 더 좋아서 조금씩 줄이려고 했었다. 

하지만! 
개발은 혼자하는것이 아니다. 그렇기 때문에 내가 축약하거나 줄여버렸을때 그 의미 전달이 왜곡될수도 있고 내가 simple하게 기재해둔 덕에 다른사람들은 못찾을수 있다는 사실을 알게되었다. 
그때부터였다... naming이 어려워지기 시작한것이...



나 같은 경우에는 한가지 예를들어 

```
MemberSearchRequest -> 멤버검색request
MemberSearchResponse -> 멤버검색response
MemberCreationRequest -> 멤버등록request
MemberCreationResponse -> 멤버등록response
...
```

이런식으로 Entity명 + 쓰여지는곳(view) + request,response 를 사용하여 
요청(Request), 응답(Response) 같은 Entity를 작성할때  최대한 풀어서정리를 한다. 

 

### 2. 명사를 사용한다.

최대한 단어를 선택할때 명사를 사용하려고 한다. 
그래서 아까위에서 보여준 예시에서 **create**가 아닌 **creation**을 사용하였다. 
가끔은 아는단어를 사용할때는 동산지 명산지 생각않고 바로 사용할때도 있지만 최대한 이렇게 
습관을 들이려고 노력하고있다. 



### 3. 단수/복수 잘 선택하기

내가 가장 놓치고 있고 가장어려운 부분인거같다. 
자체적인 의미로 복수의 의미를 가진 영어단어들도 많기때문에 
이 부분은 계속 내가 들여다보는 수밖에 없는거같다. 

Data는 그 단어 자체로 복수형을 띄고있고, 
List나 Array같은 naming에는 복수형을 사용해야하며
Class,Method,Entity의 naming말고 경로를 정할때도 중요하다.
특히 경로는 경로만으로 이 페이지가 뜻하는게 무엇인지 알아야 하기에 더욱더 중요하다.



### 4. 중복하지마라

내용을 중복하지 않는 것! 이것은 중요하다 
이 부분은 한 class를 예를 들어 설명하겠다.

```
@Controller
@Slf4j
@RequiredArgsConstructor
@RequestMapping("/attendance-survey/survey")
public class SurveyController {

    private final SurveyService surveyService;

    @GetMapping
    public String list(SurveySearchRequest surveySearchRequest) {

        log.debug("설문조사 목록 페이지 이동");
    }

    @GetMapping(params = "create")
    public String createForm() {

        log.debug("설문조사 페이지 이동");
    }

    @PostMapping
    public void create(SurveyCreationRequest surveyCreationRequest) {

        log.debug("설문조사 등록하기");
    }
    
    @GetMapping(params = "update")
    public String updateForm(M {

        log.debug("설문조사 페이지 이동");
    }


    @PutMapping("/{id}")
    public String update(@PathVariable Long id) {

        log.debug("{} 설문조사 수정하기", id);
    }
    
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Long id) {

        log.debug("{} 설문조사 삭제하기", id);
    }

}

```

Survey라는 패키지명 안에 Controller이다. 
list,create,update,delete등 모두 여러곳에서 사용하는 메소는 일것이다. 
처음에는 나도 surveyList 이런식으로 한번더 Survey를 기재해뒀던거같다. 
하지만 Survey라는 패키지 안에 들어있는 형태기에 list라고 하면 surveyList를 뜻하는것이기에 
중복을 최대한 줄이려고 노력했다. 
그리고 createForm updateForm 처럼 화면이 필요한 부분은 param 값으로 create,update 구분지었다. html파일도 Directory명을 survey로 만들어서 그안에 list.html , create-form.html, update-form.html 이렇게 규칙을 두어 선언하였다. 



그리고 패키지명,directory명 같은 naming할때는 소문자를 사용하였고 
그 밖의 class,interface,enum 등 naming할때는 대문자로 시작하고 camelCase를 사용하였다. 
하지만 html,css,js파일등은 모두 소문자로 시작하고 camelCase가 아닌 -하이픈을 표시했다.   



나의 의도를 간단하지만 정확히 전달하는 방법이 가장 중요한 naming에서 
요즘에는 구글링만 해도 애매한부분이나 아니면 사람들이 가장 많이 사용하는 naming도 
추천해주는 사이트도 여러개있다.

[변수명짓기]: https://www.curioustore.com/#!/util/naming

이런 사이트에 애매모호한건 찾아보고 사람들이 이런단어를 가장 많이 사용하는구나 하고 가끔씩 
찾아보는 습관을 기르는것도 중요한거같다. 

 

