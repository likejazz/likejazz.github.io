---
layout: wiki 
title: LLaMA-Factory
tags: ["LLM Training"]
last_modified_at: 2026/02/19 14:43:48
last_modified_history:
  - 2024/11/04
---

- [개요](#개요)
- [실행](#실행)
  - [소스로 실행 및 설치](#소스로-실행-및-설치)
- [트러블슈팅](#트러블슈팅)
- [설치](#설치)
- [코드 수정해서 디버깅](#코드-수정해서-디버깅)
  - [모델 평가 (lm-eval)](#모델-평가-lm-eval)

# 개요
[(비공개) #1-11 llama3_full_cpt.yaml, dataset_info.json](/wiki/Private-Links) 학습 및 데이터 설정

- `preprocessing_num_workers` 기본값 16일 때 tokenizer에서 killed 발생하여 1로 변경했으나 이제(241025) 괜찮아서 다시 16으로 롤백.
- `gradient_accumulation_steps`를 크게 잡으면 batch_size가 커져서 step이 줄어드는데 속도는 더 걸리기 때문에 전체 속도는 차이가 없다.

# 실행
`$ llamafactory-cli train examples/train_full/llama3_full_cpt.yaml`

## 소스로 실행 및 설치
패키지 설치하지 않고 소스 위치에서 실행할 때 수정사항:
```
LLaMA-Factory/runme.sh
LLaMA-Factory/src/llamafactory-cli
LLaMA-Factory/dna-base-cpt.yaml
LLaMA-Factory/data/dataset_info.json

$ cd /home/jovyan/LLaMA-Factory
$ pip install -r requirements.txt  # Dockerfile에서 미리 진행하여 생략 가능
$ ./runme.sh
```

`runme.sh`와 `llamafactory-cli`는 다음과 같다.

```
$ cat runme.sh
#!/bin/bash
export MASTER_ADDR=127.0.0.1
export MASTER_PORT=20001
export PYTHONPATH=/home/jovyan/LLaMA-Factory/src

./src/llamafactory-cli train dna-base-cpt.yaml

SLACK_TOKEN=xxx
curl -d "text=llama-factory train COMPLETE" \
    -d "channel=xxx" \
    -H "Authorization: Bearer $SLACK_TOKEN" \
    -X POST https://slack.com/api/chat.postMessage

---

$ cat llamafactory-cli
#!/usr/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from llamafactory.cli import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
```

# 트러블슈팅
- 2 GPUs에서 torchrun이 실행되다가 killed되고 log도 없기 때문에 원인을 알 수가 없었다. 
  - kubeflow에서 minimum 24CPU/128G로 넉넉하게 설정 후 더 이상 killed 없음.
- 1장은 CUDA out of memory발생하며 deepspeed도 동작하지 않음
  - 2 GPUs에서 deepspeed z3로 학습이 진행된다. [원래](https://huggingface.co/spaces/Vokturz/can-it-run-llm) 라마3 8B 모델에서 fp32는 111.83G, bf16은 55.92G 필요하다고 나오는데 왜 1장에서 안될까.
- torch 이전 버전에서는 torchrun 오류가 발생한다.
  - `export MASTER_ADDR=127.0.0.1`, `export MASTER_PORT=20001` 수동 설정으로 해결

fsdp는 accelerate config로 진행, deepspeed는 yaml 설정으로 진행. 여기서는 fsdp는 성공하지 못했고 deepspeed z3로 학습 성공했다.

# 설치
deepspeed [참고](https://github.com/hiyouga/LLaMA-Factory/issues/5252), 그러나 지금(2410)은 최신 버전으로 잘 동작한다.

설치하지 않아도 무방하며 아래 소스 설치시 llamafactory-cli 정도만 설치:  
```
$ pip install -e ".[torch,metrics]"
```

# 코드 수정해서 디버깅
llamafactory-cli 실행시 torchrun이 다음 명령 실행:
```bash
$ torchrun --nnodes 1 --node_rank 0 --nproc_per_node 4 --master_addr 127.0.0.1 --master_port 20001 /home/jovyan/LLaMA-Factory/src/llamafactory/launcher.py /home/jovyan/LLaMA-Factory/examples/train_full/llama3_full_cpt.yaml
```

디버깅 방법: deepspeed를 주석 처리하고 gpu 1장으로 torchrun 없이 바로 실행되도록 하여 `breakpoint()`로 디버거로 확인

또는  
`FORCE_TORCHRUN=1 CUDA_VISIBLE_DEVICES=0 PYTHONPATH=/home/jovyan/LLaMA-Factory/src ./src/llamafactory-cli train examples/train_full/llama3_full_cpt.yaml` 이렇게 deepspeed 설정한 채로 torchrun으로 해도 `breakpoint()`가 동작한다.

패치 사항 (`_load_single_dataset()` 함수의 맨 마지막, return 직전에 추가):  
`\\n` 문제: `data/loader.py`  
```python
# custom patch for '\\n' issue
def convert_double_backslash_newline(x):
    x['text'] = x['text'].replace('\\n', '\n')
    return x
dataset = dataset.map(convert_double_backslash_newline)
```
text에 `\n`이 포함되어 있을때 두 번 처리하는 문제다. jsonl에서는 문제가 없다.

- 학습 데이터를 하나로 묶어서 3,000개가 numrows 567개로 나오는 문제: yaml에서 `packing: false`로 지정

## 모델 평가 (lm-eval)

- 정상적으로 학습했는데도 kmmlu는 0점, mmlu는 20점대로 형편없이 떨어져서 보니 lm-eval에서 generate시 prompt에 bos가 붙어 있지 않아서 결과가 완전히 이상하게 나왔다. `text='론론론론'`, expected: `text=' C\n\n화학적 성질에` bos 추가 옵션을 추가해 다음과 같이 옵션 정리:
```
--model_args pretrained=$MODEL,dtype=bfloat16,gpu_memory_utilization=$MEMORY_UTIL,max_model_len=2048,max_num_batched_tokens=2048,tensor_parallel_size=$PARALLEL_SIZE,add_bos_token=True
```

나중에 다시 확인해보니 bos는 성능에 별 영향을 끼치지 않았으며, 지나치게 높은 learning rate의 문제였다.

lmeval 디버깅에 사용 CLI:  
```bash
cd lm-eval
./run.sh --model vllm \
    --model_args pretrained=/home/jovyan/LLaMA-Factory/saves/llama3.1-8b/full/pretrain-240918,dtype=bfloat16,gpu_memory_utilization=0.7,max_model_len=2048,max_num_batched_tokens=2048,tensor_parallel_size=1,add_bos_token=True \
    --tasks kmmlu_direct_chemistry \
    --num_fewshot 5 \
    --batch_size auto \
    --seed 42 \
    --write_out \
    --log_samples \
    --output_path /home/jovyan/eval-results
```

수동 평가: [(비공개) #1-13 simple-eval.sh](/wiki/Private-Links) 참고
