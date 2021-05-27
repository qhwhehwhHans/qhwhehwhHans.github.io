---
title: "AOSP 다운로드 방법"
categories:
  - AOSP
tags:
  - AOSP
  - Android
---

<br>

언젠간 다시 AOSP 다운로드가 필요한 날을 대비하여 작성하는 포스트입니다.
필요한 사람에게 도움이 되었으면 합니다.

<br>

###### 해당 글은 다음 사이트의 내용을 참고하여 정리하였습니다.
* android 공식 사이트<https://source.android.com/setup/start?hl=ko>
* 제가 처음 참고한 블로그<https://programist.tistory.com/>

<br>
<br>

# 준비물

1. linux 또는 macOS 운영체제를 사용하는 컴퓨터
2. 500GB or 1TB(추천) 하드디스크 (필자는 500GB 설치하다가 용량 부족? 문제가 발생하여 1TB로 바꾸었다.)
3. 32GB 이상 RAM (최근 16GB로 AOSP 11버전을 build하다가 메모리 부족 문제가 발생하여 32GB로 업그레이드하였다.)

<br>
<br>

# 필수 패키지 설치

우분투 18.04버전 기준으로 아래와 같은 필수 패키지 설치가 필요하다.<br>

```
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
```

그 외 버전 or macOS 경우 참고 <https://source.android.com/setup/build/initializing?hl=ko>

<br>
<br>

# Repo 설치
Repo는 android환경에서 Git을 더 쉽게 사용할 수 있게 해주는 도구입니다. <br>
해당 방법은 Python3를 사용하는 경우입니다.
  1. 홈 디렉토리에 `bin/` 디렉토리가 있고 다음 경로에 포함되어 있는지 확인합니다. <br>
  ```
  mkdir ~/bin
  PATH=~/bin:$PATH
  ```
  2. Repo 런처를 다운로드하고 실행 가능한지 확인합니다. <br>
  ```
  curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
  chmod a+x ~/bin/repo
  ```

개발환경이 Python2를 사용하고 있다면 해당 사이트를 참고 <https://source.android.com/setup/develop#installing-repo>

<br>
<br>

# 작업 디렉토리 생성

  1. 작업 디렉토리를 생성하고 이동합니다.<br>
  ```
  mkdir WORKING_DIRECTORY
  cd WORKING_DIRECTORY
  ```
    * 예시) `mkdir aosp_11.0.0_r27`(필자는 aosp 버전을 디렉토리 명으로 합니다.)

  2. Git을 실명과 이메일 주소로 구성합니다.<br>
  ```
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  ```
    * 예시) 
    `git config --global user.name qhwhehwhHans` 
    `git config --global user.email qhwhehwh@gmail.com`

<br>
<br>

# 다운로드 버전 설정 및 다운로드
  1. 다운로드 버전 설정<br>
  ```
  repo init -u https://android.googlesource.com/platform/manifest -b "BranchName" <br>
  ```
    * 예시) `repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r27` <br>

  2. 다운로드<br>
  ```
  repo sync
  ```
  필자는 보통 다운받는데 2시간 이상 소모됨

<br>

# 다운로드 버전 확인 방법
해당 사이트 참고 <https://source.android.com/setup/start/build-numbers?hl=ko> <br>
지원되는 기기와 원하는 버전을 확인해서 태그(버전)를 확인하면 된다. <br>
표 앞에 있는 빌드는 추후에 실제 기기에 올릴 때 필요한 정보이다.

![안드로이드소스코드및태크](/assets/images/post1/androidSourceCodeAndTag.PNG)