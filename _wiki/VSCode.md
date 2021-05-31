---
layout: wiki 
title: VSCode
last-modified: 2021/05/30 14:19:22
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