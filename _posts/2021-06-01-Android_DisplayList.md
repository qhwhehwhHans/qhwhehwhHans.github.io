---
title: "이게 DisplayList?"
categories:
  - AOSP
tags:
  - AOSP
  - Android
---

<br>

진짜 DisplayList를 보고 싶으신 분들을 위해 작성하는 포스트입니다.
필요한 사람에게 도움이 되었으면 합니다.

<br>

###### 해당 글은 다음 사이트의 내용을 참고하였습니다. 
* 2018 Google I/O <https://www.youtube.com/watch?v=zdQRIYOST64&t=776s>

<br>

# DisplayList란?
DisplayList의 정의는 "an encapsulation of the rendering commands for a view"이다.
번역하면 "뷰에 대한 렌더링(그리기) 명령의 캡슐화"<br>
예를들어 어디 위치에 초록색 원, 어떤 부모 view 안에서 얼만큼 떨어진 위치에 글자 등...<br>
화면 그리기에 필요한 데이터를 모아둔 객체이다.<br>
DisplayList를 알기 위해 결국 android 화면 구성 단위인 view에 대해서 알아야 한다. 사실 view를 이야기하면 끝도 없으니, view관리 방식(ViewTree)에 대해서만 잠깐 알아봅시다.

<br>

## ViewTree
android는 view를 tree형식으로 관리하고 있다.(아마 모두 아시겠죠...ㅎ) 예를 들어
![뷰예시](/assets/images/post4/image1.PNG)
![뷰트리예시](/assets/images/post4/image2.PNG)
위 이미지처럼 view와 viewGroup들의 상하관계를 논리적으로 표현한게 viewTree라고 생각합니다. (잡담: viewGroup도 view 클래스를 상속받아 구현되었으니, 디자인패턴 중 컴포지트 패턴이 떠오릅니다.) 결국 view들이 화면에 그려지기 위해서 DisplayList가 필요한데, 그럼 그리기 요청은 누가 할까? 그 함수는 onDraw라는 함수이다.

<br>
더욱 짜임새 있게 설명하려면, vSync와 view lifecycle 등 배경지식이 필요하지만, 최대한 DisplayList에 대해서 설명하겠습니다. 
<br>

## onDraw
View.java에 onDraw함수은 아래와 같이 코드가 작성되어 있다.
![onDraw함수](/assets/images/post4/image3.PNG)
17783번 줄에 주석처럼, view를 상속받은 자식 view는 onDraw함수를 재정의해야한다. 다시 말해 자신이 어떻게 그려져야 하는지 정의해야하고, 그 정의는 인자 canvas에 api를 사용해서 정의하면 된다. 예를 보자
![imageView](/assets/images/post4/image4.PNG)
위 사진은 imageView의 onDraw함수이다. 표시한 부분은 imageView객체의 drawable객체를 첫 번째 인자인 canvas에 draw하는 모습을 보여준다. 다른 view들도 위와 같은 방법으로 canvas에 drawing api를 호출한다. 사실 여기서 사용되는 RecordingCanvas객체로서(android 11버전은 BaseRecordingCanvas클래스이다), 호출한 api들은 기록하여 DisplayList객체에 데이터로 남는다. 
* RecordingCanvas.java <http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/view/RecordingCanvas.java>

## DisplayList
2개의 자료를 제공하려고 합니다. <br>
첫 번째는 android 11버전을 사용하여 얻은 DisplayList<br>
두 번째는 android Pie버전을 사용하여 얻은 DisplayList

#### android 11버전
![DisplayList예시1](/assets/images/post4/image7.PNG)
오른쪽 log가 DisplayList인데, 이전 석사 연구하면서 봤던 DisplayList형식과 달라서 매우 당황했다. (빨리 다시 알아보고 수정하겠습니다.)
#### android Pie버전
* 개인 github 자료 <https://github.com/qhwhehwhHans/portfolio/blob/main/%ED%8F%AC%ED%8A%B8%ED%8F%B4%EB%A6%AC%EC%98%A4(%EB%8C%80%ED%95%99%EC%9B%90%EC%97%B0%EA%B5%AC)/%EA%B7%B8%EC%99%B8%EC%97%B0%EA%B5%AC/android(displaylist)%EC%98%88%EC%8B%9C.pdf>
위 자료가 석사 연구할 때 한번 정리한 자료이다. 아마 이게 더 보기 좋고, 이해하기 좋은 자료같습니다. android 11버전 수정 전까지는 이 자료를 보시는게 편할거 같습니다.
(자료에 설명도 있습니다.^^)
#### DisplayList 출력 방법
절대적인 방법이 아닌, 제가 사용하는 단순한 방법입니다.
![DisplayList출력코드](/assets/images/post4/image8.PNG)
* CanvasContext.cpp <http://androidxref.com/9.0.0_r3/xref/frameworks/base/libs/hwui/renderthread/CanvasContext.cpp>
CanvasContext.cpp파일 중 draw함수에서 mRenderNodes객체의 output()함수를 호출하면, DisplayList가 Log로 출력된다. (CanvasContext의 draw함수는 rendering에 직접 관여하는 함수라 16m/s마다 log가 찍히면 AOSP가 많이 느려진다. 가끔 확인용으로 빨리 사용하고 지운다.)

## 팁: viewTree보는 법 
android studio에 tool탭에서 layout Inspector라는 기능을 제공한다.
![레이아웃인스펙터](/assets/images/post4/image5.PNG)
해당 기능을 클릭하면 viewTree를 예쁘게 볼 수 있다.
![레이아웃인스펙터디테일](/assets/images/post4/image6.PNG)
android분석을 하면서 레이아웃 분석을 위해 사용했는데, 유용한 기능같다. 참고로 삼성 스마트폰과, AVD는 android studio에서 올린 앱만 가능한걸로 알고있다. 하지만!! AOSP는 실제 스토어에 올라온 어플도 viewTree를 볼 수 있다.(매우 유용) 
* layout Inspector의 더욱 자세한 사용법 <https://developer.android.com/studio/debug/layout-inspector.html>

#끝

p.s 여러가지 배경지식을 건너 뛰어서 매우 엉성한 글이 되었습니다. 배경지식 하나 하나씩 포스팅을 하면서 모든 내용을 정리한 글로 다시 작성하겠습니다.

p.s 석사 연구를 android Pie버전으로 해서, 이 글의 어느정도 부분은 최신버전과 다릅니다. 그 부분 생각해주시고, 저도 이 글을 쓰면서 android 11버전을 사용하는데, 많이 달라서 매우 곤란하고 있습니다.