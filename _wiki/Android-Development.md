---
layout: wiki 
title: Android Development
tags:  ["Software Engineering"]
last_modified_at: 2025/06/08 19:28:02
---

<!-- TOC -->

- [Snippets](#snippets)
  - [Launch a activity](#launch-a-activity)
  - [Call from a custom view](#call-from-a-custom-view)
  - [Speech Recognition](#speech-recognition)
  - [SharedPrefrences](#sharedprefrences)
- [Activity Lifecycle](#activity-lifecycle)
- [Fragment](#fragment)
- [Glide](#glide)
- [Room](#room)
- [예전 프로젝트에 대한 빌드](#예전-프로젝트에-대한-빌드)
- [Gradle](#gradle)
- [기타](#기타)
  - [BlueStacks](#bluestacks)
  - [Flutter](#flutter)
- [안드로이드 개발 정리](#안드로이드-개발-정리)

<!-- /TOC -->

안드로이드의 이벤트 코딩 방식은 상당히 verbose하고 가독성이 떨어진다. 몇 가지 자주 사용하는 이벤트에 대해 스니펫 형태로 정의한다.

# Snippets
## Launch a activity
```java
Button btn = findViewById(R.id.button);
btn.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent intent = new Intent(getApplicationContext(), MainActivity.class);
        startActivity(intent);
    }
});
```

## Call from a custom view
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    ...
    OnMyChangeListener listener;
    listener.onChange(value);
    ...

public interface OnMyChangeListener {
    void onChange(int value);
}
...
public class MainActivity extends AppCompatActivity implements OnMyChangeListener{
    ...
    @Override
    public void onChange(int value) {
        ...
```

인터페이스를 구현한 클래스를 자신에게 등록해서 지정된 함수를 이용해 자신의 상태를 전달하는게 GoF의 옵저버 패턴 <sup>Observer Pattern</sup>이다.

## Speech Recognition
```java
Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, "ko-KR");
intent.putExtra(RecognizerIntent.EXTRA_PROMPT, "말씀해주세요.");
startActivityForResult(intent, 50);
...
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    if (requestCode == 50 && resultCode == RESULT_OK) {
        ArrayList<String> results = data.getStringArrayListExtra(RecognizerIntent.EXTRA_RESULTS);
        String result = results.get(0);
        resultView.setText(result);
    }
```

## SharedPrefrences
Device File Explorer &gt; com.google.cloud.android.speech &gt; shared_prefs &gt; com.google.cloud.android.speech_preferences.xml
```
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <boolean name="seat.heater.frontLeft.on" value="true" />
    <string name="seat.heater.frontLeft.level">3</string>
    ...
</map>
```
string과 boolean 두 가지 타입을 지원한다.

# Activity Lifecycle
<img src="https://developer.android.com/guide/components/images/activity_lifecycle.png" width="70%">

# Fragment
Fragment는 다른 뷰와 다르게 액티비티의 생명주기를 그대로 따르는 뷰이다.

# Glide
Glide는 2014년 구글 IO 행사에서 발표된 라이브러리로 원래 Bump 앱에서 내부적으로 이용하던 라이브러리를 구글이 인수하여 공개한 이미지 핸들링 라이브러리

Matisse는 갤러리를 연동하기 위한 라이브러리

# Room
룸은 SQLite 추상화 라이브러리

# 예전 프로젝트에 대한 빌드
Gradle 2.x 프로젝트는 그대로 빌드되지 않는다. 왜냐면, `repositories`에 `jcenter()`만 등록되어 있기 때문이다. 따라서 아래와 같이 수정해야 한다.
```
repositories {
    jcenter()
    maven {
        url 'https://maven.google.com/'
        name 'Google'
    }
}
```
Gradle 5.x에서는 단순히 `google()`을 지정하면 되나 예전 버전은 저렇게 주소를 명시해야 한다. 또는, 오류 화면에서 `Add Google Maven repository and sync project`를 선택 하면 자동으로 설정해준다.

# Gradle
Gradle 버전 업그레이드(6.1 설치) 및 `sourceCompatibility = 11`(LTS 버전)로 올리기 위해 다음 과정을 거쳤다.
```
$ brew list gradle      # 현재 설치 버전 확인
$ brew upgrade gradle   # 업그레이드
```
Gradle은 wrapper 기반 실행이 recommend 이므로,
```
$ gradle wrapper
```
로 wrapper를 설치한다. JDK는 [Amazon Corretto 11 설치](https://aws.amazon.com/corretto/) JAVA_HOME 설정 `export JAVA_HOME=$(/usr/libexec/java_home)` 결국 `sourceCompatibility`는 `8`에서 올리지 못함.

# 기타
## BlueStacks
원래 모바일 게임 emulator에 좀 더 가까우나 안드로이드 개발시 가장 가벼운 가상 환경이라 매우 편리하다. 사실 핸드폰을 매 번 연결하는 것은 적잖이 귀찮다. 반드시 BlueStacks 먼저 띄우고, 이후에 Android Studio를 구동한다. 잘 안될때가 있는데, 순서대로 띄우고 좀 기다리니 된다. flutter 할때 안됐는데, 일반 android native 프로젝트로 먼저 실행해서 배포한 후 다시 해보니 잘 된다. `adb`를 이용한 디버깅도 가능할 것 같은데, 자주 발생하는 문제이므로 명확한 문제와 해결 방법 파악이 필요.

## Flutter
원래 조금 다르지만 당분간은 여기 같이 정리한다. React Native와 경쟁 관계인 구글의 앱 native 개발 환경. 안드로이드 스튜디오에 잘 연동된다. 예전에 react native는 vscode 같은 별도 툴에서 웹 페이지 개발하듯이 했는데, flutter는 동일한 안드로이드 스튜디오에 플러그인을 잘 지원한다. 심지어 Dart 플러그인은 jetbrains에서 직접 만들었다. 반면 react native는 3rd party plugin만 몇 개 보인다. 적어도 android studio를 계속 사용한다면 훨씬 편리한 개발 환경.

Container 태그를 이용해 디자인 구성을 html과 유사한 형태로 기술한다. css 같은 내용도 속성으로 정의한다. 데이터는 별도의 파일에서 모듈처럼 구현했는데, dart의 문법은 기존 C 계열과 큰 차이가 없어 보인다.

# 안드로이드 개발 정리
*참고: 아래 내용은 2014년 여름 기준이며 현재는 개발 도구 및 버전이 변경 되었습니다.*

<img src="https://31.media.tumblr.com/80af5408aa9b6ce288937cde4e54247a/tumblr_inline_n98vm0WqsZ1qzgoac.jpg" />

- 작년에 받아둔 Android Studio는 0.4 Alpha 버전이었는데 이제 Beta로 올라갔고 버전도 0.8이다. 당장 툴을 업그레이드 하는데서 부터 시작했다. 왜 Eclipse ADT가 아닌 Android Studio를 택했냐면 Eclipse의 느리고 무거움, 직관적이지 않음이 싫었고 Ant로 빌드환경을 구성하고 싶지 않았다. 게다가 Java 개발툴로도 IntelliJ를 사용하고 있기도 하다.
- Java니까 기존에 존재하는 Java의 풍부한 라이브러리를 그대로 활용할 수 있다. 탄탄한 Java 생태계의 도움을 받을 수 있다.
- 프로젝트 생성시 다양한 앱 이벤트에 대해 skeleton 코드를 생성해주는건 흡사 MFC로 Win32 프로그램을 만들면 다양한 generated 코드를 보여주는 것 같다. 하지만 난이도는 Visual C++ 보다는 Visual Basic에 더 가깝다. 이는 쉽다고 비아냥 대는게 아니라 그만큼 쉽게 잘 설계됐다는 얘기이기도 하다. 물론 Java에 이미 익숙해서 일수도 있고 Visual Studio보다 Android Studio가 좀 더 편해서 그럴 수도 있다.
- 각종 troubleshooting은 SO에서 해결했다. 예전 Visual C++ 개발할때 codeguru나 codeproject를 참고하곤 했는데 거긴 질의 응답이 아니라 일종의 '개발 문서 & 예제 코드' 사이트다. 그러다보니 업데이트 주기도 늦고 문서도 풍부하지 않았다. 비슷한 주제의 문서에서 예제 코드를 추출해 한참동안 들여다보곤 했는데 이젠 정확히 일치하는 Stackoverflow 주제에서 정확하게 필요한 답변을 얻을 수 있으니 정말 편하다.
- 개발 정보를 계속 찾다보니 Stackoverflow만 나왔다. 이후에는 아예 검색창을 Stackoverflow로 고정하고 검색하기 시작했다.

<img src="https://31.media.tumblr.com/9946ca41b67a6055ce4ac2bc38f71ae5/tumblr_inline_n98vooKtOI1qzgoac.png" />

- 코드의 5할은 Stackoverflow가 채워줬고 3할은 github에서 차용했다. 내가 만든건 나머지 2할 뿐이다.
- intent-filter 속성 중 앱 실행에 관여하는 LAUNCHER와 deep link를 구성할 수 있는 BROWSABLE은 함께 명시할 수 없다. 그렇게하면 설치 후 ‘열기’ 버튼이 disabled 되고 launcher 아이콘이 보이지 않는다. intent-fileter를 두 번 만드는등 이를 피해갈 수 있는 work-around가 있지만 이렇게 하면 플레이스토어 등록은 당연히 거절될 것이다. 이 모든 경우는 Stackoverflow에 나와있으며 Activity를 2개 만들어 LAUNCHER와 BROWSABLE을 분리하는 것으로 해결했다.
- AVD는 너무 느려서 쓸만한 물건이 못된다. 하지만 디버깅을 위해 어쩔 수 없는 선택이기도 하다. [CPU를 x86으로 설정하고 Host GPU를 사용하는 방법][2]등이 그나마 도움이 된다.
- 디버깅하는데 [adb(Android Debug Bridge)][4]가 큰 도움이 됐다. 그런데 adb가 너무 편리하여 여기에 의존하다보니 testcase를 만들지 않고 계속 debug 관점에서만 접근한 점은 아쉬움으로 남는다.
- 최종 테스트를 위해선 결국 물리적인 기기에 연결해서 테스트가 필요한데 USB Debugging은 번거롭고 특히 PC가 아니라 맥과 연결하는건 잘 되지도 않았다. (LG 옵티머스 G) 그래서 간단한 Gradle 배포 task를 만들었고 아예 Dropbox로 배포해버렸다. 모바일에서 Dropbox에 접속해 apk를 설치했다. 일종의 원격 배포를 구현했고 매우 편했다.
- 이 분야도 개선 속도가 빠르다 보니 1년전 tutorial이 벌써 deprecated에 가깝다. 예를 들어 fragments가 어느새 activity의 필수 요소에 포함된 것등이고 최신 버전으로 생성해보면 PlaceholderFragment를 만들어 쓰는데 비교적 최신 tutorial을 봐야 이에 대한 예제가 나온다. 몇 년전에 사둔 안드로이드 개발 책은 전혀 도움이 되지 않았고(안드로이드의 개발 역사를 되짚어 보겠다면 모르겠으나) 그런면에서 갱신 속도가 빠른 Stackoverflow가 다시 한 번 도움이 됐다.
- 마음 같아선 ‘Android L’로 최신 기능(예를 들어 Material Design)을 마음껏 쓰고 싶었지만 범용성을 무시할 수 없다보니 4.1 Jelly Bean 기준으로 개발했다. API level 16이고 발표일은 2012년 7월이다. 프로젝트 생성할때 보면 현재 77% 기기에 대응된다고 나온다.
- [안드로이드도 API 가이드][3]도 문서화가 잘 되어 있어 큰 도움이 됐다. Visual C++ 개발시 msdn이 도움이 되는 것과 비슷하다. 안드로이드 앱도 Win32 프로그램 처럼 이벤트가 참 많다. 앱 개발시 각 이벤트 대응 설계를 제대로 하지 않으면 나중에 큰 재앙이 올 수 있겠다는 생각을 했다.

[2]: http://stackoverflow.com/questions/1554099/why-is-the-android-emulator-so-slow
[3]: http://developer.android.com/guide/index.html
[4]: http://developer.android.com/tools/help/adb.html