---
layout: post
title: macOS 설치 프로그램 정리
tags: Productivity
last_modified_at: 2024/03/11 09:21:56
---

<div class="message">
2006년, PowerBook G4를 시작으로 줄곧 맥을 사용해왔다. 이후 요세미티 출시(Oct 2014)와 함께 맥북에 클린 인스톨을 진행했던 경험의 기록을 남겼다. macOS 신규 버전이 릴리즈 될 때마다 꾸준히 내용을 업데이트했고, 이후에도 사용해 오면서 유용한 도구를 계속 정리해나가고 있다.
</div>

<small id="dates" style="line-height:70%;">
*2023년 10월 4일 iOS 추가*  
*2022년 6월 28일 맥북 M1 프로 추가*  
*2022년 6월 14일 CLI 및 기타 도구 정리*  
*2021년 12월 18일 콘솔 프로그램 정리*  
*2020년 12월 3일 빅서 업데이트*  
*2020년 6월 23일 사용 프로그램 정리*  
*2020년 1월 1일 사용 프로그램 정리*  
*2019년 5월 23일 사용 프로그램 정리*  
*2018년 5월 29일 사용 프로그램 정리*  
*2017년 11월 12일 하이 시에라 설치*  
*2017년 1월 20일 맥북 프로 터치바 셋팅*  
*2016년 6월 22일 최신 내용 반영*  
*2015년 11월 26일 엘 캐피탄 업데이트*  
*2015년 5월 19일 1차 개정*  
*2014년 10월 23일 초안 작성*  
</small>

<!-- TOC -->

- [기본 설정](#기본-설정)
- [인터넷](#인터넷)
- [생산성](#생산성)
- [드라이브](#드라이브)
- [문서](#문서)
- [금융 \& 보안](#금융--보안)
- [커뮤니케이션](#커뮤니케이션)
- [멀티미디어](#멀티미디어)
- [CLI](#cli)
- [개발 도구](#개발-도구)
- [자료](#자료)
- [기타](#기타)
- [iOS](#ios)
  - [기본 설정](#기본-설정-1)
  - [필수 앱](#필수-앱)
  - [안되는 것](#안되는-것)

<!-- /TOC -->

<img src="https://farm8.staticflickr.com/7673/17228888583_65c885c6d5.jpg" width="30%" style="float: left; margin-right: 5px">
<img src="https://c2.staticflickr.com/8/7333/27218763503_715ebb8a06_b.jpg" width="30%" style="float: left; margin-right: 5px">
<img src="https://user-images.githubusercontent.com/1250095/58238292-f4a1ac00-7d81-11e9-9762-6cd8b44b6461.jpeg" width="30%">

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==" style="clear: both">

<img src="/images/2017/touchbar.jpg" width="30%" style="float: left; margin-right: 5px">
<img src="https://user-images.githubusercontent.com/1250095/71638614-f7656780-2ca7-11ea-85ce-00c1b7c42369.jpg" width="30%" style="float: left; margin-right: 5px">
<img src="https://user-images.githubusercontent.com/1250095/176212469-1143be9d-a185-400e-8092-c9b91798072e.png" width="30%">

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==" style="clear: both">

<small>
2015년, 2016년 6월, 2017년 1월 맥북 프로 터치바  
2019년 5월 맥북 프로 터치바, 2020년 1월 맥북 프로 터치바, 2022년 6월 M1 맥북 프로 14인치  
</small>

## 기본 설정
- 언어 설정은 기본을 영어로 사용한다. 오랫동안 영어로 사용해와서 한글보다 더 익숙하다.
- `Caps Lock` 키가 한/영 변환 디폴트다. 그러나 새끼 손가락이 너무 아파서 `⌘ + 스페이스`로 변경. 기존 `Caps Lock` 자리는 컨트롤로 변경. 숏컷이 충돌하는 Spotlight는 키 설정 해제.
- Widgets는 사용하지 않는다. 모두 제거. iOS에만 있어야 할 기능을 억지로 macOS에 배치한 느낌이다.
- Dock에 기본 설치되어 있는 아이콘도 모두 제거. 기본 아이콘은 App Store, System Preferences 뿐이다.
- Night Shift를 사용하며, 스케줄은 10:00 PM to 7:00 AM으로 설정했다.
<img src="https://user-images.githubusercontent.com/1250095/101279230-2ebb8d00-3804-11eb-8a49-d43a494c1efc.png" width="40%">
  - M1 맥북에서는 터치바와 나비 키보드가 모두(!) 사라졌다.
  - 집과 사무실에서는 클램쉘 모드로 사용한다. Magic Trackpad와 Logitech MX Anywhere 3 사용. 외부에서 노트북을 사용하는 경우는 거의 없다.
  - 마우스는 **Logi Options+**를 설치하고 셋팅했다.
- **Bartender 4(유료)** 메뉴바를 정리하는 매우 유용한 앱이다. 메이저 버전업이 있을 때마다 꾸준히 추가 구매했다.
- macOS Sonoma 업데이트 후 인터넷이 안되는 문제가 발생했는데(Private IP ping 조차 되지 않음) Little Snitch의 문제였다. Filter에서 제거하고 해결.

## 인터넷
- 메인 브라우저는 **Chrome**을 사용한다. 보조 브라우저로 **Firefox**도 함께 사용한다.
- 관심 있는 링크는 **Raindrop.io**에 저장한다.
- Wi-Fi의 신호 세기를 파악하는데 **Wi-Fi Explorer Lite**가 유용하다. `$ mas install 1408727408`

## 생산성
- 번역 도구로 **DeepL**를 주로 이용한다.
- 버전 관리를 위해 앱은 가능한 앱스토어에서 설치한다. 앱스토어에서 직접 설치하고 **mas**에서 관리한다. `$ brew install mas`
- **Things(유료)** 2009년부터 정품을 구매하여 여전히 사용 중이다. 버전 업데이트를 하면서 추가 비용을 받았는데 기꺼이 구매했다. 맥과 아이폰을 포기하지 못하는 한 가지 이유를 대라면 Things를 사용할 수 있기 때문이라고 답할 것 같다. `$ mas install 904280696`
- **Rectangle** Magnet(유료), Cinch, Divvy를 쓰다가 정착했다. 무료인데도 불구하고 디테일한 설정이 유료보다 낫다.
- **AppCleaner** Uninstaller가 없는 맥에서는 삭제하는게 찝찝할때가 많다. 이 앱으로 설정까지 찾아서 삭제한다.
- 토렌트 앱은 최근 **Folx**로 변경했다. 윈도우 시절부터 uTorrent를 사용해오다 최근에는 토렌트 자체를 거의 사용하지 않는다. 유료 버전은 QoS 기능을 제공하지만 무료로도 충분하다.
- **Mathpix Snip**에 스크린 캡처를 이용해서 수식의 LaTeX를 찾아낸다.
- 알람 기능은 **Wake Up Time**을 가끔 이용한다. `$ mas install 495945638`
- 모든 프로그램 실행은 **LaunchBar(유료)**를 이용한다.
- 프리젠테이션을 할 때 **Screen Cursor**가 유용하다. `$ mas install 1577211880`
- 주소록(전화번호)은 네이버 주소록에 백업한 다음에 Contacts를 iCloud로 모든 기기에 동기화한다.

## 드라이브
- 문서를 빠르게 이기종 간 공유할 때 **Dropbox**를 사용한다. 하지만 앱은 설치하지 않았다.
- **iCloud Drive**도 사용한다. 무엇보다 최신 앱들이 iCloud Drive 저장 기능을 제공한다. 하지만 문서 저장 용도와 Find My Mac을 제외한 다른 iCloud 기능은 사용하지 않는다.
- 사진 자료가 무료 용량을 초과하여 **Google One**을 이용한다. 덤으로 VPN도 사용할 수 있다.

## 문서
- 오랫동안 **Google Drive**에 문서를 만들어왔다. 십 수년간의 기록이 남아 있고 어디서든 확인하고 편집할 수 있는 점은 큰 장점이다. 협업에 유용하기 때문에 공동 작업이 필요한 경우 반드시 사용한다.
- 개인 문서는 **Bear(유료)**에 지속적으로 정리한다. 아이폰, 아이패드 모두 사용하며 동기화를 위해 연간 구독을 이용 중이다. 아이폰을 반드시 사용해야 하는 두 가지 이유를 대라면 Things와 Bear 때문이다. Bear는 필수 메모 용도로 사용하고, 오래된 메모나 Archives는 GitHub으로 관리한다. `$ mas install 1091189122`
- 문서의 작성과 보관은 **GitHub Pages**를 이용한다. 지식 저장소로 활용하고 있으며, **MathJax**를 이용한 수식 지원도 좋다. 위키도 좋지만 수식 때문에 페이지를 별도로 사용한다.
  - 개인 문서와 집필 초고 등은 GitHub의 Private Repo를 이용한다. 유료였으나 무료가 됐다.
  - 집필시에는 **Firebase Hosting**에 HTML 페이지로 Publish하여 공유한다.
  - 프리젠테이션을 위해 **Marp for VS Code**를 이용한다.
- **Microsoft Office(유료)**에서 Excel을 주로 이용하고 집필시 Word를 가끔 이용한다.
- 간혹 아래아한글 파일을 읽을 일이 있다. **한컴오피스 한글 2014 VP 뷰어** 설치 `$ mas install 416746898`
- 모든 원고는 마크다운으로 초고를 작성하는데, VSCode를 주로 이용하지만 편리한 WYSIWIG을 위해 **Typora(유료)**와 병행한다.
- pdf, epub 같은 전자책은 **Yomu(유료)**를 이용한다. 유료 결제시 iCloud 동기화가 가능하다.

## 금융 & 보안
- 별도의 안티 바이러스 프로그램을 사용하지 않는 대신 **Little Snitch(유료)**를 사용한다. 네트워크 접속을 제한적으로 차단하는 앱인데, 프로세스, 도메인, 네트워크 위치등을 직관적으로 나열해주고 선택적으로 접속할 수 있게 한다. 내 경우 모두 허용으로 하고, 불필요한 클라우드 서비스와 버전 업데이트만 선별하여 차단하고 있다.
- **Bitwarden**을 사용한다. 기존 1Password(유료)에 남겨둔 비밀번호가 아직 남아 있어서 병행해서 사용 중이지만 곧 완전히 갈아탈 예정이다. `$ mas install 1352778147`

## 커뮤니케이션
- **카카오톡** 설명이 필요 없는 국민 메신저 `$ mas install 869223134`
- **Slack** 업무용 커뮤니케이션 도구 `$ mas install 803453959`

## 멀티미디어
- Xee<sup>3</sup>가 더 이상 업데이트가 되지 않아 **Pixea**를 발견해서 사용 중 `$ mas install 1507782672`
- **Pixelmator Pro(유료)**를 이미지 편집 용도로 사용한다. 아주 전문적인 기능이 필요한게 아니기 때문에 이 정도로 충분하다. `$ mas install 1289583905`
- **IINA** 중국 개발자가 만든 동영상 플레이어. 오픈소스로 진행되고 업데이트가 빨라 웬만한 국산 동영상 플레이어보다 좋다.

## CLI
- 개인적인 콘솔 실행 스크립트는 `~/bin`에 두고 활용한다.
- 콘솔 프로그램으로 **iTerm2** 사용. 테마는 Minimal 적용
- 개발 관련 프로젝트는 `~/workspace` 디렉토리를 만들어 저장한다. 디렉토리 구조는 깃헙 URL 기준으로 만들어 관리하고 있다.
- 콘솔은 서버와 동일한 환경을 유지해야 한다는 철학에 따라 계속 bash를 사용해왔으나 이제 서버에 직접 접속해서 작업할 일이 많지 않고 맥 콘솔 그 자체로도 유용한 점이 많아 **oh my zsh**를 사용한다.
  - **fasd** 는 `$ brew install fasd`로 별도 설치. 아래 `.zshrc`의 `plugins` 설정에 fasd를 추가하면 편리한 aliases를 사용할 수 있다.
  - **ag**와 **fzf**도 꼭 필요한 텍스트 검색 도구다.  
```
$ brew install the_silver_searcher fzf
``` 
콘솔의 모든 검색 인터페이스는 fzf로 통일했다.
  - 플러그인은 `plugins=(git aliases fasd command-not-found common-aliases dotenv fig fzf genpass)`를 사용한다. `fzf` 플러그인의 경우 `CTRL-R`을 포함한 키 바인딩을 제공한다.
- **Homebrew** 각종 도구와 컴파일러 등 모든 개발 관련 도구는 brew를 통해 설치한다.
  - `Brewfile`은 `$ brew bundle dump`로 패키지 목록을 생성할 수 있다.
- Node modules 중에 CLI 용도로 괜찮은게 있다. 대표적으로 time을 대체하는 **gnomon**이 있다.  
```
$ npm install -g gnomon
```
- AWS 서버 목록은 **awless**를 사용한다. 하지만 아쉽게도 더 이상 관리가 되지 않아 `go install`로만 설치가 가능하며 `~/go/bin` PATH를 설정해주어야 한다.
```
$ go install github.com/wallix/awless@latest
```
- **Fig** 터미널 환경에서 자동완성을 지원한다. 이외에도 여러 플러그인을 지원하며 터미널 생산성을 높이는데 많은 도움이 된다.  
  - Dotfiles 설정은 번거롭다. 깃헙에 출판도 해보았으나 여전히 관리하기가 쉽지 않다. `~/bin`, `/etc/hosts`, `.vimrc`, `.zshrc` 등.

## 개발 도구
- **Xcode**를 사용하진 않지만 LLVM을 비롯한 각종 개발 도구를 최신 버전으로 업데이트 하기 위해 Command Line Tools for Xcode를 설치한다.  
```
$ xcode-select --install
$ xcode-select --version
xcode-select version 2395.
```
- git 관리는 CLI로 대부분 가능하나 diff 등은 습관적으로 **GitHub Desktop**을 사용한다.
- **JetBrains IDE(유료)** [All Products Pack 연간 라이센스를 사용](http://likejazz.com/post/133725850005/jetbrains-all-products-pack)하고 있다. IntelliJ 뿐만 아니라 [CLion](http://likejazz.com/post/118649049333/clion-1-0), PyCharm, PhpStorm, AppCode, GoLand등을 사용하는 [가장 즐겨쓰는 최고의 IDE](http://likejazz.com/post/112670720955/jetbrains-ide)다. 설정은 [JetBrains](/wiki/JetBrains/) 참고. 최근에는 JetBrains Gateway가 원격 개발에 매우 편리하다.
  - **JetBrains ToolBox** IDE를 통합 관리할 수 있는 필수 메뉴바 앱이다.
- **Visual Studio Code**는 위키를 편집하거나 발표 자료를 작성하고, 코드 리뷰 용도로도 사용하는 메인 편집기다. Settings Sync를 하면(GitHub 계정) 모든 설정과 Extensions로 설치해준다.
  - 위키의 최근 수정 날짜를 갱신하는 **Auto Time Stamp**.
  - JetBrains Tools와 Keymaps를 맞추기 위한 **JetBrains IDE Keymap**.
  - Markdown으로 발표자료를 작성하는 **Marp for VS Code**
- 간단한 메모장 용도로는 여전히 **[Sublime Text 4](http://likejazz.com/post/102824813705/sublime-text)**를 사용한다. 메모 용도로만 사용하기 때문에 아무런 플러그인도 사용하지 않는다.
- 개발/테스트/서비스로 이어지는 설정과 설치는 항상 고민거리다. **Docker**는 모든 고민을 말끔하게 해결해줬다. 게다가 M1에서도 잘 지원하기 때문에 **[Docker Desktop](https://dev.likejazz.com/post/173377603746/docker-for-mac)**은 필수다.
- 가끔 스니펫은 **[GitHub Gist](https://gist.github.com/likejazz)**에 보관한다.

## 자료
개인 자료의 경우 사진과 비디오는 **Google Photos**를 활용하고 그외 블로그와 기타 이미지 호스팅 용도로 **GitHub**의 Issues를 활용한다. 비디오 중 공개적인건 **YouTube**에, 음악은 **YouTube** 스트리밍을 이용하고, 문서는 **Dropbox**와 **iCloud Drive**에 적절히 나눠 저장하고, 오피스 문서는 **OneDrive**에, 정리가 필요한 문서는 **GitHub** 위키에 마크다운으로 작성한다. 공개적인 코드는 공개 **GitHub**에, 회사 업무용은 사내 **GitLab**에 둔다.

맥북 SSD 크기가 제한적이라 외장 SDD, SanDisk Extreme Portable SSD를 사용하고 있다. 방진, 방수에 500MB/s 속도를 지원하고 APFS (Encrypted) 포맷을 사용하여 최초 Attach시 항상 비밀번호를 입력(저장 가능)해야 파일을 열 수 있다. 250G가 69,000원. 물론 외장 HDD가 용량이 더 크고, 가격도 저렴하고 동일하게 APFS도 지원 하지만 소음과 진동이 있고, 충격에 약해 휴대용으로는 적절치 않다. 2023년 9월 삼성 SSD 1TB를 129,000원에 추가 구매.

## 기타
대부분의 개인 자료는 클라우드에 두다 보니 맥북을 새로 구매하거나 클린 인스톨을 진행해도 별도로 백업이나 복원할게 없다. 동영상처럼 용량이 매우 큰 일부 자료나 여러 가지 아카이빙을 제외하면 더 이상 로컬에는 데이터를 보관하지 않는다. 로컬에 데이터를 백업해 둬도 한 번도 열어보지 않거나 결국은 손실되곤 한다.

## iOS
### 기본 설정
- iCloud 동기화 해제. 기본으로 모든 기기를 백업하게 되어 있어 용량을 허비하므로 해제. 설정 > 계정 > iCloud Photos, Mail, Passwords & Keychain 모두 해제. iCloud Backup도 해제. **iCloud Drive**만 사용한다.
  - 이외에 Find My Mac, Voice Memos, Bear, Yomu 정도만 동기화 한다.
  - iCloud > Access iCloud Data on the Web 설정도 해제. 보안을 위해 웹에서 액세스하지 않도록 설정한다.
- 패스워드 관리를 Chrome으로 설정. 설정 > Passwords > Passwords Options > Chrome으로 지정.
- 설정 > iMessage, FaceTime 해제. 애플 기기에서만 사용 가능한 기능이므로 모두 해제. 로그인만 하면 활성화 되므로 iPad에서 로그인하지 않도록 주의한다.
- 외부에서 아이패드는 **Logitech Combo Touch** 키보드를 사용한다. 펜슬은 거의 사용하지 않는다.
- Files에서 **Working Copy(유료)**를 활성화하고 **iA Writer(유료)**에서 마크다운을 편집한다. 노트북 보다 더 집중하기 좋은 최고의 마크다운 편집 환경이다. 작성한 문서는 Working Copy를 이용해 GitHub으로 관리한다.

### 필수 앱
- **[Things, Bear](/iphone-again/)**
- ~~**Calendar**: 구글 계정 추가(메일, 캘린더 연동)~~
- Microsoft Authenticator(GitHub), Google Authenticator(구글 2-factor), Okta Verify

아이폰은 이제 생활과 관련한 앱이 대부분이다 보니 설치앱을 모두 나열하면 개인의 사생활을 모두 엿보는 듯한 느낌이 든다.

### 안되는 것
- 클린 인스톨을 지향하다 보니 카톡 예전 대화는 복구 x