---
layout: wiki 
title: lmeval
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/04/18 11:15:46
---

- [평가](#평가)
  - [종합](#종합)
    - [base](#base)
    - [instruct](#instruct)
      - [llama-based](#llama-based)
      - [korean-specific](#korean-specific)
      - [etc](#etc)
  - [mmlu / kmmlu\_direct](#mmlu--kmmlu_direct)
    - [~ 7B](#-7b)
      - [base](#base-1)
      - [instruct](#instruct-1)
    - [7B ~ 20B](#7b--20b)
      - [base](#base-2)
      - [instruct](#instruct-2)
    - [20 ~ 70B](#20--70b)
      - [base](#base-3)
      - [instruct](#instruct-3)
    - [70B ~](#70b-)
      - [base](#base-4)
      - [instruct](#instruct-4)
- [영어 평가](#영어-평가)
  - [MMLU](#mmlu)
  - [GPQA](#gpqa)
  - [ARC-C](#arc-c)
  - [GSM8K](#gsm8k)
- [한글 평가](#한글-평가)
  - [kmmlu](#kmmlu)
  - [haerae](#haerae)
  - [kobest](#kobest)
- [Troubleshooting](#troubleshooting)
- [실행](#실행)

# 평가
## 종합
### base

| 모델 | mmlu | kmmlu | mmlu_pro | gsmk8k | boolq | copa | wic | hellaswag | sentineg |
| --- | ---- | ---- | -------- | ------ | ----- | ---- | --- | --------- | -------- |
| meta-llama/Meta-Llama-3-8B | 65.33 | 39.87 | 34.72 | 49.20 | 81.48 | 71.20 | 53.57 | 45.00 | 95.97 |
| **meta-llama/Meta-Llama-3.1-8B** | **65.03** | **41.02** | **35.46** | **50.95** | **81.48** | **71.70** | **55.63** | **45.20** | **94.96** |
| tiiuae/Falcon3-7B-Base | 70.02 | 29.21 | 41.12 | 75.44 | 63.18 | 48.80 | 52.86 | 35.60 | 64.74 |
| tiiuae/Falcon3-10B-Base | 73.07 | 31.35 | 46.79 | 80.82 | 71.65 | 50.00 | 55.00 | 35.00 | 77.58 |
| **Qwen/Qwen2.5-7B** | **74.18** | **51.70** | **48.31** | **81.88** | **90.53** | **72.80** | **68.89** | **48.80** | **92.19** |
| Qwen/Qwen2.5-14B | 79.77 | 59.49 | 56.82 | 86.35 | 92.95 | 79.50 | 81.75 | 50.00 | 97.98 |
| Qwen/Qwen2.5-32B | 83.26 | 62.81 | 54.25 | 79.38 | 96.23 | 81.40 | 88.02 | 53.00 | 97.48 |

### instruct
32B 이상은 너무 느려서 hf 대신에 vllm으로 측정했다. hf는 mmlu만 해도 40h 넘게 걸린다. multi gpu도 utilization을 50%도 채 못쓰는데 processes 수를 조정해도 다 쓰게 할 수 없었다.

#### llama-based

| 모델 | mmlu | kmmlu | mmlu_pro | gsmk8k | boolq | copa | wic | hellaswag | sentineg |
| --- | ---- | ---- | -------- | ------ | ----- | ---- | --- | --------- | -------- |
| meta-llama/Meta-Llama-3-8B-Instruct | 65.62 | 38.58 | 39.57 | 73.46 | 85.47 | 68.80 | 54.52 | 42.60 | 92.44 |
| **meta-llama/Meta-Llama-3.1-8B-Instruct** | **68.25** | **41.62** | **40.87** | **76.88** | **86.97** | **70.80** | **51.43** | **44.60** | **93.45** |
| KISTI-KONI/KONI-Llama3.1-8B-Instruct-20241024 | 63.40 | 40.73 | 37.69 | 66.49 | 87.82 | 75.90 | 62.06 | 44.40 | 94.71 |
| NCSOFT/Llama-VARCO-8B-Instruct | 63.22 | 38.25 | 37.13 | 65.13 | 88.89 | 75.90 | 58.49 | 44.00 | 91.94 |
| allganize/Llama-3-Alpha-Ko-8B-Instruct | 63.58 | 38.43 | 33.37 | 58.76 | 84.33 | 71.80 | 53.10 | 44.60 | 94.46 |
| **dnotitia/DNA-1.0-8B-Instruct** | **66.71** | **51.42** | **44.27** | **79.45** | **91.52** | **83.20** | **80.95** | **52.20** | **94.21** |

#### korean-specific

| 모델 | mmlu | kmmlu | mmlu_pro | gsmk8k | boolq | copa | wic | hellaswag | sentineg |
| --- | ---- | ---- | -------- | ------ | ----- | ---- | --- | --------- | -------- |
| **Qwen/Qwen2.5-7B-Instruct** | **74.23** | **45.61** | **42.73** | **75.82** | **92.95** | **74.10** | **74.05** | **49.40** | **95.97** |
| Qwen/Qwen2.5-14B-Instruct | 79.85 | 57.66 | 49.62 | 79.00 | 95.09 | 82.30 | 78.73 | 51.20 | 98.49 |
| Qwen/Qwen2.5-32B-Instruct | 83.30 | 60.84 | 56.02 | 82.56 | 96.72 | 82.90 | 86.90 | 50.00 | 96.98 |
| LGAI-EXAONE/EXAONE-3.0-7.8B-Instruct | 64.29 | 45.48 | 39.05 | 80.59 | 90.95 | 85.20 | 71.98 | 49.20 | 98.74 |
| LGAI-EXAONE/EXAONE-3.5-7.8B-Instruct | 65.63 | 44.71 | 40.71 | 65.50 | 90.53 | 84.60 | 66.43 | 50.00 | 98.24 |
| LGAI-EXAONE/EXAONE-3.5-32B-Instruct | 74.32 | 51.03 | 47.98 | 59.89 | 95.58 | 89.40 | 83.10 | 53.00 | 98.99 |
| CohereForAI/aya-23-8B | 55.11 | 34.34 | 23.17 | 43.14 | 81.84 | 73.00 | 56.83 | 48.80 | 94.71 |
| CohereForAI/aya-expanse-8b | 62.80 | 40.04 | 32.26 | 76.88 | 89.89 | 79.20 | 69.76 | 47.40 | 98.24 |
| CohereForAI/aya-expanse-32b | 74.40 | 49.33 | 45.31 | 84.99 | 95.09 | 89.00 | 85.08 | 53.20 | 98.49 |

#### etc

| 모델 | mmlu | kmmlu | mmlu_pro | gsmk8k | boolq | copa | wic | hellaswag | sentineg |
| --- | ---- | ---- | -------- | ------ | ----- | ---- | --- | --------- | -------- |
| tiiuae/Falcon3-7B-Instruct | 70.54 | 31.18 | 46.75 | 80.29 | 72.58 | 49.00 | 54.52 | 35.60 | 78.84 |
| tiiuae/Falcon3-10B-Instruct | 73.00 | 22.82 | 49.76 | 81.12 | 78.13 | 50.40 | 58.25 | 35.80 | 79.85 |
| **google/gemma-2-9b-it** | **72.35** | **46.62** | **49.48** | **80.52** | **93.80** | **77.70** | **73.25** | **44.80** | **97.23** |
| rtzr/ko-gemma-2-9b-it | 72.38 | 46.59 | 48.12 | 74.98 | 92.59 | 78.10 | 74.68 | 44.80 | 97.23 |
| yanolja/EEVE-Korean-10.8B-v1.0 | 63.32 | 42.36 | 30.33 | 50.11 | 91.52 | 83.40 | 68.57 | 50.60 | 97.23 |
| upstage/solar-pro-preview-instruct (22B) | 79.15 | 40.95 | 57.46 | 86.05 | 90.38 | 59.60 | 65.24 | 42.40 | 89.92 |
| allenai/OLMo-2-1124-7B-Instruct | 60.52 | 32.44 | 31.91 | 75.82 | 73.08 | 52.30 | 53.49 | 38.20 | 77.08 |
| allenai/OLMo-2-1124-13B-Instruct | 65.94 | 33.59 | 35.39 | 81.20 | 83.12 | 56.60 | 56.27 | 39.60 | 50.63 |
| microsoft/phi-4 | 80.37 | 51.88 | 59.29 | 90.60 | 93.73 | 72.20 | 74.44 | 47.80 | 96.98 |
| mistralai/Mistral-Small-24B-Instruct-2501 | 80.44 | 54.55 | 57.73 | 89.76 | 95.51 | 81.10 | 76.19 | 51.20 | 97.23 |

## mmlu / kmmlu_direct
- `kmmlu_direct` 689s elapsed. 5-shot

### ~ 7B

#### base

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| google/gemma-2b | 40.99 | 21.01 |
| google/gemma-2-2b | 52.85 | 31.45 |
| Qwen/Qwen-1_8B | 44.97 | 28.65 (fix token error) |
| Qwen/Qwen1.5-1.8B | 45.64 | 23.59 | 
| Qwen/Qwen2-1.5B | 55.93 | 37.38 | 
| Qwen/Qwen2.5-1.5B | xx | xx | 
| Qwen/Qwen2.5-3B | xx | xx | 
| 42dot/42dot_LLM-PLM-1.3B | 26.04 | 29.78 |

#### instruct

| 모델 | mmlu | kmmlu | mmlu_pro | gsmk8k | boolq(kobest) | copa(kobest) | wic(kobest) | hellaswag(kobest) | sentineg(kobest) |
| --- | ---- | ---- | -------- | ------ | ----- | ---- | --- | --------- | -------- |
| google/gemma-3-1b-it | 38.26 | 23.71 | 13.03 | 12.96 | 62.89(52.68) | 53.60(52.68) | 49.76(52.68) | 35.20(52.68) | 61.96(52.68) |
| Qwen/Qwen2.5-0.5B-Instruct | 47.06 | 30.30 | 16.67 | 32.22 | 53.85(48.38) | 52.00(48.38) | 48.25(48.38) | 34.40(48.38) | 53.40(48.38) |
| Qwen/Qwen2.5-1.5B-Instruct | 60.25 | 37.23 | 31.87 | 53.53 | 64.81(59.25) | 59.50(59.25) | 55.95(59.25) | 39.20(59.25) | 76.83(59.25) |

### 7B ~ 20B

#### base

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Llama-2-7b-hf | 45.7 | 24.35 | 
| meta-llama/Meta-Llama-3-8B | 65.04 | 40.03 |
| **meta-llama/Meta-Llama-3.1-8B** | **65.23** | **41.06** |
| google/gemma-7b | 62.98 | 40.83 |
| **google/gemma-2-9b** | **70.28** | **47.05** |
| Qwen/Qwen-7B | 58.44 | 35.36 (fix token error) | 
| Qwen/Qwen1.5-7B | 60.46 | 38.57 | 
| Qwen/Qwen2-7B | 70.55 | 49.05 | 
| **Qwen/Qwen2.5-7B** | **74.15** | **51.71** |
| mistralai/Mistral-7B-v0.1 | 62.39 | 37.38 | 
| mistralai/Mistral-7B-v0.3 | 62.33 | 36.49 | 
| upstage/SOLAR-10.7B-v1.0 | 64.20 | 39.04 | 
| yanolja/EEVE-Korean-10.8B-v1.0 | 63.25 | 42.68 |
| KISTI-KONI/KONI-Llama3-8B-20240630 | 62.68 | 0 |
| KISTI-KONI/KONI-Llama3-8B-Merged-20240724 | 65.55 | 0 |
| beomi/Llama-3-KoEn-8B | 52.44 | 40.66 |
| beomi/Llama-3-Open-Ko-8B | 56.29 | 40.50 |
| chatbaker-7b-base (300b, private) | 39.97 | 29.73 |
| chatbaker-7b-base (private) | 38.29 | 29.09 |


#### instruct

| 모델 | mmlu | kmmlu_direct | mmlu_pro | gsmk8k | kobest |
| --- | ----- | ----------- | ------ | ------ |
| **meta-llama/Meta-Llama-3.1-8B-Instruct** | **68.18** | **41.37** | **41.09** | **77.02** | **69.45** |
| google/gemma-2-9b-it | 72.26 | 46.36 |
| mistralai/Mistral-7B-Instruct-v0.3 | 61.98 | 31.41 |
| yanolja/EEVE-Korean-Instruct-10.8B-v1.0 | 63.62 | 42.17 |
| Qwen/Qwen2-7B-Instruct | 70.67 | 45.97 |
| **Qwen/Qwen2.5-7B-Instruct** | **74.25** | **46.06** | **44.77** | **79.75** | **76.84** |
| LGAI-EXAONE/EXAONE-3.0-7.8B-Instruct | 64.14 | 45.03 | 38.92 | 79.75 | 78.93 |
| LGAI-EXAONE/EXAONE-3.5-7.8B-Instruct | 65.63 | 43.67 | 40.80 | 66.03 | 78.15 |
| ghost-x/ghost-8b-beta-1608 | 61.55 | 37.01 |
| NCSOFT/Llama-VARCO-8B-Instruct | 63.35 | 38.41 |
| rtzr/ko-gemma-2-9b-it | 72.43 | 46.81 |
| MLP-KTLim/llama-3-Korean-Bllossom-8B | 64.80 | 37.30 |
| CohereForAI/aya-23-8B | 55.03 | 33.74 | 23.08 | 43.06 | 70.75 |
| CohereForAI/aya-expanse-8b | 62.84 | 39.63 | 32.47 | 75.36 | 76.60 |
| **dnotitia/DNA-1.0-8B-Instruct** | **66.58** | **51.56** | **43.13** | **79.30** | **80.76** |
| KISTI-KONI/KONI-Llama3.1-8B-Instruct-20241024 | 63.49 | 40.48 | 37.39 | 67.39 | 72.25 |

모든 평가는 chat template을 사용하지 않았다. 기존에는 vllm, 신규 종합 평가는 hf를 사용했다.

### 20 ~ 70B

#### base

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Llama-2-13b-hf | 55.06 | 31.55 | 
| google/gemma-2-27b | 75.45 | 50.30 | 
| Qwen/Qwen-14B | 67.04 | 44.05 (fix token error) | 
| Qwen/Qwen1.5-14B | 67.79 | 45.25 |
| Qwen/Qwen1.5-32B | 73.55 | 48.07 | 
| Qwen/Qwen2-57B-A14B | 75.58 | 54.25 |
| Qwen/Qwen2.5-14B | 79.84 | 59.36 |
| mistralai/Mistral-Nemo-Base-2407 | 69.01 | 43.87 | 
| mistralai/Mixtral-8x7B-v0.1 | 70.48 | 39.91 | 
| beomi/Yi-Ko-34B | 74.83 | 50.34 |
| vaiv/GeM2-Llamion-14B-Base | 67.43 | 45.51 |
| chatbaker-13b-base (private) | 51.61 | 38.45 |

#### instruct

| 모델 | mmlu | kmmlu_direct | mmlu_pro | gsm8k | kobest | 
| --- | ----- | ----------- | -------- | ----- | ------ |
| Qwen/Qwen2.5-14B-Instruct | 79.96 | 57.94 | 52.42 | 83.54 | 80.91 |
| Qwen/Qwen2.5-32B-Instruct | 83.32 | 60.83 | 55.98 | 82.48 | 82.71 | 
| mistralai/Mistral-Small-Instruct-2409 | 72.53 | 44.19 | 
| upstage/solar-pro-preview-instruct | 79.14 | 41.01 | 
| CohereForAI/aya-23-35B | 67.46 | 42.32 | 33.48 | 63.76 |
| CohereForAI/aya-expanse-32b | 74.39 | 49.30 | 45.31 | 85.51 | 84.06 |
| LGAI-EXAONE/EXAONE-3.5-32B-Instruct | 65.64 | 47.42 | 45.85 | 55.26 |

solar-pro는 vllm에서 지원하지 않아 hf에서 구동

### 70B ~

#### base

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Llama-2-70b-hf | 68.57 | 41.05 |
| meta-llama/Meta-Llama-3-70B | 78.67 | 53.23 |
| meta-llama/Meta-Llama-3.1-70B | 78.62 | 52.38 |
| Qwen/Qwen-72B | 77.28 | 52.39 (fix token error) |
| Qwen/Qwen1.5-72B | 77.18 | 52.15 |
| Qwen/Qwen2-72B | 84.22 | 65.20 | 
| mistralai/Mixtral-8x22B-v0.1 | 77.75 (expected) | 23.14 | 

#### instruct

| 모델 | mmlu | kmmlu_direct | mmlu_pro | gsm8k | kobest |
| --- | ----- | ----------- | ------ | ------ |
| meta-llama/Meta-Llama-3-8B-Instruct | 79.78 | 53.57 | 58.46 | 91.20 | 79.48 |
| meta-llama/Llama-3.1-70B-Instruct | 82.15 | 51.91 | 58.76 | 88.93 | 80.21 |
| meta-llama/Llama-3.3-70B-Instruct | 82.22 | 56.97 | 60.44 | 90.29 | 81.27 |
| Bllossom/llama-3-Korean-Bllossom-70B | 27.29 | 30.50 | 13.59 | 01.06 | 56.29 |
| moreh/Llama-3-Motif-102B-Instruct | 83.90 | 63.49 | 57.18 | 78.88 |
| KISTI-KONI/KONI-Llama3.1-70B-Instruct-20241115 | 80.15 | 51.33 | 56.17 | 91.88 | 78.42 |
| Qwen/Qwen2.5-72B-Instruct | 84.56 | 40.02 | 62.80 | 93.17 | 84.17 |

qwen2-72b는 kmmlu는 가능한데 mmlu는 계속 out of memory라서 gpu 4장에서 진행. mmlu가 메모리를 더 많이 쓰며 희안하게 0번 gpu에만 추가로 20GB 더 점유. 때문에 `gpu_utilization_memory=0.65`로 제한하여 다른 GPU도 최대 51GB밖에 사용하지 못함.

| 모델 | mmlu | kmmlu | mmlu_pro | gsmk8k | boolq | copa | wic | hellaswag | sentineg |
| --- | ---- | ---- | -------- | ------ | ----- | ---- | --- | --------- | -------- |
| nvidia/Llama-3.1-Nemotron-70B-Instruct-HF | 82.39 | 53.51 | 59.21 | 83.70 | 95.51 | 84.90 | 72.54 | 52.20 | 97.98 |

# 영어 평가
## MMLU
`write_out` 옵션은 프롬프트 예제 출력. H100에서 684s elapsed.

| 모델 | published | measured |
| --- | --- | --- |
| meta-llama/Meta-Llama-3-8B | 66.6 (5-shot) | 65.04 |
| meta-llama/Meta-Llama-3-8B-Instruct | 68.4 (5-shot) | 65.72, 33.57 (chat), 66.93 (multiturn) |
| meta-llama/Meta-Llama-3.1-8B | | 65.23 (5-shot) |
| meta-llama/Meta-Llama-3.1-8B-Instruct | 69.4 (5-shot) | 68.02, 55.86 (chat), 68.24 (multiturn) |
| mistralai/Mistral-7B-Instruct-v0.2 | | 58.28 |
| mistralai/Mistral-7B-Instruct-v0.3 | | 60.65 |
| mistralai/Mistral-Nemo-Instruct-2407 | | 65.95 |
| yanolja/EEVE-Korean-Instruct-10.8B-v1.0 | | 62.18 |
| KISTI-KONI/KONI-Llama3-8B-Instruct-20240729 | | 62.43 |
| Qwen/Qwen2-7B-Instruct | | 70.77 |
| google/gemma-2-2b-it | | 56.84 (chat error) |
| google/gemma-2-9b-it | | 72.30 (chat error) |
| google/gemma-2-27b-it | | 76.27 (chat error) |
| upstage/SOLAR-10.7B-Instruct-v1.0 | | 64.33 |

chat은 `--apply_chat_template` [옵션](https://github.com/EleutherAI/lm-evaluation-harness/blob/main/docs/interface.md) 적용. [라마3 평가](https://github.com/meta-llama/llama3/blob/main/eval_details.md)

- The micro average numbers for MMLU are: 65.4 and 67.4 for the 8B pre-trained and instruct-aligned models.
- For the instruct-aligned models, we use a dialogue prompt (*user/assistant*) for the shots and ask the model to generate the best choice character as answer.

## GPQA
- `--tasks gpqa_main_zeroshot` 33s elapsed
- `--tasks gpqa_main_cot_zeroshot` flexible-extract, 246s elapsed.

| 모델 | published | measured |
| --- | --- | --- |
| meta-llama/Meta-Llama-3-8B-Instruct | 34.2 (0-shot) | 29.46, 27.68 (chat) |
| meta-llama/Meta-Llama-3.1-8B-Instruct | | 35.71 (0-shot), 33.48 (chat), 35.71 (5-shot, no chat), 33.48 (5-shot, multiturn)  |
| meta-llama/Meta-Llama-3.1-8B-Instruct | 32.8 (0-shot, CoT) | 12.72, 10.27 (chat) |

- We report 0-shot exact match scores over the possible options using the Main subset for our models and other open-source models (Mistral, Gemma).

## ARC-C
- `--tasks arc_challenge --num_fewshot 25` 197s elapsed.

| 모델 | published | measured |
| --- | --- | --- |
| meta-llama/Meta-Llama-3-8B | 78.6 (25-shot) | 54.35 |
| meta-llama/Meta-Llama-3.1-8B-Instruct | 83.4 (0-shot) | 56.57 (25-shot), 51.71, 51.45 (chat) |

## GSM8K
- `--tasks gsm8k_cot --gen_kwargs max_gen_toks=512`[^fn-option] flexible-extract, 960s elapsed.

[^fn-option]: <https://github.com/EleutherAI/lm-evaluation-harness/issues/1799>

| 모델 | published | measured |
| --- | --- | --- |
| meta-llama/Meta-Llama-3-8B-Instruct | 79.6 (8-shot, CoT) | 77.26, 68.23 (chat) |
| meta-llama/Meta-Llama-3.1-8B-Instruct | 84.5 (8-shot, CoT) | 77.10, 82.79 (chat), 84.84 (multiturn) |

# 한글 평가
## kmmlu
- `--tasks kmmlu` 전체 12h elapsed.
  - kmmlu_hard_direct
  - kmmlu_hard 5-shot 모두 무슨 차이인지
- `kmmlu_direct`로 진행

| 모델 | measured |
| --- | --- |
| google/gemma-2-2b-it | 24.1 xx |
| google/gemma-2-9b-it | 37.7 xx |
| google/gemma-2-27b-it | 55.2 xx |

## haerae
- `--tasks haerae` 107s elapsed.
  - chat template의 경우 다음 2가지 옵션 모두 지정시 성능이 나온다. `--apply_chat_template --fewshot_as_multiturn`
- 또한 n-shot에서 결과가 더 좋다. 아래 결과는 5-shot, multiturn

| 모델 | measured |
| --- | --- |
| meta-llama/Llama-2-7b-chat-hf | 36.20 |
| meta-llama/Meta-Llama-3-8B | 61.69 |
| meta-llama/Meta-Llama-3-8B-Instruct | 63.7 |
| meta-llama/Meta-Llama-3.1-8B | 62.6 |
| meta-llama/Meta-Llama-3.1-8B-Instruct | 65.08 |
| google/gemma-2-2b-it | 50.5 |
| google/gemma-2-9b-it | 67.55 |
| google/gemma-2-27b-it | 69.11 |
| mistralai/Mistral-7B-Instruct-v0.2 | 47.84, 48.12 (no chat)|
| mistralai/Mistral-7B-Instruct-v0.3 | 49.49, 45.27 (no chat) |
| upstage/SOLAR-10.7B-Instruct-v1.0 | 61.77 |
| 한글 지원 모델 | |
| yanolja/EEVE-Korean-Instruct-10.8B-v1.0 | 75.43 |
| Qwen/Qwen2-7B-Instruct | 59.95  |
| mistralai/Mistral-Nemo-Instruct-2407 | 71.04 |
| KISTI-KONI/KONI-Llama3-8B-Instruct-20240729 | 60.04 |
| I-BRICKS/Cerebro_BM_solar_v01 | 68.28 |
| 42dot/42dot_LLM-PLM-1.3B | 19.89 |
| 42dot/42dot_LLM-SFT-1.3B | 20.62 (chat template problem), 19.24 (no chat) |

## kobest
- `--tasks kobest` 337s elapsed.

# Troubleshooting
- `lm-eval[api]`로 local-chat-completition이 잘 진행되지 않음.
- lm-eval시 예전에는 프로세스 2개가 같은 gpu를 바라보는 문제가 있었는데, 새로 셋업하니 이번에는 각각 gpu를 잘 바라본다. (비공개) #1-8 실행 스크립트 참고, 그러나 hf에서 72B는 메모리 부족으로 실행 안됨
- gemma-2-9b가 너무 느려서 조언을 받아 vllm으로 시도. 2장으로 tensor_parallel이 잘 안된다. GPU 1장에서 `max_model_len=2048,max_num_batched_tokens=2048,gpu_utilization_memory=0.7,enforce_eager=True` 해야 OOM없이 진행된다.
- 기본 lm-eval 설치 후 hf에서 gemma-2 점수가 모두 이상하게 나온다.
  - Qwen도 1버전 실행 안됨. vllm에서 mmlu는 되는데 kmmlu는 token error 발생. 다음과 같이 패치:
```
$ vi /root/.cache/huggingface/modules/transformers_modules/Qwen/Qwen-1_8B/fa6e214ccbbc6a55235c26ef406355b6bfdf5eed/tokenization_qwen.py
# 165 line
continue
# 277 line
if token_ids is None: token_ids = [151643]
```
- pytorch ngc 이미지에서 `pip install lm-eval[vllm]`로 설치하고 `pip uninstall -y transformer-engine pynvml`로 로딩 에러 해결. 로컬 설치는 `pip install -e ".[vllm]"`
- hf와 vllm의 json 결과가 다르다. vllm은 groups 결과가 없다. 직접 평균을 낼 수 밖에 없는 구조
- vllm으로 mmlu는 `gpu_utilization_memory=0.7`로 진행. 메모리 설정이 까다롭다.
- 모든 tasks를 묶어서 한 번에 진행하면 훨씬 더 빠르게 진행할텐데 계속 CUDA OOM 때문에 진행할 수가 없다. 그래서 각 task 별로 따로 진행. vllm이고, 특히 mmlu에서 유난히 에러가 잦다. 현재는 `gpu_utilization_memory=0.6`으로 진행.
- hf에서 너무 느려서 vllm으로 갈아탄건데, 지원하지 않는 모델도 있고 memory 설정도 까다로워서 다시 hf로 롤백. 생각해보면 hf에서 bfloat16으로 강제했던게 속도 문제가 아니었나 싶다. 당시 gemma에서 엄청나게 느린 속도가 나왔던 걸로 기억. flash_attn을 재설치해줬더니 에러가 발생하지 않는다.
- 점수 재현을 위해 `--seed 42`로 고정했다. 예전부터 쓰던건데 뒤늦게 기입.

# 실행

```
$ cd /home/jovyan/slm-continued-pretraining
$ ./eval.sh
```
