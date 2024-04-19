---
layout: wiki 
title: RLHF
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/12 12:53:21
---

- [서비스](#서비스)
- [trlX](#trlx)
- [SFT 학습](#sft-학습)
  - [LIMA](#lima)

# 서비스
<img src="https://user-images.githubusercontent.com/388154/229353706-230d70e1-7d09-48e2-a884-4da768bccf6f.png" width="100%">[^fn-tabby]

[^fn-tabby]: <https://github.com/TabbyML/tabby>

# trlX
발표[^fn-pt]

[^fn-pt]: <https://www.slideshare.net/likejazz/trlx-framework-reviews>

# SFT 학습
Vicuna 학습 시 wandb 로그인:
```
$ wandb login
```
wandb 사이트에서 User Settings > API 키를 찾아서 입력한다. 또는,
```
$ export WANDB_MODE=offline
```
으로 비활성화 가능하다.

## LIMA
Meta의 LIMA 논문에서는 1,000개로 gpt-4에 근접하는 성능을 낼 수 있다고 했지만 한글 데이터로 실험 결과 1,000개로는 부족했고 5,000개 정도로 괜찮은 결과를 낼 수 있었다.