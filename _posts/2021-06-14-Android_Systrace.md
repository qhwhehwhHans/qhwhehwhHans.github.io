---
title: "android 연구 중 알면 좋은 Systrace 사용법"
categories:
  - AOSP
tags:
  - AOSP
  - Android
---

<br>

이 포스트는 매번 Systrace사용법을 까먹는 미래의 저를 위해 작성합니다.
(틀린 부분이 있으면 메일 부탁드립니다. qhwhehwh@naver.com)

<br>

###### 해당 글은 다음 사이트의 내용을 참고하였습니다. 
* 공식 사이트 <https://source.android.com/devices/tech/debug/systrace?hl=ko>
* 참고 블로그 <https://codechacha.com/ko/android-system-trace/>

<br>

# Systrace란?
공식 문서에서는 "systrace는 Android 기기 성능을 분석하기 위한 기본 두구입니다." 라고 나와있습니다. 저는 주로 성능 특히 수행 시간을 관찰하기 위해서 사용하고 있습니다. 사용하기 편리하고, 보기 좋습니다.^^(100% 사용은 못합니다.ㅎㅎ)
![systrace첫화면](/assets/images/post6/image1.PNG)
처음 이 화면은 매우 무섭게 생겼습니다. 저랑 같이 자세히 보시죠!

## 프로세스 확인
![systrace프로세스확인](/assets/images/post6/image2.PNG)
왼쪽 빨간 색으로 표시된 위치는 프로세스를 나타냅니다. 현재 kernel, surfaceFlinger, com.os.noundopaint(제 어플) 등 밑에 더 많은 프로세그가 있으며, surfaceFlinger는 지금 접어둔 상태입니다. (프로세스 이름 앞, > 모양 클릭시 자세히 보기 가능)

## 확대 축소 좌우 이동
![systrace방향키](/assets/images/post6/image3.PNG)
지금 위에 사진은 제 앱(noundopaint)의 renderThread 일부를 확대한 사진입니다. 확대는 w, 축소는 s, 좌,우 이동은 a,d입니다. 마치 게임에서 이동 방향키와 매우 유사합니다. 익숙해 지는데 많이 어렵지 않습니다. 또한 마우스 포인터 위치를 기준으로 확대, 축소 됨으로 마우스도 원하는 위치에 올리셔야 합니다. **확대나 축소할 때 위에 시간 크기를 보시는게 중요합니다. 단순히 막대기가 길다고 오래걸리는게 아닙니다.** (CapsLock으로 소문자 상태를 맞추고 하는걸 추천합니다.)

## 구체적인 시간 보기
![systrace구체적시간](/assets/images/post6/image4.PNG)
해당 이미지는 flush commands라는 trace데이터를 **클릭**한 결과입니다. 단순히 저 박스를 클릭하면 **시간이 조금 걸리고** 밑에 데이터가 나옵니다. 데이터로 start시간, 함수 수행시간(wall duration), CPU Duration은 정확하지 않지만, 개인적으로 wall duration 시간 중 순수하게 해당 함수 시간,(내부에서 다른 함수 수행시간 제외)라고 생각합니다. 나머지 self time은 저도 잘 모르겠습니다;;;;

## 통계 데이터 보기
![systrace통계데이터](/assets/images/post6/image5.PNG)
해당 이미지는 flush commands라는 trace데이터를 **더블클릭**한 결과입니다. 사진에 1번이라고 표시된 곳에서 맨 위 커서 모양을 누르고, 원하는 박스를 더블 클릭하면 **시간이 좀 걸리고** 사진과 같이 통계 데이터가 나옵니다. 이 데이터는 보기 어렵지 않기에 설명을 패스하겠습니다.
<br><br>

**더 구체적인 데이터는 공식 홈페이지 참고 바랍니다.^^**

# 측정하는 방법
이 내용은 참고한 사이트에 있으니 참고 바랍니다.(무단 카피는 나쁩니다.!)
* 참고 블로그 <https://codechacha.com/ko/android-system-trace/>
<br>
먼저 측정하기 위해서는 **systrace.py** 파일과 python 2.x버전(저는 3.x로 하니깐 동작을 안해서;;;)이 필요합니다. 해당 파일은 C:\Users\사용자명\AppData\Local\Android\Sdk\platform-tools\systrace에 있습니다.
저 같은 경우 cmd에서 해당 디렉토리로 이동을 합니다.
![cmd창](/assets/images/post6/image6.PNG)
그리고 다음과 같은 명령어를 입력합니다 
``` python systrace.py -o mytrace.html -t 5 sched freq idle am wm gfx view input res ```
![cmd창](/assets/images/post6/image7.PNG)
명령어를 입력하면 starting traceing이라는 문자가 cmd에 나오면 측정하려는 행동을 취하면 됩니다.(ex 스크롤링, 앱 실행 등) 측정하려는 시간이 종료되면 결과가 systrace.py파일 위치에 저장됩니다.
![cmd창](/assets/images/post6/image8.PNG)
이제 해당 파일을 크롬으로 열어 고민하시면 됩니다.!!

# 끝
다음에는 자신이 측정하고 싶은 함수를 traceing되게 코드를 추가하는 방법에 대해서 설명하겠습니다.