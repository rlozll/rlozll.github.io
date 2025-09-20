---
layout: post
title:  "suninatas #9 (reversing)"
date:   2025-09-21 04:52:06 +0900
categories: writeup
---

![alt text](image1.png)
우선, 이런 화면이 보인다. Download 버튼을 눌러 exe 파일 하나를 다운받아줬다.

![alt text](image2.png)
이런 exe 파일이 보여서, 이뮤니티 디버거로 열어주도록 하겠다.

우선, 텍스트를 볼 수 있는 창에 가봤는데 아니나 다를까 congratulations라는 글자가 보였다.

바로 이 부분으로 이동하여 BP를 걸어주었다. 

![alt text](image3.png)
흠, 일단 실행해볼까? 

![alt text](image4.png)
BP를 걸어준 곳까지 실행하니까, 메시지창이 떴다. 

딱 봐도, 문제에서 요구하는 문자열을 입력하면 문제를 풀 수 있는 것 같다.

사진을 넣진 않았지만, 그냥 보면 아 이거랑 비교해서 맞으면 그냥 성공이구나~를 알 수 있다. 

![alt text](image5.png)

asm을 다시 보고, 아스키 문자열로 913465라는 숫자가 보여서 입력했더니 문제를 풀 수 있었다.

![alt text](image6.png)
이 913465라는 숫자가 바로 키값인 것 같아서 입력했더니, 최종적으로 clear를 했다.

아주아주 쉬움... 