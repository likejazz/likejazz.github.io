---
layout: wiki 
title: Android Development
last_modified_at: 2021/06/08 13:03:45
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
