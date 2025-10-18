---
layout: wiki
title: vLLM
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/10/18 20:52:07
last_modified_history:
  - 2025/10/18 내용 정리
  - 2025/03/14 이전 버전
---

- [실행](#실행)
- [Multi Nodes](#multi-nodes)

# 실행
Reasoning 모델 (Qwen3) 구동:
```
$ vllm serve . \
--served-model-name xxx \
--reasoning-parser deepseek_r1 \
--max-model-len 32000 \
--port 18001
```

# Multi Nodes
`run_cluster.sh`를 제공하며, ray를 사용하고 docker로 구동한다. K8s내에서는 해당 docker 이미지를 배포하는 방식으로 적용이 가능할 거 같다. 스크립트에는 docker가 구동되자마자 ray를 실행하도록 되어 있다.