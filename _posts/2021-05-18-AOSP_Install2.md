---
title: "AOSP 설치 방법"
categories:
  - AOSP
tags:
  - AOSP
  - Android
---

<br>

언젠간 다시 AOSP 설치가 필요한 날을 대비하여 작성하는 포스트입니다.
필요한 사람에게 도움이 되었으면 합니다.

<br>

###### 해당 글은 다음 사이트의 내용을 정리하였습니다.
* android 공식 사이트<<https://source.android.com/setup/start?hl=ko>>
* 제가 처음 참고한 블로그<<https://programist.tistory.com/>>

<br>
해당 포스트는 AOSP를 build하고, 실제 기기에 이미지를 올리는 방법을 설명하려고 합니다.
<br>

# 시작 전 확인사항
1. AOSP 코드를 다운받았는지? (이전 포스트를 참고하기 바랍니다.)
2. HDD or SSD에 저장 공간이 남아있는지? (필자생각: build하면서 용량이 커졌다 작아진다.)
3. 시간이 많이 있는지? (필자생각: 처음 build는 2~3시간 정도 소모된다...)

<br>

# 현재 상태(AOSP코드 다운 완료)
![cli환경 ls명령어 입력한 상태](/assets/images/post2/image1.PNG)
아마 AOSP가 모두 다운되면 해당 디렉토리에 위와 같은 파일들이 다운된다.
build를 진행하기 전에 실제 기기에 필요한 `드라이버 .sh파일`이 필요하다.

<br>

# 드라이버 파일 다운로드 방법
자신 AOSP에 맞는 드라이버를 찾기 위해 아래 사진에 나온 표에서 다운받은 버전의 가장 앞 열(빌드)를 알아야한다. <<https://source.android.com/setup/start/build-numbers?hl=ko>>
![aosp빌드버전 확인](/assets/images/post2/image2.PNG)

<br>

위 내용을 확인하면 <https://developers.google.com/android/drivers> 해당 사이트로 이동하여 방금 확인한 빌드와 같은 드라이버 파일을 찾으면 된다.
![드라이버 파일 확인](/assets/images/post2/image3.PNG)
필자는 11.0.0_r27에 맞는 드라이버를 ctrl+f로 찾았다. 주의사항으로는 본인 핸드폰 기기도 확인을 해야한다.(필자는 pixel 4xl를 사용하여 실험을 진행하였다.)

확인 후 Download의 두 Link를 다운받는다. 해당 파일은 tar.gz이며, 압축을 해제하면 각 1개 총 2개의 .sh파일을 얻을 수 있다. 해당 .sh파일을 AOSP소스코드가 있는 디렉토리로 이동시키면 된다.
(다운방법 1. wget 명령어 사용, 2. GUI환경으로 다운 후 이동)
(필자는 2번 방법을 사용했다.)

<br>

결과적으로 다음 화면처럼 된다.
![터미널 화면](/assets/images/post2/image4.PNG)

# 드라이버 파일 실행

다운 받은 .sh파일은 다음과 같이 실행한다.
```
./extract-google_devices-coral.sh
./extract-qcom-coral.sh
```
실행하면 다음과 같은 화면이 출력된다.
![터미널 화면](/assets/images/post2/image5.PNG)
엔터를 누른다.
![터미널 화면](/assets/images/post2/image6.PNG)
라이센스를 다 읽으면 or "Q key"로 종료가 가능하다.
![터미널 화면](/assets/images/post2/image7.PNG)
I ACCEPT를 입력하면 드라이브에 필요한 파일들이 설정된다.
두 .sh파일 모두 동일 방식을 진행한다.

다음으로 Android기기의 빌드 환경을 설정한다.
```
source build/envsetup.sh
```
![터미널 화면](/assets/images/post2/image8.PNG)

빌드 타겟을 설정한다.
```
lunch
```
![터미널 화면](/assets/images/post2/image9.PNG)
자신에게 맞는 빌드 타켓의 숫자를 입력한다.
필자는 19번 aosp_coral_userdebug를 선택한다.
만약 잘 모르겠으면 aosp_"자신의버전"_userdebug를 추천한다.
자신의 버전을 모른다면, 드라이브 .sh파일 이름에 나와있다.
![터미널 화면](/assets/images/post2/image10.PNG)

# 빌드
```
make
```
위 명령어를 입력하면 빌드가 진행된다.
![터미널 화면](/assets/images/post2/image11.PNG)
빌드과 완료된 모습이다.
(처음 빌드는 2시간 이상 걸린다. 에러가 발생하는 경우도 많이 있다. 필자가 격은 에러는 open jdk버전 에러, 메모리 부족 에러, 하드 디스크 부족 에러 등 있었다.)
(처음 빌드 이후로는 빨리 빌드된다. 그나마....)

# 기기에 이미지 올리기

1. 해당 명령어 이전에 기기에 oem을 unlocking해야한다.
스마트폰에 setting 애플리케이션에서 개발자 모드를 활성화 시키고,
개발자 옵션에 `oem unlocking`을 찾으면 쉽게 찾을 수 있다.
(1회만 진행하면 된다.)

<br>

2. bootloader 진입
```
adb reboot bootloader
```
위 명령어를 입력하기 전에 스마트폰을 해당 linux컴퓨터와 연결해야한다.(usb)

<br>

3. bootloader화면에 lock이라고 표시된다면
```
fastboot flashing unlock
```
위 명령어를 통해 unlock할 수 있다.(1회만 진행하면 된다.)

4. 이미지 올리기
```
fastboot flashall
```
위 명령어를 입력하면 build한 이미지가 스마트폰에 올라간다.
![터미널 화면](/assets/images/post2/image12.PNG)
다음과 같이 진행된다.


# 끝

이것으로 앞으로 
코드 수정 -> build -> adb reboot bootloader -> fastboot flashall만 진행하면 된다.
(fastboot flashall에 여러가지 옵션을 추가 할 수 있다.)

또한 필자는 완벽하게 알지는 못하지만 많은 도움이 되었으면 한다.
그리고 안드로이드 버전이 빠르게 업데이트 되면서, 가끔 변경되는 점도 많이 있다.
맨 위에 참고한 사이트를 참고해주었으면 한다.

이 글을 봐주셔서 감사합니다.











