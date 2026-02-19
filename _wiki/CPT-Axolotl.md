---
layout: wiki 
title: CPT Axolotl
tags: ["LLM Training"]
last_modified_at: 2026/02/19 14:22:00
last_modified_history:
  - 2025/01/10
---

- [설치](#설치)
- [실행](#실행)
- [학습](#학습)

# 설치
기존에 아래와 같은 어려움이 있었으나, pytorch 2.5설치, torchvision==0.17.0 설치 (torch 2.2로 자동 다운그레이드), flash-attn 재설치 후 문제 없이 실행된다.

[axolotl 홈](https://github.com/axolotl-ai-cloud/axolotl)
```
git clone https://github.com/axolotl-ai-cloud/axolotl.git
cd axolotl
pip3 install packaging ninja
pip3 install --no-build-isolation -e '.[flash-attn,deepspeed]'
```

---
패키지를 하나씩 설치하려 했으나 너무 많다. 그냥 소스 설치로 자동 설치되도록 처리했으나,  
```bash
$ cd /home/jovyan/axolotl
$ pip install -e '.[flash-attn,deepspeed]'
$ PYTHONPATH=/home/jovyan/axolotl/src accelerate launch -m axolotl.cli.train dna-base-cpt.yaml
```

이후에도 실행이 되지 않고(bitsandbytes 에러) 설치에 어려움이 많아 기존에 제민님이 빌드한 이미지 harbor에서 땡겨서 사용했다. C++ 라이브러리 의존성이 있어서 설치는 llama-factory가 좀 더 수월한편.

`examples/llama-3/fft-8b.yaml` 수정해서 사용

# 실행

학습시 다음 에러가 발생했는데, 레퍼런스를 찾을 수 없었고 `pip install --upgrade transformers`로 문제를 해결할 수 있었다. axolotl은 up-to-date 버전.
```
TypeError: Trainer.compute_loss() got an unexpected keyword argument 'num_items_in_batch'
```

학습 타입을 지정하는 llama-factory와 달리 데이터셋과 타입을 지정하면 학습이 진행된다.

# 학습

| 모델 | mmlu | mmlu_pro | kmmlu_direct | tokens | elapsed (8 GPUs, b=4) |
| --- | ----- | ------- | ------------ | ------ | --------------------- |
| meta-llama/Meta-Llama-3.1-8B | 65.26 | 35.63 | 41.04 |  |  |
| textbook-cleansing-with-parser-midquality-ver0.1 (step 1 of 3) | 64.74 | 35.67 | 40.05 | 809M | 9:30:05 |
| textbook-cleansing-with-parser-ver0.1 (step 2 of 3) | 64.81 | 35.39 | 40.14 | 9M | 0:07:05 |
| kmmlu-train-208k-token-150m (step 3 of 3) | 64.82 | 38.55 | 43.36 | 150M | 7:23:47 |

llama-factory 학습과 다른 점은 eval을 0.00001로 설정했고 (오류로 착각하고 낮은 값으로 설정), sample_packing을 모두 false로 했다. `\\n` issue NOT patched. 데이터 jsonl로 변경이 필요하다. 데이터에 오류가 있지만 그래도 kmmlu 점수가 올랐다.

textbook-cleansing-with-parser-midquality-ver0.1 (step 1 of 3):  
```
{'train_runtime': 34205.9233, 'train_samples_per_second': 12.248, 'train_steps_per_second': 0.048, 'train_loss': 2.0703651164738064, 'epoch': 2.0}
1636/1636 [9:30:05<00:00, 20.91s/it]
```

textbook-cleansing-with-parser-ver0.1 (step 2 of 3):  
```
{'train_runtime': 425.5262, 'train_samples_per_second': 10.899, 'train_steps_per_second': 0.042, 'train_loss': 2.0686539345317416, 'epoch': 1.97}
18/18 [07:05<00:00, 23.64s/it]
```

kmmlu-train-208k-token-150m (step 3 of 3):  
```
{'train_runtime': 26627.9743, 'train_samples_per_second': 15.664, 'train_steps_per_second': 0.061, 'train_loss': 0.8294176438809613, 'epoch': 2.0}
1628/1628 [7:23:47<00:00, 16.36s/it]
```

llama-factory에서는 2h 37m에 끝났고, 실제로 토큰수도 훨씬 적은데, 왜 step 1과 비슷한 step에 시간도 오래 걸리는지 모르겠다.