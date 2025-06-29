---
layout: wiki 
title: VSCode
tags: ["Productivity"]
last_modified_at: 2025/06/24 13:40:38
---

<!-- TOC -->

- [검색 설정](#검색-설정)
- [Compare Folders](#compare-folders)
- [미니맵 색상 변경](#미니맵-색상-변경)
- [Extentions](#extentions)
- [커서가 느려지는 문제](#커서가-느려지는-문제)

<!-- /TOC -->

# 검색 설정
전체 설정에서 `@id:search.exclude`에서 `_site` 추가해서 generated html은 검색이 되지 않도록 설정

# Compare Folders
첫 번째가 my folder, 두 번째는 compared folder. 자동으로 순서가 선택되는 형태는 아니고 항상 위에서 아래로 선택되는 구조다. 변경은 Compare Folders 내에서 Swap Sides로 변경 가능

# 미니맵 색상 변경
노란색으로 눈에 잘 띄게 변경하고 항상 슬라이더가 표현되도록 수정한다.

```
Workbench: Color Customizations
Overrides colors from the currently selected color theme.
Edit in settings.json
```
내가 사용중인 `settings.json` 전체는 다음과 같다.

```
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
    "lpubsppop01.autoTimeStamp.modifiedTimeStart": "[lL]ast[ -_][mM]odified(?:|_at): ",
    "editor.fontSize": 14,
}
```

vscode.dev에도 동일하게 settings.json 수정하고, 깃헙 계정으로 `Settings Sync is On`으로 동기화해서 사용한다.

# Extentions
현재 사용 중인 extensions는 다음과 같다. (Settings Sync is On 상태)
- Auto Time Stamp
- Markdown All in One(Yu Zhang)
  - TOC를 생성해주는 익스텐션이 매우 많은데, 다들 조금씩 문제가 있다. TOC를 생성하는 용도로만 사용 중
- JetBrains IDE Keymap: 단축키는 모두 IntelliJ 기준으로 통일
- Marp for VS Code: 발표자료도 모두 markdown으로
- Compare Folders(MoshFeu): 첫 번째 upstream, 두 번째 working을 두고 Compare Folders를 진행하면 왼쪽에 최신 버전을 오른쪽으로 반영할 수 있다.

# 커서가 느려지는 문제
어느날 부터 키보드 움직임이 느려졌는데, 편집 중인 파일 사이즈가 너무 커서 그런가 했는데 작은 파일에서도 마찬가지였다. `.zshrc`에 다음과 같이 설정하여 임시로 해결했다.

```
alias code="/usr/local/bin/code --disable-renderer-accessibility"
```
