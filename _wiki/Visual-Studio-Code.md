---
layout: wiki 
title: Visual Studio Code
tags: ["Productivity"]
last_modified_at: 2025/10/17 18:45:13
last_modified_history:
  - 2025/10/17 Windows 설치
  - 2025/06/24 이전 버전
---

<!-- TOC -->

- [개인 설정 정리](#개인-설정-정리)
  - [설치 플러그인](#설치-플러그인)
- [Compare Folders](#compare-folders)
- [미니맵 색상 변경](#미니맵-색상-변경)
- [커서가 느려지는 문제](#커서가-느려지는-문제)

<!-- /TOC -->

# 개인 설정 정리
- 설정에서 `search.useIgnoreFiles` 해제
- 설정에서 `search.exclude`에서 `_site` 추가해서 generated html은 검색이 되지 않도록 설정
- WSL2에서 Auto Time Stamp는 `/mnt/c/Users/xxx/AppData/Roaming/Code/User/settings.json`에 설정 필요
```json
{
    "security.allowedUNCHosts": [
        "wsl.localhost"
    ],
    "search.useIgnoreFiles": false,
    "search.exclude": {
        "**/_site": true
    },
    "lpubsppop01.autoTimeStamp.modifiedTimeStart": "last_modified(_at)?: ",
    "lpubsppop01.autoTimeStamp.modifiedTimeEnd": "$",
    "lpubsppop01.autoTimeStamp.modifiedTimeFormat": "YYYY/MM/DD HH:mm:ss"
}
```

## 설치 플러그인
- [**Visual Studio Code**](https://code.visualstudio.com/download)는 위키를 편집하거나 발표 자료를 작성하고, 코드 리뷰 용도로도 사용하는 메인 편집기다. Settings Sync를 하면(GitHub 계정) 모든 설정과 Extensions를 설치해준다. `$ brew install visual-studio-code`
  - 편리한 이용을 위해 [Shell Command](https://code.visualstudio.com/docs/setup/mac#_launching-from-the-command-line)를 설치한다.
  - 위키의 최근 수정 날짜를 갱신하는 **[Auto Time Stamp](https://marketplace.visualstudio.com/items?itemName=lpubsppop01.vscode-auto-timestamp)**.
  - JetBrains Tools와 Keymaps를 맞추기 위한 **[JetBrains IDE Keymap](https://marketplace.visualstudio.com/items?itemName=isudox.vscode-jetbrains-keybindings)**.
  - Markdown TOC를 비롯한 여러 편리한 기능 **[Markdown All in One(Yu Zhang)](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)**
  - ~~Markdown으로 발표자료를 작성하는 **[Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode)**~~
  - 문서의 길이 비율 표시 **[scroll percentage](https://marketplace.visualstudio.com/items?itemName=IdanRudich.scroll-percentage)** 긴 문서를 편집할 때 유용하다.

---

# Compare Folders
Compare Folders(MoshFeu): 첫 번째 upstream, 두 번째 working을 두고 Compare Folders를 진행하면 왼쪽에 최신 버전을 오른쪽으로 반영할 수 있다.

첫 번째가 my folder, 두 번째는 compared folder. 자동으로 순서가 선택되는 형태는 아니고 항상 위에서 아래로 선택되는 구조다. 변경은 Compare Folders 내에서 Swap Sides로 변경 가능

# 미니맵 색상 변경
노란색으로 눈에 잘 띄게 변경하고 항상 슬라이더가 표현되도록 수정한다.

```
Workbench: Color Customizations
Overrides colors from the currently selected color theme.
Edit in settings.json
```
내가 사용중인 `settings.json` 전체는 다음과 같다.

```json
{
    "editor.renderWhitespace": "all",
    "markdown-preview-enhanced.scrollSync": true,
    "markdown-preview-enhanced.enableEmojiSyntax": false,
    "markdown-preview-enhanced.enableExtendedTableSyntax": true,
    "markdown-preview-enhanced.mathRenderingOption": "MathJax",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    "workbench.editorAssociations": {
        "*.ipynb": "jupyter.notebook.ipynb"
    },
    "workbench.colorCustomizations": {
        
        "minimapSlider.activeBackground": "#ffff00b6",
        "minimapSlider.hoverBackground": "#ffff006b",
        "minimapSlider.background": "#ffff002a",
    },
    "editor.minimap.showSlider": "always",
    "window.menuBarVisibility": "classic",
    "lpubsppop01.autoTimeStamp.modifiedTimeStart": "[lL]ast[ -_][mM]odified(?:|[ -_]at): ",
    "editor.fontSize": 14,
}
```

vscode.dev에도 동일하게 settings.json 수정하고, 깃헙 계정으로 `Settings Sync is On`으로 동기화해서 사용한다.

# 커서가 느려지는 문제
어느날 부터 키보드 움직임이 느려졌는데, 편집 중인 파일 사이즈가 너무 커서 그런가 했는데 작은 파일에서도 마찬가지였다. `.zshrc`에 다음과 같이 설정하여 임시로 해결했다.

```
alias code="/usr/local/bin/code --disable-renderer-accessibility"
```
