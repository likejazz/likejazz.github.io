---
layout: wiki 
title: Cursor
tags: ["Productivity"]
last_modified_at: 2026/09/17 18:44:10
last_modified_history:
  - 2026/09/11 작성
---

- [설정](#설정)

# 설정
User Settings (JSON) `settings.json`은 다음과 같다.

```json
{
    "window.commandCenter": true,
    "makefile.configureOnOpen": true,
    "git.confirmSync": false,
    "git.openRepositoryInParentFolders": "always",
    "security.promptForLocalFileProtocolHandling": false,
    "remote.SSH.remotePlatform": {
        "box": "linux",
        "runpod": "linux",
        "b200": "linux"
    },
    "window.autoDetectColorScheme": false,
    "workbench.agentsWindowButton.enabled": false,
    "outline.showVariables": false,
    "outline.showFields": false,
    "outline.showProperties": false,
    "outline.showConstants": false,
    "outline.showModules": false,
    "editor.minimap.enabled": true,
    "editor.minimap.showSlider": "always",
    "workbench.colorCustomizations": {
        "minimapSlider.activeBackground": "#ffff00b6",
        "minimapSlider.background": "#ffff0068",
        "minimapSlider.hoverBackground": "#ffff007f",
    },
    "editor.minimap.renderCharacters": false,
    "editor.minimap.maxColumn": 60,
    "editor.minimap.size": "fit"
}
```

서버 설정으로 하려 했으나 그렇게 하면 서버별로 모두 설정해줘야 한다. 유저 설정 또한 사용 기기별로 해야한다.

Cursor Settings → Git & PRs → Attribution 모두 off