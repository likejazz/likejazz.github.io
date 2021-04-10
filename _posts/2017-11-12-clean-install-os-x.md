---
layout: post
title: macOS 설치 프로그램 정리
tags: Productivity
---

<div class="message">
이 글은 원래 요세미티 출시와 함께 맥북에 클린 인스톨을 했던 경험의 기록이다. 그러나, 엘 캐피탄 출시와 함께 업데이트된 내용을 추가했고, 이후 꾸준히 갱신하여 새로운 맥북 프로 터치바에 설치한 기록까지 정리해 본다.
</div>

<small>
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
- [금융 & 보안](#금융--보안)
- [커뮤니케이션](#커뮤니케이션)
- [멀티미디어](#멀티미디어)
- [콘솔, 개발 도구](#콘솔-개발-도구)
- [자료](#자료)
- [기타](#기타)

<!-- /TOC -->

<img src="https://user-images.githubusercontent.com/1250095/101013610-e6a02e80-35a7-11eb-8765-c2d7da794fe2.png" width="45%" style="float: left; margin-right: 5px"><img src="https://user-images.githubusercontent.com/1250095/71638614-f7656780-2ca7-11ea-85ce-00c1b7c42369.jpg" width="45%">[^fn-screen1]

<img src="https://user-images.githubusercontent.com/1250095/58238292-f4a1ac00-7d81-11e9-9762-6cd8b44b6461.jpeg" width="45%" style="float: left; margin-right: 5px"><img src="/images/2017/touchbar.jpg" width="45%">[^fn-screen2]

<img src="https://c2.staticflickr.com/8/7333/27218763503_715ebb8a06_b.jpg" width="45%" style="float: left; margin-right: 5px"><img src="https://farm8.staticflickr.com/7673/17228888583_65c885c6d5.jpg" width="45%">[^fn-screen3]

[^fn-screen1]: 2020년 12월 빅서(Big Sur), 2020년 1월 맥북 프로 터치바  
[^fn-screen2]: 2019년 5월 맥북 프로 터치바, 2017년 1월 맥북 프로 터치바  
[^fn-screen3]: 2016년 6월, 2015년

## 기본 설정
- 키보드 감도는 가장 짧게 조정한다.
- `Caps Lock` 키가 한/영 변환 디폴트가 되었다. 그대로 사용하려 했으나 새끼 손가락이 너무 아프다. 다시 `⌘ + 스페이스`로 변경했다.
- 알림 센터(Notification Center)는 사용하지 않는다. 대시보드도 마찬가지. iOS에만 있어야 할 기능을 억지로 macOS에 배치한 느낌이다.
- 메뉴바 설정에서 배터리 % 표시, 날짜 표시로 변경했다.
- Night Shift를 사용하며, 스케줄은 10:00 PM to 7:00 AM으로 설정했다.
- 터치바 인터페이스는 거의 쓰지 않는다. 나비 키보드와 함께 신형 맥북의 새로운 기능은 모두 실패작이라 생각한다. 맥북 키보드위에 한성 GK888B 키보드를 얹어서 사용한다.  
<img src="https://user-images.githubusercontent.com/1250095/101279230-2ebb8d00-3804-11eb-8a49-d43a494c1efc.png" width="50%">
- **Bartender 3(유료)** 메뉴바를 정리하는 매우 유용한 앱이다. 빅서가 등장하면서 메이저 업데이트를 했고, 추가 비용을 들여 업그레이드 했다.

## 인터넷
- 메인 브라우저는 **Safari**를 사용한다. 크롬보다 훨씬 더 부드럽게 동작한다.
- 관심 있는 링크는 **Pocket**에 저장한다.
- Wi-Fi의 신호 세기를 파악하는데 **Wi-Fi Explorer(유료)** 가 유용하다.
- 서브 브라우저로 **Chrome**, **Firefox Developer Edition**을 사용한다.

## 생산성
- 번역 도구로 **Google Translate**를 이용한다. 최근에는 **[카카오 번역기](https://translate.kakao.com/)**를 주로 이용하며, 카톡 친구로 등록하면 훨씬 더 편리하게 이용할 수 있다.
- 버전 관리를 위해 앱은 가능한 앱스토어에서 설치한다. 예전에는 cask를 사용했지만 지금은 앱스토어와 호환성을 위해 직접 설치하고 **mas**로 확인한다.
- 캘린더 서비스는 업무 용도가 대부분이라 사내 캘린더를 사용한다. Exchange Server로 구현되어 있다.
- **Things(유료)** 2009년에 정품을 구매하여 여전히 사용 중이다. 버전 업데이트를 하면서 추가 비용을 받았는데 기꺼이 구매했다. 아마 맥과 아이폰을 포기하지 못하는 결정적인 이유를 한 가지 대라면 Things를 사용할 수 있기 때문이라고 답할 것 같다.
- **Microsoft To-Do** Things와 병행해서 사용하고 있다. 안드로이드에서 Things가 되지 않기 때문에 어쩔 수 없는 선택이기도 하다. 최근에 폰을 아이폰에서 안드로이드로 변경했고, 더 이상 Things를 사용할 수 없어 불편했는데, To-Do가 빈자리를 채워주고 있다. 물론 메인은 여전히 Things다.
- **Rectangle** Magnet(유료), Cinch, Divvy를 쓰다가 정착했다. 무료인데도 불구하고 유료보다 더 낫다.
- **AppCleaner** Uninstaller가 없는 맥에서는 삭제해도 뭔가 찜찜할때가 많다. 이 앱으로 불필요한 설정까지 찾아서 삭제한다.
- 토렌트 앱은 최근 **Folx**로 변경했다. 윈도우 시절부터 uTorrent를 사용해오다 최근에는 토렌트 자체를 거의 사용하지 않게 됐지만 갈아탔다. 유료 버전은 QoS 기능을 제공하지만 무료로도 충분하여 무료 버전을 사용하고 있다.
- **VMware Fusion(유료)** 비싼 가격으로 Virtual Box로 이동하려 했으나 기존 이미지 파일을 복구하려면 방법이 없다. 윈도우를 다시 설치할 엄두가 나지 않아 어쩔 수 없이 계속 사용하고 있다. 이미 이미지 파일 크기가 59.5G이며, 현재 11.0.3 버전을 사용하고 있다. 최근에는 거의 사용하지 않아서 이 정도면 삭제해도 될 것 같다는 생각이 든다.
  - **Parallels(유료)** 회사 업무용으로 패러럴즈를 택해서 지급 중이라 사용 중이다. 좀 더 편리한 기능이 돋보이지만 VMWare와 비교해 성능상에 큰 차이는 없는 것 같다. Coherence는 매우 유용하게 활용하고 있다.
- 수식은 **Mathpix Snip**에 스크린 캡처를 이용해서 LaTeX를 찾아낸다.

## 드라이브
- **Dropbox** 경쟁 서비스가 많지만 여전히 최고의 클라우드 서비스다. 
  - `~/bin` 콘솔에서 컴맨드로 처리해야할 명령은 이 디렉토리에 두고 Dropbox에 동기화 한다.
- **iCloud Drive**도 사용한다. 무엇보다 최신 앱들이 iCloud Drive 저장 기능을 제공한다. 하지만 문서 저장 용도와 Find My Mac을 제외한 다른 iCloud 기능은 불필요하게 번거로워 사용하지 않는다.
- **Microsoft OneDrive**는 오피스를 위한 전용 저장소로 활용한다.
- 오랫동안 **Google Drive**를 사용해왔다. 무엇보다 그동안의 기록이 남아 있고 어디서든 확인하고 편집할 수 있는 점은 매우 큰 장점이다. 구글 Docs도 훌륭하다.

## 문서
- 이 문서를 호스팅하는 공간은 **GitHub Pages**다. 수년간 최고의 지식 저장소로 활용하고 있으며, **MathJax**를 이용한 수식 지원도 좋다. 깃헙 위키의 경우 수식을 입력할 수 있는 방법이 마땅찮아 모두 깃헙 페이지로 이관했다.
- **Microsoft Office(유료)**는 맥에서도 최고의 업무용 도구다.
- 오랫동안 **Evernote**를 사용해왔다. 그 동안의 기록들은 이제 히스토리 아카이빙의 역할을 하고 있다. 구글 드라이브 또한 비슷한 용도로 활용중이다.
  - 아이패드에서는 **Flexcil(유료)**을 활용하고 있다.
- 프리젠테이션을 위해 KeyNote 부터 Deckset등 여러 앱을 오갔지만 결국 마지막 종착역은 다시 **Microsoft Office 365**의 웹 기반 **PowerPoint**를 사용하고 있다. 웹에서 사용할때는 무료다.

## 금융 & 보안
- 별도의 안티 바이러스 프로그램을 사용하지 않는 대신 **Little Snitch(유료)**를 사용한다. 네트워크 접속을 제한적으로 차단하는 앱인데, 프로세스, 도메인, 네트워크 위치등을 직관적으로 나열해주고 선택적으로 접속할 수 있게 한다. 내 경우 모두 허용으로 하고, 불필요한 클라우드 서비스와 버전 업데이트만 선별하여 차단하고 있다.
- **1Password(유료)** SuperGenPass를 이용하다 비밀번호 자동 생성 만으로는 부족함이 있어 1Password를 구매해 사용 중이다. 서비스 별로 ID가 다른 경우도 있고 주기적으로 비밀번호를 다르게 바꿔야 할 때도 있는데 기억할 필요가 없어 매우 편리하며, Dropbox에 싱크해 사용 중이다. 7 버전이 나오면서 서브스크립션 모델로 변경되었는데, read-only로 사용한다면 추가 구매 없이 계속 사용이 가능하다. 마음 같아선 서브스크립션 해주고 싶지만 이미 무료로 잘 사용 중이라 추가/삭제는 아이패드에서 하고 맥에서는 계속 read-only로 사용한다.
- 회사 VPN으로 **BIG-IP Edge Client**라는걸 사용하고 있다.

## 커뮤니케이션
- **카카오톡** 메신저는 개인 용도 뿐만 아니라 업무용으로도 훌륭한 커뮤니케이션 수단이다.
- **Slack** 업무용 메신저로 활용한다.

## 멀티미디어
- **Xee<sup>3</sup>(유료)** 빠른 속도 뿐만 아니라 편리한 브라우징이 돋보이는 최고의 이미지 뷰어다. 권한 때문에 파일 브라우징이 다소 불편하게 됐다.
- 이미지 편집은 **Pixelmator Pro(유료)**를 사용하고 있다.
- **IINA** Swift로 만든 동영상 플레이어. 중국 개발자가 만들었으며 무척 깔끔하다.
- ~~**Air Video Server HD(유료)** 동영상을 맥에서 아이패드로 스트리밍 할 수 있는 앱이다. 맥 스트리밍 서버를 설치해야 한다.~~ 느려서 더 이상 사용하지 않는다.

## 콘솔, 개발 도구
- **iTerm2** 최고의 터미널 프로그램이다.
- **Xcode**를 거의 사용하진 않지만 LLVM을 비롯한 각종 개발 도구를 최신 버전으로 업데이트 하기 위해 필수적으로 설치한다.
- 개발 관련 프로젝트는 `~/workspace` 디렉토리를 만들어 저장한다. 디렉토리 구조는 깃헙 URL 기준으로 만들어 관리하고 있다.
- 유료 git 클라이언트를 사용하다가 최근에는 깃헙에서 공식 배포하는 **GitHub Desktop**을 사용한다. 무료이면서도 매우 직관적이고 diff를 보기 편하게 보여준다.
- 어느덧 zsh 기반의 **oh my zsh** 가 대세가 됐다. 서버와 동일한 환경을 유지해야 한다는 생각에 계속 bash를 사용해왔으나 이제 서버에 직접 접속해서 작업할 일은 많지 않고 또한 그간 무거워서 사용하지 않았으나 최근 플러그인 구조도 많이 가벼워지고 fish의 도무지 적응 안되는 문법으로 인해 다시 zsh 기반으로 이전했다.
  - **fasd** 는 `brew install fasd`로 별도 설치. 아래 `.zshrc`의 `plugins` 설정에 fasd를 추가하면 편리한 aliases를 사용할 수 있다.
  - **ag**와 **fzf**도 꼭 필요한 텍스트 검색 도구다. `brew install the_silver_searcher` fzf는 `CTRL-R`을 포함한 키 바인딩과 추가 맵핑을 함께 설치했다. `brew install fzf` 이후 환경 설치. 사실상 모든 검색 인터페이스는 fzf로 통일했다.
  - 플러그인은 `plugins=(git history python fasd history-substring-search docker)`를 사용한다.
- **Homebrew** 이제 brew 없는 맥 콘솔은 상상할 수 없다. 각종 도구와 컴파일러등 모든 개발 관련 도구는 brew를 통해 설치한다.
<img width="70%" src="https://user-images.githubusercontent.com/1250095/32696328-ea701530-c7b8-11e7-9501-149a4ce0dd86.png">  
  - `Brewfile`(`brew bundle dump`로 생성)은 아래와 같다.  
```
tap "homebrew/bundle"
tap "homebrew/cask"
tap "homebrew/cask-fonts"
tap "homebrew/core"
tap "homebrew/services"
tap "wallix/awless"
brew "python@3.9"
brew "glib"
brew "nettle"
brew "gnutls"
brew "emacs"
brew "cask"
brew "clisp"
brew "cmake"
brew "ctop"
brew "deno"
brew "exa"
brew "fasd"
brew "fzf"
brew "gd"
brew "gdk-pixbuf"
brew "gdrive"
brew "gh"
brew "git"
brew "glances"
brew "gnu-tar"
brew "go"
brew "gobject-introspection"
brew "gradle"
brew "jasper"
brew "netpbm"
brew "gts"
brew "harfbuzz"
brew "pango"
brew "librsvg"
brew "graphviz"
brew "grpcurl"
brew "helm"
brew "htop"
brew "httpie"
brew "hub"
brew "libheif"
brew "libomp"
brew "imagemagick"
brew "jq"
brew "m-cli"
brew "make"
brew "mas"
brew "maven"
brew "meson"
brew "nasm"
brew "nload"
brew "nmap"
brew "node"
brew "pandoc"
brew "protobuf"
brew "protoc-gen-go"
brew "pyenv"
brew "quickjs"
brew "r"
brew "rbenv"
brew "rust"
brew "speedtest-cli"
brew "telnet"
brew "terraform"
brew "the_silver_searcher"
brew "tldr"
brew "tmux"
brew "tree"
brew "typescript"
brew "watch"
brew "wget"
brew "youtube-dl"
brew "wallix/awless/awless"
cask "anaconda"
cask "font-jetbrains-mono"
mas "com.adriangranados.wifiexplorerlite", id: 1408727408
mas "com.pixelmatorteam.pixelmator.x", id: 1289583905
mas "Evernote", id: 406056744
mas "Hancom Office HWP 2014 VP Viewer", id: 416746898
mas "KakaoTalk", id: 869223134
mas "Keynote", id: 409183694
mas "Messenger", id: 1480068668
mas "Microsoft Excel", id: 462058435
mas "Microsoft PowerPoint", id: 462062816
mas "Microsoft Word", id: 462054704
mas "Mirror HD", id: 413878049
mas "Numbers", id: 409203825
mas "Pages", id: 409201541
mas "Polyglot", id: 1471801525
mas "Things", id: 904280696
mas "Unicorn HTTPS", id: 1475628500
mas "Wake Up Time", id: 495945638
mas "Xcode", id: 497799835
mas "Xee³", id: 639764244
```
- **JetBrains IDE(유료)** [All Products Pack 연간 라이센스를 사용](http://likejazz.com/post/133725850005/jetbrains-all-products-pack)하고 있다. IntelliJ 뿐만 아니라 [CLion](http://likejazz.com/post/118649049333/clion-1-0), PyCharm, PhpStorm, AppCode, GoLand등을 사용하는 [가장 즐겨쓰는 최고의 IDE](http://likejazz.com/post/112670720955/jetbrains-ide)다.
  - **JetBrains ToolBox** 제트브레인의 IDE를 통합 관리할 수 있는 메뉴바 앱이다. `Update all tools automatically`로 설정하고 사용한다.
  - IntelliJ의 테마는 Darcula를 사용한다. `Editor > General > Appearance`에서 **Show whitespaces**와 **Show method separators**는 활성화 한다.
  - IntelliJ의 **Identifier under caret** 기능을 유용하게 사용하는 편이라 기본 Darcula에서 이 색상만 눈에 띄게 지정하고 사용한다.
    - Background를 `FFFF00`로 지정
    - `Identifier under caret (write)`는 `FFFFE0`로 지정
- **Visual Studio Code**는 위키를 편집하거나 코드 리뷰 용도로 사용하는 메인 편집기다. 아래 Extensions를 함께 사용한다.
  - **JetBrains IDE Keymap** 모든 단축키는 혼동되지 않도록 JetBrains IDE를 기준으로 한다.
  - 마크다운을 깃헙 위키 스타일로 미리 보기 위한 **Markdown Preview Enhanced**.
  - 마크다운 TOC 생성을  위한 **Markdown TOC**.
  - 위키의 최근 수정 날짜를 갱신하는 **Auto Time Stamp**.
- 주로 업무와 관련한 프로젝트 개발을 할때는 항상 JetBrains의 IDE를 사용하지만 간단한 메모 용도로는 여전히 **[Sublime Text 3](http://likejazz.com/post/102824813705/sublime-text)**를 사용한다. 메모 용도로만 사용하기 때문에 더 이상 플러그인은 사용하지 않는다.
- dotfiles 설정은 번거롭다. 깃헙에 공개 형태로 출판도 해보았으나 여전히 관리하기가 쉽지 않다. 새 노트북을 셋팅할때 Dropbox에 보관했다가 셋팅했다. `/etc/hosts`, `.vimrc`, `.zshrc` 등.
- **Docker** 개발/테스트/서비스로 이어지는 설정과 설치는 항상 고민거리다. 도커는 그 고민을 말끔하게 해결해줬다. 사실상 OS까지 함께 설치하는 컨테이너 개념은 정말 편하다. 게다가 맥에서도 작년부터 native hypervisor를 지원해서 virtualbox를 사용해야 하는 불편함도 말끔히 해결됐다.
- 스니펫은 [GitHub Gist](https://gist.github.com/likejazz)에 보관한다.

## 자료
개인 자료의 경우 사진과 비디오는 **Google Photos**를 활용하고 그외 블로그와 기타 이미지 호스팅 용도로 **GitHub**의 Issues를 활용한다. 비디오 중 공개적인건 **YouTube**에, 음악은 **YouTube** 스트리밍을 이용하고, 문서는 **Dropbox**와 **iCloud Drive**에 적절히 나눠 저장하고, 오피스 문서는 **OneDrive**에, 정리가 필요한 문서는 **GitHub** 위키에 마크다운으로 작성한다. 공개적인 코드는 공개 **GitHub**에, 회사 업무용은 사내 **GitHub Enterprise**에 둔다.

## 기타
대부분의 개인 자료는 클라우드에 두다 보니 맥북을 새로 구매하거나 클린 인스톨을 진행해도 별도로 백업이나 복원할게 없다. 동영상 처럼 용량이 매우 큰 일부 자료나 여러가지 아카이빙을 제외하면 더 이상 로컬에 데이터를 보관하지 않는다.