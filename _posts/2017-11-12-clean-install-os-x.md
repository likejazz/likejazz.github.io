---
layout: post
title: macOS 설치 프로그램 정리
tags: Productivity
last_modified_at: 2025/10/17 18:43:33
last_modified_history:
  - 2025/10/17 Windows 설치
  - 2024/03/25 맥북 M2 에어 신규 설치
  - 2023/10/04 iOS 추가
  - 2022/06/28 맥북 M1 프로 추가
  - 2022/06/14 CLI 및 기타 도구 정리
  - 2021/12/18 콘솔 프로그램 정리
  - 2020/12/03 빅서 업데이트
  - 2020/06/23 사용 프로그램 정리
  - 2018/05/29 사용 프로그램 정리
  - 2017/11/12 하이 시에라 설치
  - 2017/01/20 맥북 프로 터치바 셋팅
  - 2016/06/22 최신 내용 반영
  - 2015/11/26 엘 캐피탄 업데이트
  - 2015/05/29 1차 개정
  - 2014/10/23 초안 작성
---

<div class="message">
2006년 PowerBook G4를 시작으로 줄곧 맥을 사용해왔다. 이후 요세미티 출시(Oct 2014)와 함께 맥북을 클린 인스톨 했던 경험을 남겼다. 이후 macOS 신규 버전이 릴리즈 될 때마다 꾸준히 내용을 업데이트했고, 이후에도 꾸준히 사용해 오면서 유용한 프로그램을 계속 정리해나가고 있다.
</div>

- [필수 설정 \& 도구](#필수-설정--도구)
- [인터넷](#인터넷)
- [생산성](#생산성)
- [드라이브](#드라이브)
- [문서](#문서)
- [커뮤니케이션](#커뮤니케이션)
- [멀티미디어](#멀티미디어)
- [CLI](#cli)
- [개발 도구](#개발-도구)
- [자료](#자료)
- [기타](#기타)
- [iOS](#ios)
  - [기본 설정](#기본-설정)
  - [필수 앱](#필수-앱)
  - [안되는 것](#안되는-것)

<img src="https://github.com/user-attachments/assets/ce3c3eca-cff3-42c5-b3b7-c7a3518f32bc" width="30%" style="float: left; margin-right: 5px">
<img src="https://user-images.githubusercontent.com/1250095/58238292-f4a1ac00-7d81-11e9-9762-6cd8b44b6461.jpeg" width="30%" style="float: left; margin-right: 5px">
<img src="/images/2017/touchbar.jpg" width="30%">

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==" style="clear: both">

<img src="https://user-images.githubusercontent.com/1250095/71638614-f7656780-2ca7-11ea-85ce-00c1b7c42369.jpg" width="30%" style="float: left; margin-right: 5px">
<img src="https://user-images.githubusercontent.com/1250095/176212469-1143be9d-a185-400e-8092-c9b91798072e.png" width="30%" style="float: left; margin-right: 5px">

<img src="/images/2024/IMG_4542.jpg" width="30%">

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==" style="clear: both">

<small>
2016년 6월, 2017년 1월, 2019년 5월 맥북 프로 터치바  
2020년 1월 맥북 프로 터치바, 2022년 6월 M1 맥북 프로 14인치, 2024년 3월 M2 맥북 에어
</small>

윈도우를 설치한 경험은 [Windows 설치 프로그램 정리](/windows) 참고

## 필수 설정 & 도구
- 언어 설정은 영어를 기본으로 사용한다. 오랫동안 영어로 사용해와서 한글보다 더 익숙하다.
- `Caps Lock` 키가 한/영 변환 디폴트다. 이외에 기존처럼 `⌘ + 스페이스`도 함께 사용한다.
```
Keyboard > Keyboard Shortcuts... > Input Sources > Select the previous input source 변경
                                     Modifier Keys > Caps Lock key 변경
                                     Spotlight > Show Spotlight search ⌥⌘Space 설정
                                     Spotlight > Show Finder search window 해제
```
  - Modifier Keys에서는 키보드 타입이 맞는지 확인 필요
- 각종 도구와 컴파일러 등 모든 개발 관련 도구는 [**Homebrew**](https://github.com/Homebrew/brew/releases)를 통해 설치한다. pkg로 설치하고 `eval "$(/opt/homebrew/bin/brew shellenv)"`를 나중에 oh my zsh 설치 후 `.zshrc`에 두어 링크를 건다.
  - 버전 관리를 위해 앱은 가능한 앱스토어에서 설치한다. **mas**가 이를 CLI에서 관리해준다. `$ brew install mas`
- 터미널은 [**iTerm2**](https://iterm2.com/downloads.html)를 사용한다. `$ brew install iterm2`
  - [**Oh My Zsh**](https://ohmyz.sh/)를 설치했다.
  - Preferences > Profiles > Colors에서 Blue/Normal의 색상이 눈이 아프므로 조정한다.
- 메인 브라우저는 [**Chrome**](https://www.google.com/chrome/)을 사용한다. `$ brew install google-chrome` 보조 브라우저로 **Firefox**도 함께 사용한다. `$ brew install firefox`
- ~~키보드는 MX Mechanical Mini for Mac, MX Anywhere 3 for Mac을 사용하고~~ [**Logi Options+**](https://www.logitech.com/en-us/software/logi-options-plus.html)를 설치하고 셋팅했다. `$ brew install logi-options-plus`
- 간단한 메모 용도로는 여전히 **[Sublime Text 4](http://likejazz.tumblr.com/post/102824813705/sublime-text)**를 사용한다. 메모 용도로만 사용하기 때문에 아무런 플러그인도 사용하지 않는다. `$ brew install sublime-text`
- 비밀번호 관리는 온라인의 경우 Chrome에 탑재되어 있는 Google Password Manager를 이용하고, 오프라인은 Bear에 그냥 메모로 기입한다. 이외 WiFi 정보가 애플의 Password Manager에, 일부 사이트가 Firefox의 Passwords에 저장되어 있다.
- **카카오톡** 설명이 필요 없는 국민 메신저 `$ mas install 869223134`
- [**Visual Studio Code**](/wiki/Visual-Studio-Code) 정리 참고
- [**Rectangle**](https://rectangleapp.com/) Magnet(유료), Cinch, Divvy를 쓰다가 정착했다. 무료인데도 불구하고 디테일한 설정이 유료보다 낫다. 특히 1/3, 1/2 순으로 Rotate되는 기능이 좋다. `$ brew install rectangle`
- 런처는 기본 탑재된 Spotlight Search를 이용한다.
- Notification Center는 전혀 이용하지 않는다. 일부러 모든 Widget을 삭제했다.

## 인터넷
- 관심 있는 링크는 **Raindrop.io**에 저장한다. `$ brew install raindropio`
- Wi-Fi의 신호 세기를 파악하는데 **Wi-Fi Explorer Lite**가 유용하다. `$ mas install 1408727408`
- 별도의 안티 바이러스 프로그램을 사용하지 않는 대신 **Little Snitch(유료)**를 사용한다. 기본 옵션인 Silent Mode로 해서 모두 허용해주고 필요시 차단하는 형태로 관리하고 있다. `$ brew install little-snitch`
  - OS 버전업이 되면 네트워크가 안될 때가 있는데, Quit로 종료 처리 후 AppCleaner에서 삭제하고 휴지통을 비운 후 재시작하면 된다.

## 생산성
- 집과 사무실에서는 클램쉘 모드로 사용한다. 외부에서 아이패드는 사용해도 노트북을 굳이 사용하는 경우는 거의 없다.
- **Things(유료)** 2009년부터 정품을 구매하여 여전히 사용 중이다. 버전 업데이트를 하면서 추가 비용을 받았는데 기꺼이 구매했다. 맥과 아이폰을 포기하지 못하는 한 가지 이유를 대라면 Things를 사용할 수 있기 때문이라고 답할 것 같다. Quick Entry 설정은 `⌥⌘N` `$ mas install 904280696`
- [**Bartender(유료)**](https://www.macbartender.com/) 메뉴바를 정리하는 매우 유용한 앱이다. 메이저 버전업이 있을 때마다 꾸준히 추가 구매했다. `$ brew install bartender`
- 번역 도구로 [**DeepL**](https://www.deepl.com/en/macos-app/)를 이용한다. `$ brew install deepl`
- **AppCleaner** Uninstaller가 없는 맥에서는 삭제하는게 찝찝할때가 많다. 이 앱으로 설정까지 찾아서 삭제한다. `$ brew install appcleaner`
- 토렌트 앱은 최근 **Folx**로 변경했다. 윈도우 시절부터 uTorrent를 사용해오다 최근에는 토렌트 자체를 거의 사용하지 않는다. 유료 버전은 QoS 기능을 제공하지만 무료로도 충분하다.
- 프리젠테이션을 할 때 **Screen Cursor**가 유용하다. `$ mas install 1577211880`
- 주소록(전화번호)은 네이버 주소록에 백업한 다음에 Contacts를 iCloud로 모든 기기에 동기화한다.

## 드라이브
- 문서를 공유하고 간단하게 백업할 때 **Dropbox**를 사용한다.
- **iCloud Drive**도 사용한다. 무엇보다 최신 앱들이 iCloud Drive 저장 기능을 제공한다. 하지만 문서 저장 용도와 Find My Mac을 제외한 다른 iCloud 기능은 사용하지 않는다.
- 사진 자료가 무료 용량을 초과하여 **Google One(유료)**을 구독한다. 덤으로 VPN도 사용할 수 있다.

## 문서
- 오랫동안 **Google Drive**에 문서를 만들어왔다. 십 수년간의 기록이 남아 있고 어디서든 확인하고 편집할 수 있는 점이 장점이다. 협업에 유용하기 때문에 공동 작업이 필요한 경우 반드시 사용한다.
- 개인 문서는 **Bear(유료)**에 지속적으로 정리한다. 아이폰, 아이패드 모두 사용하며 동기화를 위해 연간 구독을 이용 중이다. 아이폰을 반드시 사용해야 하는 두 가지 이유를 대라면 Things와 Bear 때문이다. 필수 메모 용도로 사용하며 20~30개 내외로 관리한다. `$ mas install 1091189122`
- 문서의 작성과 보관은 **GitHub Pages**를 이용한다. 지식 저장소로 활용하고 있으며, **MathJax**를 이용한 수식 지원도 좋다. 위키도 좋지만 수식을 지원하지 않기 때문에 페이지를 별도로 사용한다.
  - 개인 문서와 집필 초고 등은 GitHub의 Private Repository를 이용한다.
  - 집필한 원고는 **Firebase Hosting**에 HTML 페이지로 Publish하여 공유한다.
  - ~~프리젠테이션을 위해 **Marp for VS Code**를 이용한다.~~
- **Microsoft Office(유료)**에서 Excel을 주로 이용하고 집필시 Word를 가끔 이용한다. `$ mas install 462058435`
- 간혹 아래아한글 파일을 읽을 일이 있다. **한컴오피스 한글 2014 VP 뷰어** 설치 `$ mas install 416746898`
- 모든 원고는 마크다운으로 초고를 작성하는데, VSCode를 주로 이용하지만 편리한 WYSIWIG을 위해 [**Typora(유료)**](https://typora.io/)와 병행한다. `$ brew install typora`
- pdf, epub 같은 전자책은 **Yomu(유료)**를 이용한다. 유료 결제시 iCloud 동기화가 가능하다. `$ mas install 562211012`

## 커뮤니케이션
- **Slack** 업무용 커뮤니케이션 도구 `$ mas install 803453959`

## 멀티미디어
- Xee<sup>3</sup>가 더 이상 업데이트가 되지 않아 **Pixea**를 발견해서 사용 중이다. `$ mas install 1507782672`
- **Pixelmator Pro(유료)**를 이미지 편집 용도로 사용한다. 아주 전문적인 기능이 필요한게 아니기 때문에 이 정도로 충분하다. `$ mas install 1289583905`
- **IINA** 중국 개발자가 만든 동영상 플레이어. 오픈소스로 진행되고 업데이트가 빨라 웬만한 국산 동영상 플레이어보다 좋다.

## CLI
- 실행 스크립트는 `~/bin`에 두고 활용한다.
- 개발 관련 프로젝트는 `~/workspace` 디렉토리를 만들어 저장한다. 디렉토리 구조는 깃헙 URL 기준으로 만들어 관리하고 있다. 실험은 `~/workspace/sandbox`에서 진행한다.
  - 플러그인은 `plugins=(git fzf)`를 사용한다. `fzf` 플러그인의 경우 `CTRL-R`을 포함한 키 바인딩을 제공한다.
- Node modules(`$ brew install node`) 중에 CLI 용도로 괜찮은게 있다. 대표적으로 time을 대체하는 **gnomon**이 있다.  
```
$ npm install -g gnomon
```
- AWS 서버 목록은 **awless**를 사용한다. 하지만 아쉽게도 더 이상 관리가 되지 않아 `go install`로만 설치가 가능하며 `~/go/bin` PATH를 설정해주어야 한다.
```
$ go install github.com/wallix/awless@latest
```
- 자주 사용하는 brew CLI 패키지는 다음과 같다.
  - `ollama`: 로컬 LLM
  - `llm`: OpenAI, Claude등 호출
  - `the_silver_searcher`: 필수 검색 도구
  - `fzf`: 필수 검색 도구
  - `huggingface-cli`: 모델, 데이터셋 관리
  - `pwgen`: 패스워드 자동 생성기
  - `kubernetes-cli`: kubectl
  - `firebase-cli`: HTML 호스팅 배포
  - `uv`: 파이썬 버전 관리
  - `pipx`: 파이썬 실행 패키지 별도 관리

## 개발 도구
- **Xcode**를 사용하진 않지만 LLVM을 비롯한 각종 개발 도구를 최신 버전으로 업데이트 하기 위해 Command Line Tools for Xcode를 설치한다.  
```
$ xcode-select --install
$ xcode-select --version
xcode-select version 2395.
```
- git 관리는 CLI로 대부분 가능하나 diff 등은 습관적으로 **GitHub Desktop**을 사용한다. `$ brew install github`
- **JetBrains IDE(유료)** [All Products Pack 연간 라이센스를 사용](http://likejazz.tumblr.com/post/133725850005/jetbrains-all-products-pack)하고 있다. IntelliJ 뿐만 아니라 [CLion](http://likejazz.tumblr.com/post/118649049333/clion-1-0), PyCharm, PhpStorm, AppCode, GoLand등을 사용하는 [가장 즐겨쓰는 최고의 IDE](http://likejazz.tumblr.com/post/112670720955/jetbrains-ide)다. 설정은 [JetBrains](/wiki/JetBrains/) 참고. 최근에는 JetBrains Gateway가 원격 개발에 매우 편리하다.
  - **JetBrains ToolBox** IDE를 통합 관리할 수 있는 필수 메뉴바 앱이다. `$ brew install jetbrains-toolbox`
- ~~개발/테스트/서비스로 이어지는 설정과 설치는 항상 고민거리다. **Docker**는 모든 고민을 말끔하게 해결해줬다. 게다가 M1에서도 잘 지원하기 때문에 **Docker Desktop**은 필수다.~~
- 가끔 스니펫은 **[GitHub Gist](https://gist.github.com/likejazz)**에 보관한다. 비공개도 URL이 노출되면 안되기 때문에 별도로 관리한다. 

## 자료
개인 자료의 경우 사진과 비디오는 **Google Photos**를 활용한다. 비디오 중 공개적인건 **YouTube**에, 음악은 **멜론(유료)** 스트리밍을 이용하고, 문서는 **Dropbox**와 **iCloud Drive**에 적절히 나눠 저장하고, 정리가 필요한 문서는 **GitHub** 위키에 마크다운으로 작성한다. 코드는 **GitHub**을 활용한다.

맥북 SSD 크기가 제한적이라 외장 SDD, SanDisk Extreme Portable SSD를 사용하고 있다. 방진, 방수에 500MB/s 속도를 지원한다. Disk Utility에서 APFS (Encrypted) 포맷으로 Attach시 비밀번호를 입력(저장 가능)해야 접근 가능하다. 250G가 69,000원. 2024년 9월 Portable SSD 2TB는 189,000원에 구매. 물론 외장 HDD가 용량이 더 크고, 가격도 저렴하지만 소음과 진동이 있고, 충격에 약해 휴대용으로는 적절치 않다. 2023년 9월 삼성 SSD 1TB를 129,000원에 구매. 삼성은 exFAT인데 인식 속도가 너무 늦다. SanDisk도 인식이 늦는 문제가 있었는데, 다른 케이블로 해결됐다.

## 기타
대부분의 개인 자료는 클라우드에 두다 보니 맥북을 새로 구매하거나 클린 인스톨을 진행해도 별도로 백업이나 복원할게 없다. 동영상처럼 용량이 매우 큰 일부 자료나 여러 가지 아카이빙을 제외하면 더 이상 로컬에는 데이터를 보관하지 않는다. 회사 문서나 개인 자료만 별도로 시간순 월 단위로 관리하는데, 이 또한 한 번도 열어보지 않거나 결국은 손실되곤 한다.

## iOS
### 기본 설정
- iCloud 동기화 해제. 기본으로 모든 기기를 백업하게 되어 있어 용량을 허비하므로 해제. 설정 > 계정 > iCloud Photos, Mail, Passwords & Keychain 모두 해제. iCloud Backup도 해제. **iCloud Drive**만 사용한다.
  - 이외에 Find My Mac, Voice Memos, Bear, Yomu 정도만 동기화 한다.
  - iCloud > Access iCloud Data on the Web 설정도 해제. 보안을 위해 웹에서 액세스하지 않도록 설정한다.
- 패스워드 관리를 Chrome으로 설정. 설정 > Passwords > Passwords Options > Chrome으로 지정.
- 설정 > iMessage, FaceTime 해제. 애플 기기에서만 사용 가능한 기능이므로 모두 해제. 로그인만 하면 활성화 되므로 iPad에서 로그인하지 않도록 주의한다.
- 외부에서 아이패드는 **Logitech Combo Touch** 키보드를 사용한다. 펜슬은 사용하지 않는다.
- Files에서 iCloud를 이용해 **iA Writer(유료)**에서 마크다운을 편집한다. 노트북 보다 더 집중하기 좋다. iCloud를 통해 자동으로 동기화 된다.

### 필수 앱
- **[Things, Bear](/iphone-again/)**
- Microsoft Authenticator(GitHub, Microsoft 365, AWS, LinkedIn)
  - Google Authenticator(구글 2-factor)

아이폰은 이제 생활 필수 앱이 대부분이다 보니 설치된 앱을 모두 나열하면 개인 사생활을 나열하는 느낌이다.

### 안되는 것
- 클린 인스톨을 지향하다 보니 카톡 예전 대화는 복구하지 못했다.