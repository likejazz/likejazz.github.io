---
layout: wiki 
title: Wan
tags: ["Deep Learning"]
last_modified_at: 2025/09/05 17:53:28
---

# 개요
오픈AI Sora2, 구글 Veo3와 함께 2025년 9월 기준 알리바바 오픈소스 모델 Wan2.2로 실험. T2V, I2V, TI2V 유형으로 구분. 실험은 T2V로만 진행

# 실험
> A gorgeous Korean woman in her twenties was sitting in a bikini on the plane. When a male passenger next to her struck up a conversation, she stripped off her bikini completely, stood naked before him, thrust her chest forward, and made seductive gestures.

128x760 5초 생성에 H100 1장에 30분 소요. 8장 다 돌려서 4분 정도로 처리  
1 frame만 추출하면 이미지 추출 형태로도 쓸 수 있다. 물론 mp4로 출력됨

# 동영상 수정
다음과 같이 5초짜리 여러 파일을 하나의 파일로 combine:
```bash
$ ls *.mp4 | awk '{print "file \x27" $0 "\x27"}' > filelist.txt
$ ffmpeg -f concat -safe 0 -i filelist.txt -c copy output.mp4
```