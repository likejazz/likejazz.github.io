---
layout: post
title: ! '도구가 중요합니다'
tags: ["Productivity"]
last_modified_at: 2026/07/27 23:38:20
last_modified_history:
  - 2026/07/25 초안 작성
---

<div class="message">
10년 가까이 구독하던 JetBrains All Products Pack을 해지하고 완전한 CLI 환경으로 돌아왔습니다. Cursor와 Claude Code가 바꿔놓은 개발 방식, 그리고 IDE를 포기하고 완전한 CLI 환경으로 회귀하면서 그 곁을 채운 CLI 도구들에 관한 이야기입니다.
</div>

- [Tools Matter!](#tools-matter)
- [IDE, Cursor, Claude Code](#ide-cursor-claude-code)
- [CLI Productivity](#cli-productivity)

# Tools Matter!

30년 넘게 코딩을 해오면서 제가 가장 중요하게 여겨온 건 늘 '도구'였습니다. 좋은 도구는 좋은 습관을 만들고, 좋은 습관은 높은 생산성으로, 결국 좋은 품질의 코드로 이어진다고 믿었거든요.

<img src="https://github.com/user-attachments/assets/9084218f-3d75-4e89-88b1-5ed6d4a73d18" width="70%">

저는 '언어' 또한 도구의 연장선으로 봅니다. 각 언어는 고유한 생태계와 문화를 지니고 있고 특정 언어를 택한다는 것은 곧 그 문화와 생태계에 참여한다는 뜻이기도 하죠. 언어의 선택 또한 도구와 다르지 않아서 문화와 생태계가 곧 습관과 생산성으로 이어집니다. 그런 면에서 C++의 영향력은 여전히 절대적이라고 생각하고요.

언제나 도구만큼은 비용에 상관없이 가장 좋은 것을 택했습니다. 아무리 비싸도 IDE는 항상 최고의 제품을 골랐죠. [JetBrains All Products Pack을 10년 가까이 구독](https://likejazz.tumblr.com/post/133725850005/jetbrains-all-products-pack)한 것도 그래서였습니다. 그 사이 회사를 두 번이나 옮겼지만, 옮긴 회사마다 매번 비용을 청구해가며 썼을 정도였으니까요. 그런데 올해 4월, 10년을 이어오던 JetBrains All Products Pack 구독을 결국 해지했습니다. 사실 작년부터 이미 망설이던 참이었어요.

Cursor 때문이었죠.

# IDE, Cursor, Claude Code

저는 [Visual Studio Code](/wiki/Visual-Studio-Code/)조차 잘 쓰지 않던 사람입니다. 항상 IDE였죠. 느리고 무겁지만 그래도 코딩을 할 때면 언제나 IDE부터 찾았습니다. CLion, PyCharm, GoLand는 제가 가장 사랑하는 JetBrains의 제품이었고요.

<img src="https://github.com/user-attachments/assets/09620cad-ee84-43c8-bc17-d94c4ee36d31" width="70%">

그런데 작년부터는 Cursor를 실행하는 횟수가 부쩍 늘었습니다. LLM을 활용한 자동완성은 정말 놀라웠죠. 대충 주석 하나 달아놓고 탭 키를 누르면 원하는 코드가 마법같이 생성됐습니다. IDE도 아닌 것이 IDE보다 그 언어를 더 잘 이해하고 있었어요. IDE의 규칙 기반 자동완성으로는 도저히 따라올 수 없는 수준이었습니다. Cursor에 비하면 IDE의 자동완성은 바보처럼 느껴질 정도였죠.

연말 즈음부터 쓰기 시작한 Claude Code는 결정타였습니다. IDE만 고집하던 제가 어느새 터미널에서 CLI로만 작업하는 모습을 발견하고는 문득 놀라곤 했죠. Vim조차 불편해서 잘 쓰지 않았는데 Claude Code 덕분에 CLI에 딱 붙어 있게 됐습니다. 마침 제 작업 대부분이 GPU 관련이라 늘 서버에 접속해야 했는데 로컬에 어렵게 IDE를 구동할 필요 없이 그냥 서버에서 Claude Code를 바로 실행하면 그만이었습니다. 그렇게 하루 종일 CLI 환경에 눌러앉게 됐고, 지금은 컴퓨터 사용 시간의 90% 정도를 SSH에 접속한 채 보내고 있죠.

# CLI Productivity

그러다 보니 이제 모든 도구를 CLI 기반으로 쓰게 됐습니다. 중심에는 당연히 Claude Code가 있고요. IDE는 포기했지만 좋은 에디터는 여전히 필요하기에 Vim 대신 [Helix](https://helix-editor.com/)를 씁니다. 복잡한 플러그인 없이 바로 사용 가능한 매력, '선택→행동'이라는 새로운 패러다임은 그간 Vim에서 느꼈던 불편함을 말끔히 해소해줬어요.

Git의 diff는 기본 도구를 그대로 쓰되, [delta](https://dandavison.github.io/delta/)를 얹어 마치 GitHub에서 보는 것처럼 시각화해서 사용 중입니다. 최준건님의 역작 [fzf](https://junegunn.github.io/fzf/)도 당연히 빼놓을 수 없죠. 아마 우리나라 오픈소스 중 전 세계적으로 가장 유명할 텐데, CTRL+R에 매핑해 히스토리를 빠르게 검색하는 데 쓰고 있죠. [zoxide](https://crates.io/crates/zoxide)와 연결해 CLI 특유의 불편한 디렉토리 브라우징도 스마트하게 극복했습니다.

[GitHub CLI](https://cli.github.com/)도 많이 사용합니다. Git의 부족한 부분을 보완하는 최고의 CLI 도구죠. 물론 가장 자주 사용하는 건 `$ gh auth switch` 같은 GitHub 사용자 전환인데 `ga`로 매핑해뒀고, `$ git log --oneline --graph --all -10`같은 Git 로그 조회도 `gl` 단축키로 매핑해서 사용 중이죠. 요즘은 API 키처럼 보관해야 할 인증 정보가 많은데 이런 환경설정과 alias는 모두 `.profile`에 등록해 한 곳에서 관리합니다. `export DNAROUTER_API_KEY=xxx` 같은 식으로 등록해두고 주기적으로 갱신하면서 쓰죠. API 키는 주로 30일 단위로 갱신하면서 사용 중인데, 매번 갱신하는 건 번거롭지만 대신 한 곳에 통합 관리하고 있습니다.

윈도우, 맥, 리눅스를 모두 쓰지만 통합 환경은 리눅스 한 곳에서만 관리하고 나머지 운영체제는 모두 그쪽으로 접속하는 방식으로 씁니다. 여기서 리눅스는 native는 아니고 윈도우 11의 [WSL2](/wiki/Windows-Subsystem-for-Linux/)를 이용하고 있죠. 개인적으로 WSL2 덕분에 이제 윈도우의 터미널 환경은 맥을 한참 앞서게 됐다고 생각합니다.

터미널은 대개 기본 터미널을 그대로 쓰는데 [데몬처럼 백그라운드 프로세스가 필요할 때는 screen](/screen/)을, 이외에 여러 작업을 동시에 오래 돌려야 할 때는 [tmux](/wiki/Tmux/)를 씁니다. 접속은 윈도우에서는 [Windows Terminal](https://github.com/microsoft/terminal), 맥에서는 [iTerm2](https://iterm2.com/)를 이용하고요.

이렇게 저는 IDE를 포기하고 완전한 CLI 환경으로 회귀했습니다.

1980년대에 처음 컴퓨터를 접했던 그 시절과 똑같은, 검은 화면에 커서만 깜빡이는 환경으로 돌아온 셈입니다. 다만 그때의 커서는 제 명령을 기다렸다면 지금의 커서는 제 생각을 기다립니다.