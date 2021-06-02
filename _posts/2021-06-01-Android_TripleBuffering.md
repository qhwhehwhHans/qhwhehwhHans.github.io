---
title: "android에서 사용되는 TripleBuffering을 간단히 설명"
categories:
  - AOSP
tags:
  - AOSP
  - Android
---

<br>

이 포스트는 TripleBuffering을 조금이나마 이해하기 쉽게 설명하려고 작성했습니다.
(틀린 부분이 있으면 메일 부탁드립니다. qhwhehwh@naver.com)

<br>

###### 해당 글은 다음 사이트의 내용을 참고하였습니다. 
* 참고 사이트 <https://www.androidpolice.com/2012/07/12/getting-to-know-android-4-1-part-3-project-butter-how-it-works-and-what-it-added/>

<br>

# TripleBuffering이란?
단순하게 설명하면 android는 **도화지 3장**에 순차적으로 그림(view or 화면ui)을 그리서, 사용자가 볼 수 있는 화면에 출력하는 방식이다. 예를 들어 1번 도화지에 ui를 보여주는 동안 2번 도화지에 다음 ui을 그리고, 2번을 보여주는 동안에는 3번을 그리고, 3번을 보여주는 동안에는 다시 1번에 ui를 그려서 사용자에게 보여준다.
<br><br>
그럼 이런 생각이 든다. 도화지 2개로 충분하지 않나?
<br><br>
물론 DoubleBuffering이라는 도화지 2장의 방식도 존재한다. 하지만 android가 왜 TripleBuffering을 사용했는지 알아보자.
<br>
##### 잠시 용어 정리
vSync: 화면을 그리는 요청이면서, 그려진 화면을 교체하는 신호?라고 우선 생각하자.
<br><br>
## 일반적인 DoubleBuffering 시나리오
![일반적인doubleBuffering](/assets/images/post5/image1.PNG)
<br>
우선 이 사진으로 그림 보는법에 대해서 설명하자(DoubleBuffering상황). 왼쪽부터 보면, A도화지는 화면에 display중이고, B는 cpu, gpu에서 다음 화면이 그려지고 있다.(CPU는 여러 동작이 있지만, DisplayList생성이라고 보면 좋고, GPU는 openGL명령어가 실행된다고 보면 쉽다.) 처음 vSync가 발생하면, display를 B도화지로 변경하고, A도화지에 다음 그림을 그리는 동작을 반복한다. 전혀 문제가 없어보인다.
<br><br>
## DoubleBuffering에서 jank(버벅거림)가 발생하는 시나리오
![JankDoubleBuffering](/assets/images/post5/image2.PNG)
<br>
위 사진은 DoubleBuffering 상황에서 만약 vSync안에 B도화지에 그림을 다 못그리면 어떤 상황이 발생하는지 보여주는 그림이다. 결과적으로 vSync가 발생해도 B도화지로 교체가 안되는 문제가 발생한다. 이때 사용자는 버벅거림, 불편함을 느낄 수 있다. **또한** 위 상황에서 B의 GPU가 vSync 조금 넘어서 완료되어, 나머지 vSync 동안 시간을 버리게 된다. 그리고 다시 A도화지가 그려지는데 위 사진은 극단적으로 또 vSync를 넘는 경우를 보여준다. 개인적으로 A도 다음 vSync를 넘는다고 보장은 못한다. 잘 그릴수도 있다.
<br><br>
## TripleBuffering에서 jank(버벅거림)가 발생하는 시나리오
![JankTripleBuffering](/assets/images/post5/image3.PNG)
<br>
위 사진은 A,B,C도화지를 사용하는 TripleBuffering 시나리오이다. 위 예제처럼 처음 B도화지에 그림을 vSync안에 그리지 못한다면, jank가 발생한다. 하지만 DoubleBuffering과 **차이점은 C도화지를 준비한다는 점이다.** 위에서도 언급했듯이, DoubleBuffering에서는 처음 vSync 이후 시간이 남는다고 했다. 하지만 TripleBuffering은 다음 도화지까지 준비하여, 다음, 다다음 화면까지 jank가 발생하지 않도록 하는 동작을 볼 수 있다.(C의 GPU가 엄청길어지면 jank가 발생 할 수 있다.)
<br><br>
이러한 이유로 android는 사용자에게 좀 더 좋은 화면을 제공하기 위해 TripleBuffering을 사용한다. 위 3개 사진의 출처는 아래 url이다.
[사진 출처] <https://www.androidpolice.com/2012/07/12/getting-to-know-android-4-1-part-3-project-butter-how-it-works-and-what-it-added/>

## Buffer 정보 확인법
많은 사람들이 그래서 buffer가 3개인지 어떻게 확인하는지 궁금할거 같다. 석사 기간 동안 이리저리 굴러다니다가 발견한 방법이 있다.
```
adb shell dumpsys SurfaceFlinger
```
위 명령어를 cmd에 치면 엄청 많은 데이터가 나온다. 그 중
![dumpsys](/assets/images/post5/image4.PNG)
<br>
(**분명 석사 기간에 확인할때는 동일한 이름이 3개씩 나왔다.(1개도 있었지만) 근데 이 글을 작성하면서 android11버전으로 입력했는데 4개씩 나왔다...... 매우 당황스럽다... 추가로 알아본 결과 android11은 quad buffer를 지원하는거 같다.**)
이 결과와 똑같은 출력문을 찾으면 된다. 우선 네비바, 상태바가 4개씩, 어플 이름은 2개이지만 메모리의 크기와 사이즈로 유추하자면 FramebufferSurface도 같다고 볼 수 있다. 그래서 4개, 간혹 4개가 아닌 buffer도 있는데, 사실 무조건 4개가 있어야 하는게 아니다. 최조 생성자에서 개수를 지정할 수도 있는걸로 알고있다.(하단에 네비바, 상단에 상태바도 다른 프로세스라 다른 buffer로 관리된다.)(surfaceView같은 경우 더 많은 buffer를 생성한걸 본적이 있다. 추후에 surfaceView에 대해서 다루고 싶다.)

## QuadBuffer?
![quadBuffer](/assets/images/post5/image5.PNG)
TripleBuffering을 더욱 자세히 이해하기 위해서 RingBuffer을 알아야 하는데.... 갑자기 android11에서 Quad buffer라뇨...... 저도 이제 알았습니다....ㅠㅠ**하지만 구글 공식사이트에서 정보를 찾아보고 정확하게 다시 작성하겠습니다.**

# 고민 중
다음 설명을 Triple롤 설명해야하는지... Quad로 설명해야하는지....