# 자료구조의 이해(1)

### Stack이란?

stack은 쌓다라는 의미를 가지고있습니다. 
어렵게 말하자면 제한적으로 접근할 수 있는 나열구조이다. 
쉽게 말하자면 및에가 막힌 상자에 물건을 집어넣는 구조라고 생각하면됩니다. 

##### 스택은 한 쪽 끝에서만 자료를 넣거나 뺄 수 있는 선형 구조로 (LIFO - Last In First Out : 마지막에 들어간게 첫번째로 나온다는 의미) 되어있습니다.

자료를 밀어넣는 것은 'push' - 삽입
자료를 꺼내는 것은 'pop' - 삭제
자료를 읽는것은 'peek' 입니다.  - 읽기

모든 자료구조는 삽입,삭제,읽기 를 기본으로 가진다는 사실을 기억하세요 

그럼 스택은 언제 쓰일까요?
브라우저의 뒤로가기 버튼도 스택으로 구현된 method중 하나입니다. 

인터넷 서핑을 하다가 뒤로가기 버튼을 누르면 가장 나중에 본 페이지가 나오게됩니다. 
바로 LIFO 구조입니다. 

![img](https://t1.daumcdn.net/cfile/tistory/2679DF3358881D3934)



위 그림은 스택의 사용법을 설명해줍니다. 

#### Stack의 사용법

```
public static void main(String[] args) {
    Stack<Integer> stack = new Stack<>();

    //stack 데이터 추가
    stack.push("1");
    stack.push("2");
    stack.push("3");
    
    //stack크기 출력
    stack.size();
    
    //stack이 비어있는지 check (비어있으면 true 반혼)
    stack.empty();
    
    //stack에 1이 있는지 check (맞으면 true 반환)
    stack.contains(1);
    
    //stack의 가장 상단의 값 출력 
    stack.peek();
    
    //stack값 제거 (가장 나중에 들어간 데이터 3이 제거)
    stack.pop();
    
    //stack 전체값 제거(초기화)
    stack.clear();
   
}
```