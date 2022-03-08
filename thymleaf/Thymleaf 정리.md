# Thymeleaf 문법정리
Thymeleaf 란?
템플릿 엔진의 일종으로 흔히 View Template (뷰 템플릿) 이라고 불리며 
컨트롤러가 전달하는 데이터를 이용하여 동적으로 화면을 구성할 수 있게 해줍니다.



`application.properties`에

`spring.thymeleaf.mode=HTML`을 추가한 후

#### 1.Tymeleaf dependency 추가 (maven/gradle)

Maven

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Gradle

```
dependencies {
...
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	}
```



### 기본사용법

---

크게 4가지 방법으로 사용할 수 있다. 

1. ${} - 변수식
2. #{} - 메세지방식
3. *{} - 객체변수식
4. @{} -링크방식

```
<!DOCTYPE html> 
<html lang="ko" xmlns:th="http://www.thymeleaf.org"> 
<head> 
<meta charset="UTF-8"> 
    <title>Insert title here</title> 
</head> 
<body> 
</body> 
</html>
```

먼저 html파일을 만들면 자동으로 생성이 되는데 
<html>태그에  위와 같이 바꿔줍니다. 
html 문서안에 한글과 타임리프를 사용하겠다는 의미입니다. 

Controller에서 Model을 통해 테이터를 화면으로 넘겨줍니다. 

```
@RequestMapping("/test")
	public String test(Model model) {
        model.addAttribute("test","controller에서 데이터보내기");
		return "test";
	}
```

`model.addAttribute`는 model 객체안에다가 데이터를 넣어주는 것으로 
 key:value형식으로 앞에 첫번째는 키이름, 두번째는 키의 값을 넣으면 됩니다. 


그리고 다시 html로 와서 아래처럼 사용을 하면됩니다.
타임리프 엔진에서는 th:를 사용합니다. 

```
<p th:text="${test}"></p>
```

`th:text` `th:value` `th:id  ` `th:name` `th:onclick` `th:src` ... 이런형태로 사용할수있습니다.



#### for문 사용법

```
<th:block th:each="searchRequest : ${searchRequest}">
	<a th:text="${searchRequest.name}+TEST" th:id="${searchRequest.id}"></a>
</th:block>
```

```
<tr th:each="postList, i : ${postList}">
	<a name="positions" th:id="${i.index}" th:value="${i.index}"></a>
</tr>
```

for문을 사용할때는 두가지 방법이있습니다. 
직접쓰고 싶은 태그에 바로 선언을 해도되는데 첫번째 처럼 `th:block` 으로 한번 감싸고 사용할 수도있고 for문안에서 index가 필요할때는 두번째처럼 선언해서 사용합니다.

- index 현재 반복 인덱스 (0부터 시작)
- count 현재 반복 인덱스 (1부터 시작)
- current 현재 요소
- even 현재 반복이 짝수인지 여부 (boolean)
- odd 현재 반복이 홀수인지 여부 (boolean)
- first 현재 반복이 첫번째인지 여부 (boolean)
- last 현재 반복이 마지막인지 여부 (boolean)



컬렉션 없이 단순 반복처리를 하고싶을때는 Numbers Class의 Utility method인 `#numbers.sequence`를 사용하여 원하는 반복횟수 만큼의 배열을 생성한뒤 th:each의 컬렉션에 넣어주면됩니다.

```
${#numbers.sequence(from,to)} 
${#numbers.sequence(from,to,step)}
```

```
<th:block th:each="num : ${#numbers.sequence(1,5)}"> 
	<div th:text="${num}"></div> 
</th:block> --단순반복
```

```
<th:block th:each="num : ${#numbers.sequence(0, test.size()-1)}">
</th:block> --데이터의 개수만큼 반복시킬때 사용
```



#### URL 링크 표현식

th:href로 치환해 링크를 적을때는 @{}사용합니다.
그런데 그안에 변수를 사용하고싶을때는 아래처럼 사용합니다. 

```
th:href="@{/search/list?id={id}(id=${test.id})}"
```

앞에있는 {id}와 옆에 id=은  내가 원하는 변수명으로 바뀌줄수있습니다. 

```
<img th:src="@{/test/} + ${main.thumbnailId}"/>
```

@{}안에 변수를 사용할때는 많은 방법이 있는거같습니다. 
이렇게 + 로 나누어서 사용할수도있습니다.

```
<form th:action="@{/search/list}" method="POST" id="postForm">
```

form 태그 안에있는 action또한 @{}링크로 표현합니다. 



#### onclick안에 

onclick안에는 바로 href나 아니면 자바스크립트 함수를 쓸때 안에 변수를 쓰고싶을때가 있습니다.
그럴때는 아래처럼 사용합니다.

```
th:onclick="location.href='/test/[[${main['id']}]]?list'"
```

```
th:onclick="modalOpen('[(${main.id})]');"
```



#### Null check

객체안에 null인지 체크할때는 
if문을 사용하여 빈값을 체크하고 그반대의 경우에는 unless를 사용합니다. 

(if문과 unless는 조건식이 동일합니다.)

```
<div th:if="${not #strings.isEmpty(main)}">
<div th:unless="${not #strings.isEmpty(main)}">
```

만약 if문으로 감싸기가 싫을때는 나는 이 방법도 사용합니다.
`"${main?.id}"` ?를 사용하면 없으면 빈값이 들어오게됩니다. 
만약 이렇게 사용하지 않았는데 빈값일때 error발생합니다.

```
<td><input type="text" name="mainId" th:value="${main?.id}" ></td> 
```



#### 날짜변환

만약 날짜를 내 format으로 변환하고싶을때는 #temporals.format() 사용하여 변환할 수 있습니다.

```
<p th:text="${#strings.isEmpty(member.updateDate)}? '-': ${#temporals.format(member.updateDate, 'yyyy.MM.dd HH:mm')}"></p>
```