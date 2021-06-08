---
layout: wiki 
title: VSCode
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [미니맵 색상 변경](#미니맵-색상-변경)

<!-- /TOC -->

# 미니맵 색상 변경
노란색으로 눈에 잘 띄게 변경 및 항상 슬라이더가 표현되도록

```json
"workbench.colorTheme": "Visual Studio Dark",
"workbench.colorCustomizations": {
    "minimapSlider.activeBackground": "#ffff00b6",
    "minimapSlider.hoverBackground": "#ffff006b",
    "minimapSlider.background": "#ffff002a",
},
"editor.minimap.showSlider": "always",
```