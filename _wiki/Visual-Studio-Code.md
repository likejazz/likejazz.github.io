---
layout: wiki 
title: Visual Studio Code
tags: ["Productivity"]
last_modified_at: 2026/09/11 09:37:39
last_modified_history:
  - 2026/09/11 내용 정리
  - 2025/10/17 Windows 설치
  - 2025/06/24 이전 버전
---

<!-- TOC -->

- [설치 플러그인](#설치-플러그인)
- [Compare Folders](#compare-folders)
- [설정](#설정)

<!-- /TOC -->

# 설치 플러그인
- [**Visual Studio Code**](https://code.visualstudio.com/download)는 위키를 편집하거나 발표 자료를 작성하고, 코드 리뷰 용도로도 사용하는 메인 편집기다. Settings Sync를 하면(GitHub 계정) 모든 설정과 Extensions를 설치해준다. `$ brew install visual-studio-code`
  - 편리한 이용을 위해 [Shell Command](https://code.visualstudio.com/docs/setup/mac#_launching-from-the-command-line)를 설치한다.
  - 위키의 최근 수정 날짜를 갱신하는 **[Auto Time Stamp](https://marketplace.visualstudio.com/items?itemName=lpubsppop01.vscode-auto-timestamp)**. SSH에서 작업시 서버에도 플러그인 설치 필요
  - JetBrains Tools와 Keymaps를 맞추기 위한 **[JetBrains IDE Keymap](https://marketplace.visualstudio.com/items?itemName=isudox.vscode-jetbrains-keybindings)**.
  - Markdown TOC를 비롯한 여러 편리한 기능 **[Markdown All in One(Yu Zhang)](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)** SSH에서 작업시 서버에도 플러그인 설치 필요
  - 문서의 길이 비율 표시 **[scroll percentage](https://marketplace.visualstudio.com/items?itemName=IdanRudich.scroll-percentage)** 긴 문서를 편집할 때 유용하다. SSH에서 작업시 서버에도 플러그인 설치 필요
  - SSH 접속을 하면 다음 3개가 모두 설치된다. 모두 Microsoft의 공식 익스텐션이다. **[Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)**, **Remote - SSH: Editing Configuration Files**, **Remote Explorer**

# Compare Folders
Compare Folders(MoshFeu): 첫 번째 upstream, 두 번째 working을 두고 Compare Folders를 진행하면 왼쪽에 최신 버전을 오른쪽으로 반영할 수 있다.

첫 번째가 my folder, 두 번째는 compared folder. 자동으로 순서가 선택되는 형태는 아니고 항상 위에서 아래로 선택되는 구조다. 변경은 Compare Folders 내에서 Swap Sides로 변경 가능

# 설정
사용중인 `settings.json`는 다음과 같다.

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
        "minimapSlider.background": "#ffff0068",
        "minimapSlider.hoverBackground": "#ffff007f",
    },
    "editor.minimap.showSlider": "always",
    "editor.minimap.maxColumn": 60,
    "editor.minimap.size": "fit",
    "window.menuBarVisibility": "classic",
    "editor.fontSize": 14,
    "chat.disableAIFeatures": true,
    "security.allowedUNCHosts": [
        "wsl.localhost"
    ],
    "search.useIgnoreFiles": false,
    "search.exclude": {
        "**/_site": true
    },
    "lpubsppop01.autoTimeStamp.modifiedTimeStart": "[lL]ast[ -_][mM]odified(?:|[ -_]at): ",
    "lpubsppop01.autoTimeStamp.modifiedTimeEnd": "$",
    "lpubsppop01.autoTimeStamp.modifiedTimeFormat": "YYYY/MM/DD HH:mm:ss"
}
```

WSL2에서 Auto Time Stamp는 `/mnt/c/Users/xxx/AppData/Roaming/Code/User/settings.json`에 설정 필요
