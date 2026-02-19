---
layout: wiki 
title: mergekit
tags: ["LLM Training"]
last_modified_at: 2026/02/19 14:22:50
last_modified_history:
  - 2025/01/12
---

- [테스트를 위한 성능 비교](#테스트를-위한-성능-비교)
- [Tokenizer](#tokenizer)
- [기타](#기타)

# 테스트를 위한 성능 비교

| 모델 | mmlu | mmlu_pro | gsmk8k | arc_easy | arc_challenge |
| --- | ---- | -------- | ------ | -------- | ------------- |
| meta-llama/Meta-Llama-3-8B-Instruct | 65.76 | 39.84 | 75.21 | 84.72 | 56.14 |
| HuggingFaceTB/SmolLM2-135M | 25.17 | 7.01 | 0.61 | 66.67 | 28.41 |
| HuggingFaceTB/SmolLM2-135M-Instruct | 25.30 | 6.56 | 1.67 | 62.96 | 27.65 |
| HuggingFaceTB/SmolLM2-360M | 25.18 | 9.92 | 4.40 | 71.04 | 37.63 |
| HuggingFaceTB/SmolLM2-360M-Instruct | 25.92 | 10.17 | 4.32 | 70.24 | 34.56 |
| HuggingFaceTB/SmolLM2-1.7B | 50.19 | 19.33 | 31.31 | 79.80 | 50.09 |
| HuggingFaceTB/SmolLM2-1.7B-Instruct | 49.44 | 17.62 | 0.23 | 78.91 | 47.70 |

SFT 결과:  
base는 모두 SmolLM2-1.7B [생성 실험](/wiki/Text-Generation), [axolotl](/wiki/CPT-Axolotl)로 학습

| 모델 | mmlu | mmlu_pro | gsmk8k | arc_easy | arc_challenge |
| --- | ---- | -------- | ------ | -------- | ------------- |
| fft-smollm2-r250110 | 50.23 | 19.39 | 29.87 | 79.80 | 50.09 |
| fft-smollm2-finetome-r250110 | 50.04 | 19.51 | 34.42 | 80.68 | 49.66 |

- fft-smollm2-r250110: 학습이 거의 안되었을 뿐더러 special_tokens가 잘못지정되어 있어 모델이 비정상이다.
- fft-smollm2-finetome-r250110: 학습 데이터는 더 크지만 special tokens 문제는 동일하다.

학습시 다음 3가지 처리:
- special tokens 처리, 기존 smollm2 instruct와 동일하게
- wandb 연동
- batch size 키움

여전히 special token 문제인지, 출력 결과가 정상적이지 않다. 너무 엉뚱한 대답이 나온다. `chat_template: llama3`를 좀 수정해봐야할거 같다.

# Tokenizer
[smollm2 instruct tokenizer_config.json](https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct/blob/main/tokenizer_config.json) 참조. chat_template을 적을 수 없어 링크로 처리

# 기타
42dot:

| 모델 | mmlu | kmmlu | mmlu_pro | gsmk8k | boolq | copa | wic | hellaswag | sentineg |
| 42dot/42dot_LLM-PLM-1.3B | 26.90 | 29.08 | 9.27 | 1.36 | 51.42 | 71.20 | 51.51 | 43.60 | 90.68 |
| 42dot/42dot_LLM-SFT-1.3B | 25.09 | 23.94 | 9.84 | 1.82 | 60.90 | 74.10 | 51.27 | 42.20 | 89.92 |

영어 성능이 너무 떨어지므로 사용할 수 없다.

---
CPU에서 진행하므로 GPU 메모리와 큰 관련이 없다.

정리 후 삭제:
- mergekit <https://github.com/arcee-ai/mergekit>
- llamafy qwen <https://github.com/hiyouga/LLaMA-Factory/blob/main/scripts/llamafy_qwen.py>
- cpt 논문: Don't Stop Pretraining: Adapt Language Models to Domains and Tasks <https://arxiv.org/abs/2004.10964>
