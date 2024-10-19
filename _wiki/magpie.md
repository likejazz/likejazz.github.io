---
layout: wiki 
title: magpie
tags: ["Continued Pretraining"]
last_modified_at: 2024/10/20 02:34:32
---

- [원리](#원리)

# 원리
`pre_query_template`를 지정하면 질문을 생성해준다. 학습한 데이터를 추출하겠다는 개념. 한국어로 contraint걸고 필터링 한다는 얘기가 있음.

프롬프트를 직접 주입하다 보니 API로는 호출이 안되고 직접 모델을 돌려야 한다. Qwen2 72B는 GPU 2장을 잡아야 한다.