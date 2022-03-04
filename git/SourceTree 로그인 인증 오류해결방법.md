# Git/Github/SourceTree 로그인 인증 오류해결방법

SurceTree에서 Github연동이 안되고 인증 오류가 나서 
아래와 같은 방법으로 해결하였습니다.

기존에 SurceTree에 등록되어있는 계정이있다면 모두 지워줍니다.
그리고 C:\Users\\[사용자이름]\AppData\Local\Atlassian\SourceTree
이 경로로 들어가 passwd 라는 파일을 찾아 지워줍니다. 



#### 1.토큰 생성하기

![4](C:\Users\thdgk\OneDrive\바탕 화면\4.PNG)

Github에서 우측 상단에 있는 프로필을 클릭한 후 아래에 있는 Settings 메뉴에 들어갑니다. 
그리고 하단에 있는 Developer settings를 눌러줍니다. 

![5](C:\Users\thdgk\OneDrive\바탕 화면\5.PNG)

그 다음 Personal access tokens를 눌러 들어가줍니다. 

![6](C:\Users\thdgk\OneDrive\바탕 화면\6.PNG)

Gnenrate new token 을 눌러줍니다. 

![7](C:\Users\thdgk\OneDrive\바탕 화면\7.PNG)

그 다음 Note에는 편하게 적고 (영어권장)
날짜는 마음대로 선택해주세요 
그러고 repo를 클릭해줍니다 (전체선택해주세요)
그다음! 여기서 

![8](C:\Users\thdgk\OneDrive\바탕 화면\8.PNG)

꼭 user도 선택해주세요 이거를 선택안하니깐 소스트리에서 사용자 이름을 넣을때 자꾸 오류가났습니다. 
이렇게 모두 완료하셨다면 하단의 Generate token을 클릭해줍니다. 
그 다음 화면에서 토큰이 생성되었을 겁니다. 그거를 복사하고 다음 화면으로 넘어가면 사라지니 

> 꼭 메모장에 기입해두세요!



#### 2. SourceTree 계정만들기



![9](C:\Users\thdgk\OneDrive\바탕 화면\9.PNG)

이제 소스트리에서 계정을 생성하고 사용자명에는 제가 사용하는 별명을 넣어주고 비밀번호에 아까 복사한 토큰을 넣어주면됩니다! 



#### 3. SSH key 생성하기 

토큰이 아닌 ssh를 이용하여 인증하는 방법도 있습니다. 

![img](https://blog.kakaocdn.net/dn/p7k2g/btrdITf2GxG/jxvc1skI1aMe8bdC6W2Xx1/img.png)

SSH Key 생성 또는 불러오기를 선택합니다.

![0](C:\Users\thdgk\OneDrive\바탕 화면\0.PNG)

Genetate를 눌러 키를 생성해줍니다. 
여기서 저는 가만히 있었는데 마우스를 움직여 주셔야합니다 ㅎㅎㅎ..

그리고 생성된 화면에서 Key를 복사 해주고 
Key passphrase는 키의 비밀번호로 필수는 아닙니다. 

다시 github로 와서

![1](C:\Users\thdgk\OneDrive\바탕 화면\1.PNG)

SSH and GPS key 메뉴로 들어갑니다.

![2](C:\Users\thdgk\OneDrive\바탕 화면\2.PNG)

New SSH key를 선택해주고 

![3](C:\Users\thdgk\OneDrive\바탕 화면\3.PNG)

Key란에 복사한 Key를 붙여줍니다. title은 편하게 적어줍니다. 
완료했으면 Add SSH key를 클릭하여 등록해줍니다.

이젠 다시 putty로 돌아와서
아래 save public key, save private key로 보기 편한곳에 저장해둡니다. 

![img](https://blog.kakaocdn.net/dn/bUErf7/btrdC5PPi6d/gr9KpR5REMRXhE9cVKbB2K/img.png)



비밀번호를 설정 안하셨다면 다음과 같은 경고문이 뜨는데 예를 눌러주시면 비밀번호 없이 생성됩니다. 

![img](https://blog.kakaocdn.net/dn/cgAhCk/btrdFnhTSYA/B2nsa75X62snxwbXXZSnR0/img.png)

여기까지 설정이 완료되었습니다. 
하지만 이대로 놔두면 인증시마다 키를 불러와야하는 불편함이 있기 때문에 자동으로 인증하도록 키를 저장하도록 하겠습니다. 

![img](https://blog.kakaocdn.net/dn/BWMtF/btrdJkxJBK3/26McPnj8QEaGNQZkgv3otk/img.png)

SSH 관리자 실행을 눌러줍니다. 

![img](https://blog.kakaocdn.net/dn/lbDP7/btrdISOZ6xT/qhspAx03fgO3fDRy5nAnCK/img.png)

그리고 작업 표시줄에 모자를 쓰고있는 컴퓨터 모양을 클릭해줍니다.

![img](https://blog.kakaocdn.net/dn/SB6Fp/btrdCydGCXQ/9K6fUXucU8XkPS3KgbGqd1/img.png)

그럼 이렇게 창이 하나 뜨는데 Add key를 눌러서 아까 생성한 ppk 파일을 등록해주면 끝입니다. 
이렇게 SSH를 사용할 경우 기존 https 레퍼지토리 주소가 아닌 SSH 주소를 사용해야합니다.

![img](https://blog.kakaocdn.net/dn/Njlcg/btrdC6urVHc/dX57PnprD2v8USP8K8pBP0/img.png)



여기까지 하셨다면 소스트리와 깃허브를 연동하실수있습니다.