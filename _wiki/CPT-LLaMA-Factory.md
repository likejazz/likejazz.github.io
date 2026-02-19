---
layout: wiki 
title: CPT LLaMA-Factory
tags: ["LLM Training"]
last_modified_at: 2026/02/19 14:18:27
---

- [LLaMA-Factory](#llama-factory)
  - [kmmlu train](#kmmlu-train)
    - [1.0e-4 (default)](#10e-4-default)
    - [1.0e-5](#10e-5)
    - [5.0e-6](#50e-6)
    - [3.0e-6 (채택)](#30e-6-채택)
    - [1.0e-6](#10e-6)
    - [기타 실험](#기타-실험)
      - [device / packing](#device--packing)
  - [textbooks 1000 cleaned](#textbooks-1000-cleaned)
    - [textbooks-context-4k-token-10m](#textbooks-context-4k-token-10m)
    - [textbooks-1294-raw-token-151m](#textbooks-1294-raw-token-151m)
  - [release](#release)
    - [DeepSpeed 실험하면서 textbook 학습](#deepspeed-실험하면서-textbook-학습)
    - [textbook 3단계 학습 (r241025)](#textbook-3단계-학습-r241025)
  - [Claude 실험](#claude-실험)
    - [Claude 학습과 비교](#claude-학습과-비교)
    - [HAERAE-HUB 데이터](#haerae-hub-데이터)
  - [mergekit 실험](#mergekit-실험)
    - [SFT 진행](#sft-진행)
    - [SFT 모델 + llama instruct merge 실험](#sft-모델--llama-instruct-merge-실험)
    - [최종 모델 merge 전후 평가](#최종-모델-merge-전후-평가)
    - [merged 모델 SFT 후 다시 merge](#merged-모델-sft-후-다시-merge)

# LLaMA-Factory

## kmmlu train
llama-factory를 이용해서 kmmlu token 150m 학습을 진행했으나 llama-factory SFT learning rate 기본값 `1.0e-4`에서 모델이 엉망이 됐다.

### 1.0e-4 (default)
SFT default는 learning rate가 너무 높다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| checkpoint-1000 | 36.66 | 13.26 |
| checkpoint-1000 (다시) | 37.33 | 14.88 |

kmmlu train 3000:  
같은 이유로 모델이 엉망이 됐다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| kmmlu train 3000 | 43.08 | 20.42 |

### 1.0e-5
성능이 오락가락하므로 정상이 아니다. 조금 더 낮춰야 한다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| checkpoint-1000 | 62.10 | 38.60 |
| checkpoint-2000 | 63.04 | 42.94 |
| checkpoint-3000 | 61.05 | 40.78 |
| checkpoint-4000 | 61.54 | 41.59 |
| checkpoint-5000 | 61.91 | 42.93 |
| checkpoint-6000 | 62.61 | 44.75 |
| final | 62.48 | 44.58 |

### 5.0e-6
여기서부터는 괜찮다. 하지만 좀 더 안정적이면 좋겠다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| checkpoint-1000 | 64.39 | 41.34 |
| checkpoint-2000 | 65.01 | 44.68 |
| checkpoint-3000 | 64.00 | 43.26 |
| checkpoint-4000 | 64.17 | 44.72 |
| checkpoint-5000 | 64.60 | 45.15 |
| checkpoint-6000 | 64.58 | 46.13 |
| final | 64.60 | 45.89 |

### 3.0e-6 (채택)
가장 합리적으로 보인다. 향후 이 값으로 계속 사용

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| checkpoint-8000 | 64.04 | 44.01 |
| checkpoint-10000 | 63.53 | 42.85 |
| checkpoint-13000 (1epch) | 63.83 | 44.65 |
| checkpoint-15000 | 64.71 | 44.96 |
| checkpoint-20000 | 64.75 | 46.62 |
| checkpoint-25000 | 64.81 | 46.82 |
| checkpoint-26000 | 64.69 | 46.77 |
| final | 64.87 | 46.91 |

### 1.0e-6
`3.0e-6`으로 학습하려고 했으나 졸려서 설정을 잘못한거 같다. `1.0e-6`으로 학습되어 있다. 아래와 결과가 유사하다. lr이 낮아서 2epch가 의미가 있다. device2로 8h 진행되며, 4일때와 달리 checkpoint 26070까지 진행된다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| checkpoint-1000 | 65.22 | 41.13 |
| checkpoint-5000 | 65.31 | 42.47 |
| checkpoint-12000 (1epch) | 64.76 | 43.76 |
| checkpoint-15000 | 64.94 | 43.83 |
| checkpoint-26000 | 65.20 | 44.46 |
| final | 65.23 | 43.68 |

가장 먼저 lr로 문제를 해결한 결과. 오랫동안 고생하다 드디어 문제를 해결했다. 하지만 점수 상승폭이 너무 낮아 이후 다양한 실험을 진행하고 최적의 lr을 찾아나섰다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| checkpoint-1000 | 65.51 | 41.32 |
| checkpoint-4000 | 65.20 | 42.78 |
| checkpoint-6000 (1epch) | 64.89 | 43.08 |
| checkpoint-12000 | 65.29 | 43.89 |
| final | 65.30 | 43.91 |

### 기타 실험

평가시 bos 실험:  
별 영향을 끼치지 않는다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| add bos=True | 65.30 | 43.91 |
| add bos=False (default) | 65.21 | 43.83 |

#### device / packing
lr `3.0e-6`에서 device, packing에 따른 성능 비교:  
당연히 kmmlu는 packing: false에서 성능이 더 좋다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| device4 (5h 20m), packing: false | 65.23 | 46.16 |
| device2 (3h 13m), packing: true | 65.73 | 45.74 |
| device2 (8h), packing: false | 64.87 | 46.91 |

## textbooks 1000 cleaned
전처리해준 textbooks 1000권을 device4, `3.0e-6`, `packing: true`로 진행

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| textbook_dataset-20240816-0414-0 | 65.43 | 41.16 |
| textbook_dataset-20240816-0414-2 | 65.41 | 41.06 |
| textbook_dataset-20240816-0414-2 & 3 | 65.54 | 41.50 |
| textbook_dataset-20240816-0414 (2h, packing: true) | 65.14 | 39.79 |
| textbook_dataset-20240816-0414 (1h, packing: false) | 65.16 | 40.44 |
| dataset-epub (6.8M) | 65.02 | 40.89 |

- textbook_dataset-20240816-0414-0.jsonl 3.3m
- textbook_dataset-20240816-0414-1.jsonl 4.9m
- textbook_dataset-20240816-0414-2.jsonl 45.9m
- textbook_dataset-20240816-0414-3.jsonl 46.6m
- textbook_dataset-20240816-0414.jsonl 100m
- dataset-epub 6.8M 추가로 정제한 데이터 진행

이 데이터셋은 8k 기준으로 맞춰져 있으므로 packing: false일때 원본 그대로 12,509 examples 및 1h에 학습이 끝나지만 packing: true일때는 4k 단위로 정리되고 24,605 examples가 되어 2h 학습이 진행된다.

jsonl 토큰 갯수 카운팅 스크립트:
```python
from transformers import AutoTokenizer
t = AutoTokenizer.from_pretrained('meta-llama/Meta-Llama-3.1-8B-Instruct')

with open('./textbook_dataset-20240816-0414.jsonl', 'r') as json_file:
    jsonl_content = list(json_file)

import json
tokens = []
for j in jsonl_content:
  jj = json.loads(j)
  tt = len(t(jj['text']).input_ids) - 1
  tokens.append(tt)

import numpy as np
np.sum(tokens)
```

### textbooks-context-4k-token-10m
100여권 textbooks 정제 데이터, 2epch 진행

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| textbooks-context-4k-token-10m (lr 1.0e-4) device4 | 57.14 | 23.74 |
| textbooks-context-4k-token-10m (lr 1.0e-4) | 47.87 | 20.45 |
| textbooks-context-4k-token-10m (lr 1.0e-5) | 64.72 | 40.79 |
| textbooks-context-4k-token-10m (lr 5.0e-6) | 65.06 | 41.21 |
| textbooks-context-4k-token-10m (23m, lr 3.0e-6) | 65.18 | 41.54 |
| textbooks-context-4k-token-10m (14m, lr 3.0e-6, packing: true) | 65.21 | 41.39 |
| textbooks-context-4k-token-10m (lr 1.0e-6) | 65.44 | 41.20 |

- lr 1.0e-5에서 OOM이 발생하여 per_device_train_batch_size를 4에서 2로 낮췄다. 3.0e-6에서도 device4는 OOM이 발생해 진행하지 못했다.
- 전자책은 세심하게 단락 단위로 문장을 구분했으나 packing: true로 해도 성능에 차이가 없다.
  - 4k 넘는 문장이 꽤 많기 때문에 packing: true가 더 유용한듯 하다. 그렇지 않다면 학습시 `cutoff_len` 조정이 필요하다.

### textbooks-1294-raw-token-151m
무정제 데이터를 `3.0e-6`으로 진행했으며, unsloth 만큼은 아니지만 성능이 점차 하락하는걸 관찰할 수 있다. 다행히 unsloth와 달리 영어 점수는 거의 떨어지지 않는다.

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| textbooks-1294-raw-token-151m (checkpoint 5000) | 65.48 | 40.22 |
| textbooks-1294-raw-token-151m (checkpoint 9000, 7% 진행) | 65.17 | 39.69 |

## release
textbooks 1000 (token 100m) + textbooks (token 10m), packing: true로 학습에 2h 20m 소요됐다.  
모델명: dnotitia/dna-base-v1-preview-r240924

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| dna-base-all-textbooks-lr-3e6-device4 | 64.97 | 40.18 |
| dna-base-v1-preview-r240924 (5h 20m, kmmlu-train packing: false) | 64.59 | 45.59 |

모델명: dnotitia/dna-base-v1-preview-r240925  
8gpus, deepspeed z2로 학습했다. z0은 OOM으로 실패  

| 모델 | mmlu | kmmlu_direct |
| --- | ----- | ----------- |
| meta-llama/Meta-Llama-3.1-8B | 65.22 | 41.17 |
| dna-base-textbooks-8gpus (only textbooks token 10m) | 65.16 | 41.16 |
| dna-base-v1-preview-r240925 (2h 40m, kmmlu-train) | 65.21 | 45.47 |

### DeepSpeed 실험하면서 textbook 학습

| 모델 | mmlu | mmlu_pro | kmmlu_direct | 
| --- | ----- | ------- | ------------ |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 |
| dataset-epub (6.8M) | 64.90 | 35.12 | 40.38 |

dnotitia/textbook-cleansing-with-parser-ver0.1를 가장 큰 파일만 따로 추출해서 로컬에 정리한게 dataset-epub이다.

### textbook 3단계 학습 (r241025)

`dnotitia/textbook-cleansing-with-parser-midquality-ver0.1` 추가로 전처리 완료. 2 epchs. kmmlu만 packing: false

| 모델 | mmlu | mmlu_pro | kmmlu_direct | tokens | elapsed (8 GPUs, b=4) | loss |
| --- | ----- | ------- | ------------ | ---- | ----------- | -- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 |  |  |  |
| textbook-cleansing-with-parser-midquality-ver0.1 (step 1 of 3) | 64.90 | 35.95 | 40.22 | 809M | 9:03:37 | 2.0209 |
| textbook-cleansing-with-parser-ver0.1 (step 2 of 3) | 64.54 | 35.60 | 40.51 | 9M | 0:06:25 | 1.8807 |
| kmmlu-train-208k-token-150m (step 3 of 3) | 65.10 | 38.75 | 45.45 | 150M | 2:37:32 | 0.8346 |

## Claude 실험
### Claude 학습과 비교

| 모델 | mmlu | mmlu_pro | kmmlu_direct | tokens | elapsed (8 GPUs, b=4) | loss | 
| --- | ----- | ------- | ------------ | ---- | ----------- | -- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 |  |  |  |
| kmmlu-train-208k-token-150m (step 3 of 3) | 65.10 | 38.75 | 45.45 | 150M | 2:37:32 | 0.8346 |
| dnotitia/kmmlu-claude-3.5-sonnet-train-208k-token-91m<br>(base: textbook-cleansing-with-parser-ver0.1 (step 2 of 3)) | 64.15 | 39.07 | 46.76 | 91M | 2:40:10 (6 GPUs) | 0.7147 |
| dnotitia/kmmlu-claude-3-haiku-train-208k-token-78m<br>(base: textbook-cleansing-with-parser-ver0.1 (step 2 of 3)) | 64.14 | 39.07 | 47.37 | 78M | 5:21:00 (6 GPUs, b=2) | 0.6522 | 
| dnotitia/kmmlu-claude-3-haiku-train-208k-token-78m<br>(base: textbook-cleansing-with-parser-ver0.1 (step 2 of 3)) | 64.71 | 39.54 | 46.74 | 78M | 2:37:45 (6 GPUs) | 0.6686 | 
| dnotitia/kmmlu-claude-3-haiku-test-35k-token-13m<br>(base: kmmlu-claude-3.5-sonnet-train-208k-token-91m) | 64.47 | 38.58 | 54.32 | 13M | 0:27:05 (6 GPUs) | 0.6492 |

### HAERAE-HUB 데이터

| 모델 | mmlu | mmlu_pro | kmmlu_direct | tokens | elapsed (8 GPUs, b=4) | loss | 
| --- | ----- | ------- | ------------ | ---- | ----------- | -- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 |  |  |  |
| textbook-cleansing-with-parser-midquality-ver0.1 (step 1 of 3) | 64.90 | 35.95 | 40.22 | 809M | 9:03:37 | 2.0209 |
| HAERAE-HUB/KOREAN-SyntheticText-1.5B | 64.37 | 36.50 | 40.09 | 1.46B | 15:23:15 | 0.9799 |
| HAERAE-HUB/KOREAN-SyntheticText-1.5B<br>(base: KOREAN-WEBTEXT) | 63.59 | 35.41 | 40.54 | 1.46B | 20:43:08 (6 GPUs) | 0.936 |
| HAERAE-HUB/KOREAN-WEBTEXT | 63.72 | 34.07 | 40.56 | 2.22B | 1 day, 7:22:59 (6 GPUs) | 1.7314 |
| textbook-cleansing-with-parser-midquality-ver0.1<br>(base: KOREAN-SyntheticText-1.5B, KOREAN-WEBTEXT) | 63.55 | 33.72 | 40.76 | 809M | 8:29:41 | 1.92 |
| dnotitia/kmmlu-claude-3.5-sonnet-train-208k-token-91m<br>(base: KOREAN-SyntheticText-1.5B) | 63.81 | 38.72 | 46.36 | 91M | 2:45:22 (6 GPUs) | 0.6997 |
| dnotitia/kmmlu-claude-3.5-sonnet-train-208k-token-91m<br>(base: KOREAN-SyntheticText-1.5B, KOREAN-WEBTEXT) | 63.04 | 36.99 | 46.95 | 91M | 2:44:26 (6 GPUs) | 0.6777 |
| dnotitia/kmmlu-claude-3-haiku-train-208k-token-78m<br>(base: KOREAN-SyntheticText-1.5B) | 64.40 | 38.81 | 46.48 | 78M | 2:40:42 (6 GPUs) | 0.6572 |
| dnotitia/kmmlu-claude-3-haiku-train-208k-token-78m<br>(base: kmmlu-claude-3.5-sonnet-train-208k-token-91m, KOREAN-SyntheticText-1.5B, KOREAN-WEBTEXT) | 63.30 | 37.00 | 48.49 | 78M | 2:41:45 (6 GPUs) | 0.5592 |

## mergekit 실험

dnotitia/dna-1.0-8B-preview-r241105 모델과 dnotitia/exaone-instruct-llamafied를 LINEAR로 merge했다. 그런데 exaone은 instruct라는 문제가 있다.

| 모델 | mmlu | mmlu_pro | kmmlu_direct | tokens | elapsed (6 GPUs, b=4) | loss | 
| --- | ----- | ------- | ------------ | ---- | ----------- | -- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 |  |  |  |
| dnotitia/dna-1.0-8B-preview-r241105 | 64.14 | 39.07 | 47.37 | 78M | 5:21:00 (b=2) | 0.6522 | 
| dnotitia/kmmlu-claude-3.5-sonnet-train-208k-token-91m<br>(base: KOREAN-SyntheticText-1.5B, KOREAN-WEBTEXT) | 63.04 | 36.99 | 46.95 | 91M | 2:44:26 | 0.6777 |
| dnotitia/kmmlu-claude-3-haiku-train-208k-token-78m<br>(base: KOREAN-SyntheticText-1.5B) | 64.40 | 38.81 | 46.48 | 78M | 2:40:42 | 0.6572 |
| dnotitia/kmmlu-claude-3.5-sonnet-train-208k-token-91m<br>(base: LINEAR) | 23.57 | 0.0 | 16.68 | 91M | 3:00:03 | 1.3804 |

freeze 평가는 의미 없어 보인다.

### SFT 진행

Mix Data or Merge Models 논문에 따라 SFT → (merge) 실험 진행

| 모델 | mmlu | mmlu_pro | kmmlu_direct | tokens | elapsed (6 GPUs, b=4) | loss | 
| --- | ----- | ------- | ------------ | ---- | ----------- | -- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 |  |  |  |
| meta-llama/Meta-Llama-3.1-8B-Instruct	| 68.18	| 41.09 | 41.37 | | | |
| run-20241101-0000-b0-1<br>(base: meta-llama/Meta-Llama-3.1-8B)	| 67.88 | 39.21 | 51.77 | |
| dna-instruct-r241111<br>(base: meta-llama/Meta-Llama-3.1-8B-Instruct)	| 64.97 | 37.84 | 40.92 | | 3:20:04 | 0.5233 |
| dna-instruct-base-r241113<br>(base: meta-llama/Meta-Llama-3.1-8B)	| 61.72 | 33.67 | 39.22 | | 3:20:26 | 0.5253 |

그렇다면 우리 2모델을 SLERP merge 시도
1. 두 학습 모델 merge (하나는 이미 instruct를 기반으로 했으므로 다른 경우)

merge 외에 L3i - L3b 값을 cpt 모델에 element-wise add 하는 실험도 병행해볼 것

### SFT 모델 + llama instruct merge 실험
한글 학습 모델 + 라마 instruct

| 모델 | mmlu | mmlu_pro | kmmlu_direct | gsm8k | 
| --- | ----- | ------- | ------------ | ---- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 | 51.78 |
| meta-llama/Meta-Llama-3.1-8B-Instruct	| 68.18	| 41.09 | 41.37 | 77.02 |
| dnotitia/run-20241128-fc1-sf| 66.72 | 42.12 | 51.35 | 77.10 |
| dnotitia/run-20241128-fc1-sf<br>+ meta-llama/Meta-Llama-3.1-8B-Instruct	| 69.38 | 45.07 | 48.99 | 82.03 |
| dnotitia/run-20241128-p6-sf-8	| 66.66 | 42.62 | 51.53 | 79.22 |
| dnotitia/run-20241128-p6-sf-8<br>+ meta-llama/Meta-Llama-3.1-8B-Instruct	| 69.09 | 44.90 | 48.74 | 81.04 |
| dnotitia/run-20241128-fc1-sf<br>+ meta-llama/Meta-Llama-3.1-8B-Instruct<br>+ NCSOFT/Llama-VARCO-8B-Instruct	| 68.33 | 44.52 | 45.77 | 81.80 |

### 최종 모델 merge 전후 평가

| 모델 | mmlu | mmlu_pro | kmmlu_direct | gsm8k | kobest |
| --- | ----- | ------- | ------------ | ---- | ------- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 | 51.78 | 70.10 |
| meta-llama/Meta-Llama-3.1-8B-Instruct	| 68.18	| 41.09 | 41.37 | 77.02 | 69.45|
| slm-dn-dpo-011/run-20241128-p10-sf	| 66.49 | 40.56 | 50.99 | 77.86 | 79.42 |
| slm-dn-dpo-011/run-20241128-p10-sf-merged | 68.97 | 44.38 | 48.50 | 81.19 | 77.51 |
| slm-dn-dpo-011/run-20241128-p6-sf-8	| 66.66 | 42.62 | 51.53 | 79.22 | 80.89 |
| slm-dn-dpo-011/run-20241128-p6-sf-8-merged | 69.09 | 44.90 | 48.74 | 81.04 | 78.09 |
| slm-dn-dpo-011/run-20241128-fc2-sf | 66.86 | 42.54 | 51.99 | 78.84 | 81.00 |
| slm-dn-dpo-011/run-20241128-fc2-sf-merged | 69.29 | 45.14 | 49.13 | 82.41 | 78.52 |
| slm-dn-dpo-011/run-20241128-fc2-sf-slerp-merged | 69.00 | 44.78 | 50.45 | 82.86 | 79.47 |

### merged 모델 SFT 후 다시 merge

| 모델 | mmlu | mmlu_pro | kmmlu_direct | gsm8k | kobest |
| --- | ----- | ------- | ------------ | ---- | ------- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 | 51.78 | 70.10 |
| meta-llama/Meta-Llama-3.1-8B-Instruct	| 68.18	| 41.09 | 41.37 | 77.02 | 69.45|
| sllm-llam31-finetune-0008-merge/run-20241128-fc1-sf-merged--0002	| 68.75	|  | 53.14 | 78.92 | 82.78 |
| run-20241128-fc1-sf-merged--0002-slerp-merged-1	| 69.12 | 43.30 | 47.99 | 80.43 | 77.87 |
| run-20241128-fc1-sf-merged--0002-slerp-merged-2	| 69.47 | 43.23 | 45.35 | 80.59 | 74.83 |
| run-20241128-fc1-sf-merged–0002-linear-0.5-merged	| 69.18 | 44.19 | 48.74 | 80.66 | 78.27 |
| run-20241128-fc1-sf-merged–0002-linear-0.2-merged	| 69.04 | 43.60 | 50.53 | 80.51 | 79.43 |
| run-20241128-fc1-sf-merged–0002-linear-0.1-merged	| 69.00 | 43.61 | 51.23 | 80.36 | 79.89 |
| run-20241128-fc1-sf-merged–0002-linear-0.05-merged	| 68.81 | 43.44 | 51.52 | 79.98 | 80.10 |
| run-20241128-fc1-sf-merged–0002-linear-0.01-merged	| 68.67 | 43.44 | 51.72 | 79.83 | 80.21 |
| run-20241128-fc1-sf-merged–0002-linear-ours-0.5-merged	| 69.16 | 43.14 | 45.06 | 81.12 | 73.85 |
| run-20241128-fc1-sf-merged–0002-linear-ours-0.2-merged	| 68.90 | 41.88 | 43.43 | 78.99 | 71.72 |
| run-20241128-p6-sf-8-linear-0.05-merged | 66.74 | 43.42 | 51.57 | 80.13 | 80.71 |

merged 모델 이후 SFT를 진행해서 kmmlu와 kobest는 올렸으나 다시 gsm8k가 떨어졌다. 따라서 SLERP 방식으로 한 번 더 merge 진행. 1은 base 모델이 llama, 2는 우리 모델이다.