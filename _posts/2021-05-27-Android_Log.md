---
title: "AOSP framework Log 출력하기"
categories:
  - AOSP
tags:
  - AOSP
  - Android
---

<br>

언제 Log가 다시 필요한 날을 대비하여 작성하는 포스트입니다.
필요한 사람에게 도움이 되었으면 합니다.

<br>

###### 해당 글은 다음 사이트의 내용을 참고하였습니다. 
* android 공식 사이트 <https://developer.android.com/studio/debug/am-logcat?hl=ko>
* 참고한 블로그 <https://thdev.net/408>


<br>
해당 포스트는 AOSP에서 특히 frameworks 디렉토리 하위에서 log를 출력하는 방법입니다.
<br>

# java파일에서 Log 출력
- android 개발 할 때 사용하는 Log방식을 그대로 사용하면 됩니다.
- ```import android.util.Log;``` 를 추가하면 됩니다.
```
Log.e(TAG,MESSAGE) //ERROR
Log.w(TAG,MESSAGE) //WARN
Log.i(TAG,MESSAGE) //INFO
Log.d(TAG,MESSAGE) //DEBUG
Log.v(TAG,MESSAGE) //VERBOSE
ex) Log.d("string","string") //예시
```
- TAG와 MESSAGE로 구분된다. 둘 다 "String"자료형을 사용한다.
- 더욱 자세한 정보는 공식사이트에서 참고 바랍니다.<https://developer.android.com/studio/debug/am-logcat?hl=ko>

<br>

# cpp파일에서 Log 출력
1. .mk 파일에 아래 내용을 추가 ```LOCAL_LDLIBS := -llog```
![이미지](/assets/images/post3/image1.PNG)
혹시 의미가 궁금하시면 참고 부탁드립니다.
<br>
2. head 추가 ```#include <android/log.h>```를 추가
실제 파일의 위치는 system/core/include/android/log.h 입니다.
소스코드 <http://androidxref.com/9.0.0_r3/xref/system/core/include/android/log.h>
혹시 궁금하시면 참고 부탁드립니다.
<br>
3. LOG_TAG 정의하기 ```#define LOG_TAG "TAG"```
"TAG" 위치에 원하는 tag를 입력하시면 됩니다. 예시)```#define LOG_TAG "HANS"```
<br>
4. LOG 정의하기 
```
#define  ALOGUNK(...)  __android_log_print(ANDROID_LOG_UNKNOWN,LOG_TAG,__VA_ARGS__)
#define  ALOGDEF(...)  __android_log_print(ANDROID_LOG_DEFAULT,LOG_TAG,__VA_ARGS__)
#define  ALOGV(...)  __android_log_print(ANDROID_LOG_VERBOSE,LOG_TAG,__VA_ARGS__)
#define  ALOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,LOG_TAG,__VA_ARGS__)
#define  ALOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)
#define  ALOGW(...)  __android_log_print(ANDROID_LOG_WARN,LOG_TAG,__VA_ARGS__)
#define  ALOGE(...)  __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)
#define  ALOGF(...)  __android_log_print(ANDROID_FATAL_ERROR,LOG_TAG,__VA_ARGS__)
#define  ALOGS(...)  __android_log_print(ANDROID_SILENT_ERROR,LOG_TAG,__VA_ARGS__)
```
- 우선 LOG가 엄청 많은데, 사용하시는 LOG만 정의하시면 됩니다.
- 앞에 A를 붙이는 이유는 뒤에서 다시 설명하겠지만, 기존 AOSP에서 사용하는 LOG의 이름입니다.
-__android_log_print 함수는  ```android/log.h```파일에 있습니다.
![이미지](/assets/images/post3/image2.PNG)
(사진은 참고하면 좋습니다?)
<br>
- __android_log_print의 첫번째 인자는 예상하시겠지만, LOG의 우선순위를 나타냅니다. 참고로 ```android/log.h```에 정의되어 있습니다.
![이미지](/assets/images/post3/image3.PNG)
(사진은 참고하면 좋습니다?)
<br>
- 두번째 인자는 LOG_TAG라고 3번에 정의한 LOG_TAG가 사용됩니다.
- 세번째 인자"__VA_ARGS__"는 "가변인수 매크로"라고 합니다. ALOGD(...)에서 ...에 대응되는 변수입니다.(결과적으로 앞 두 인자는 고정이고, 세번째 인자만 사용합니다.)
<br>

5. 한 눈에 다시 보기(결론!)
- 선언
![이미지](/assets/images/post3/image4.PNG)
위에 설명 그대로 작성하면 된다!
<br>
- 사용방법
![이미지](/assets/images/post3/image5.PNG)
c언어의 printf처럼 format이 된다.!
<br>

6. 발생가능한 에러
![이미지](/assets/images/post3/image6.PNG)
이 에러는 이미 LOG_TAG가 정의되어 있다는 에러이다. 즉 include한 파일 중 이미 LOG를 정의한 파일이 있다는 말이다. 이런 상황에서는 굳이 LOG를 정의하지 말고 그냥 LOG를 사용하면 된다. 쉽게 말해 #include, #define한거 지우고, ALOGI만 사용하면 된다.
```ALOGI("Hello World") ALOGD("Hello World") ALOGV("Hello World")```
근데 왜 하필 이름이 ALOGI, ALOGD, ALOGV일까? 위에서 설명했지만, AOSP에서는 원래 ALOG를 접두사로 사용하는 LOG방식을 사용한다. (그래서 LOG는 정의해도 ALOG접두사를 추천함. 개인적으로)

# 끝











