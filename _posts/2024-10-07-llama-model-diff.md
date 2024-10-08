---
layout: post
title: ! 'Llama 3 vs 3.1 모델 비교'
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/10/09 01:38:00
---

<div class="message">
Llama 3와 3.1 모델의 차이를 비교해본다.
</div>

<small>
*Oct 7, 2024*
</small>

- [개요](#개요)
- [모델 구조 비교](#모델-구조-비교)
  - [라마 3.2 1B까지 차이](#라마-32-1b까지-차이)
- [모델 변화](#모델-변화)
  - [파라미터 설명](#파라미터-설명)
- [Llama 3.1 모델 및 파라미터 위치](#llama-31-모델-및-파라미터-위치)

# 개요

Llama 3와 3.1 모델이 얼마만큼 차이가 있는지 비교해본다. 또한 Qwen2와 아키텍처 차이를 확인해본다.

# 모델 구조 비교
llama 3 8B의 마지막 레이어는 다음과 같다. 이후 버전인 3.1, 3.2도 모두 동일한 구조를 갖고 있다.

```
model.layers.31.input_layernorm.weight	[4 096]	
model.layers.31.mlp.down_proj.weight	[4 096, 14 336]	
model.layers.31.mlp.gate_proj.weight	[14 336, 4 096]	
model.layers.31.mlp.up_proj.weight	[14 336, 4 096]	
model.layers.31.post_attention_layernorm.weight	[4 096]	
model.layers.31.self_attn.k_proj.weight	[1 024, 4 096]	
model.layers.31.self_attn.o_proj.weight	[4 096, 4 096]	
model.layers.31.self_attn.q_proj.weight	[4 096, 4 096]	
model.layers.31.self_attn.v_proj.weight	[1 024, 4 096]	
model.norm.weight	[4 096]	
lm_head.weight	[128 256, 4 096]	
```

Qwen2 7B의 마지막 레이어는 다음과 같다. 2.5도 동일한 구조를 갖고 있다.

```
model.layers.27.input_layernorm.weight	[3 584]	
model.layers.27.mlp.down_proj.weight	[3 584, 18 944]	
model.layers.27.mlp.gate_proj.weight	[18 944, 3 584]	
model.layers.27.mlp.up_proj.weight	[18 944, 3 584]	
model.layers.27.post_attention_layernorm.weight	[3 584]	
model.layers.27.self_attn.k_proj.bias	[512]	
model.layers.27.self_attn.k_proj.weight	[512, 3 584]	
model.layers.27.self_attn.o_proj.weight	[3 584, 3 584]	
model.layers.27.self_attn.q_proj.bias	[3 584]	
model.layers.27.self_attn.q_proj.weight	[3 584, 3 584]	
model.layers.27.self_attn.v_proj.bias	[512]	
model.layers.27.self_attn.v_proj.weight	[512, 3 584]	
model.norm.weight	[3 584]	
lm_head.weight	[152 064, 3 584]	
```

Vocab Size뿐만 아니라 아키텍처의 사이즈도 많이 다르며, Qwen2의 경우 QKV에 bias도 별도로 갖고 있다.

## 라마 3.2 1B까지 차이

GPT-2 이후 라마 2부터 변화 과정을 도식화하면 다음과 같다.[^fn-1] 라마 3 이후 GQA가 모든 모델에 적용된 것 외에는 계속 동일하다.

<img src="https://lh3.googleusercontent.com/pw/AP1GczObBjlI1I0SJgUchGokYCli1z3YJn9DxS8RwBkaLh8pT-s9jNEipcUnl4w1CYDkihF8B3u2fEeM0CHdYYjNtqi86CFZKQWrEoMBUU8hJUHJoGj_O4WqmMT8tNHKvdVcRu3QfSazPpMgDlY8iGiIkZzuug=w1200-h800-s-no-gm?authuser=0" width="100%">

[^fn-1]: <https://www.linkedin.com/posts/sebastianraschka_the-llama-32-1b-and-3b-models-are-my-favorite-activity-7248317830943686656-yyYD>

# 모델 변화

Llama 3 8B와 Llama 3.1 8B의 변화를 첫 번째 모델을 기준으로 퍼센트로 산출하면 다음과 같다. 일부 아웃라이어가 있기 때문에 평균이 아닌 중앙값(median)으로 산출했다.

<img src="https://lh3.googleusercontent.com/pw/AP1GczPdlU0gLzMtFmmrsyICF7YQfucBLMkkLvfTj90XBu1P5Oyps4Pnx-iWJprusm4M0OWkjGS3J69mQ7lryquvEEhQEKLG6V2GrGdJ1Y1LzNFuXJHrRzaS06iuBCbNdZC1HclXJx74OSVkAKOmVEIHDJfFog=w2874-h948-s-no-gm?authuser=0" width="100%">

주로 인풋에 가까운 레이어의 변화가 크며, Q의 첫 번째 레이어는 40% 가까운 변화를 보인다. input_layernorm과 post_attention_layernorm은 뒤로 갈수록 2%내로 변화가 줄어들며 전반적으로 큰 차이가 없다. 그러나 MLP를 비롯한 나머지 레이어는 대부분 20% 내외의 변화가 있다.

파라미터의 변화를 추출하는 코드는 [(비공개) #1-14 `model-diff.py`](/wiki/Private-Links/)에 비공개로 관리한다.

## 파라미터 설명
히트맵으로 언급한 각 파라미터와 모델에서의 위치는 다음과 같다.

- input_layernorm
- self_attn_q_proj
- self_attn_k_proj
- self_attn_v_proj
- self_attn_o_proj
- post_attention_layernorm
- mlp_gate_proj
- mlp_up_proj
- mlp_down_proj

<img src="https://lh3.googleusercontent.com/pw/AP1GczP7qBGzf3EvvWUBlWnz0vYOvP5MPbV-GUKty6YGH67gy5Z2q9sx9joJrEEpuP5v-qwx1tsc9oEY3bcFyB96D9j_AlA4b0XTlxLvSxtWQsQ-3ITSJi24T5lrEj3ZTdPkj058u6o_d1JuwFxZ3uzFGMP7IA=w1368-h1824-s-no-gm?authuser=0" width="70%">

# Llama 3.1 모델 및 파라미터 위치
앞서 모델 구조에 파라미터 위치를 표기하여 최종 정리해보면 다음과 같다.

<img src="https://lh3.googleusercontent.com/pw/AP1GczOr8S_3_JGIpBx8zByZaJP0DEBebH3bPv63UtmZaqdS6KgMMQ3dGOITnyggkeisWGh2qFRjPOxY-LO52W6xapfBcEUfG3CaTpIuu3oBIVTmPPXPy_e2yeMBgMJSvXQd-Ir5q6s0yuyBbAR3qFs92ApCDw=w922-h941-s-no-gm?authuser=0" width="100%">