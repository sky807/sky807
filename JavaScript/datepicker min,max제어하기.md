# datepicker min,max제어하기

웹 페이지를 검색하다 보면, 날짜 관련해서 입력을 요구할 때가 있다. 
그럴때 jQuery에서 제공하는 달력 형식의 UI위젯중 하나인 datepicker를 사용해서 minDate, maxDate를 
제어하는 방법을 알아보자.

 우선 사용하기 전에 아래 스크립트를 import해주자!

```
 //jQuery ui css
 <link rel="stylesheet" href="//code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">
 
 //jQuery style css
 <link rel="stylesheet" href="/resources/demos/style.css">
 
 //jQuery js
 <script src="https://code.jquery.com/jquery-1.12.4.js"></script>
 
 //jQuery ui js
 <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
```



간단하게 예제를 살펴보면 

```
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>제이쿼리 위젯 달력 사용하기</title>
  <link rel="stylesheet" href="//code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">
  <link rel="stylesheet" href="/resources/demos/style.css">
  <script src="https://code.jquery.com/jquery-1.12.4.js"></script>
  <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
  <script>
  $( function() {
    $( "#datepicker" ).datepicker();
  } );
  </script>
</head>
<body>
 
<p>Date: <input type="text" id="datepicker"></p>
 

</body>
</html>
```

위 코드 처럼 사용할 수 있다. 
나는 여기서 추가적으로 두개의 datepicker를 사용할때 
시작일, 종료일을 설정할 때가 있다. 
그럴때마다 주로 쓰이는 기능들을 알아보겠다. 

1. 시작일 선택시 자동으로 종료일도 시작일로 설정
2. 시작일 선택시 종료일 이후는 선택할 수 없음 
3. 종료일 선택시 시작일 이전은 선택할 수 없음 

위 3가지의 기능을 구현해보도록 하자. 
그러기 위해서는 나는 화면의 설정을 일단 아래처럼 진행하였다. 

```
<input class="datepicker" type="text" id="start"/>
<span>~</span>
<input class="datepicker" type="text" id="end"/>
```

```
$("#start").change(function () {
    var minDate = $(this).val();
    var end = $('#end').val();
    if (end == "" || end == null || end == undefined) {
      $('#end').val(minDate); //최소날짜 적용
      $("#start").datepicker("option", "maxDate", minDate); //최대날짜 제한하기
      $("#end").datepicker("option", "minDate", minDate); //최소날짜 제한하기
    } else {
      $("#start").datepicker("option", "maxDate", end); //최대날짜 제한하기
      $("#end").datepicker("option", "minDate", minDate); //최소날짜 제한하기
    }
});
```

```
$("#end").change(function () {
    var maxDate = $(this).val();
    var start = $('#start').val();
    if (start == "" || start == null || start == undefined) {
    $('#start').val(maxDate); //최소날짜 적용
      $("#start").datepicker("option", "maxDate", maxDate); //최대날짜 제한하기
      $("#end").datepicker("option", "minDate", maxDate); //최소날짜 제한하기
    } else {
      $("#start").datepicker("option", "maxDate", maxDate); //최대날짜 제한하기
      $("#end").datepicker("option", "minDate", start); //최소날짜 제한하기
    }
});
```

