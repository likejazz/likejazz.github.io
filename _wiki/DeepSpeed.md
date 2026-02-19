---
layout: wiki 
title: DeepSpeed
tags: ["LLM Training"]
last_modified_at: 2026/02/19 14:22:32
last_modified_history:
  - 2025/08/07
---

- [실행](#실행)
- [적용](#적용)
- [설명](#설명)
- [Trainer 실험](#trainer-실험)
  - [axolotl](#axolotl)
  - [LLaMA-Factory](#llama-factory)

# 실행
```shell
$ deepspeed --num_gpus=2 train_deepspeed.py
or
$ torchrun --nproc_per_node=2 train_deepspeed.py
```
torchrun으로 실행할 경우 `LOCAL_RANK` OS 환경변수 사용

ds_config.json에 batch size, micro batch size가 하드 코딩되어 있어 좀 불편하다.

deepspeed와 torchrun 실행 시 학습 속도에 차이가 많이 난다. 무슨 차이일까.   
batch size를 어떻게 잡아야 제대로 되는 건지 잘 모르겠음. global size로 잡으면 시간이 50s대로 GPU 갯수와 관계없이 계속 동일

```json
  "zero_optimization": {
    "stage": 3,
    "offload_optimizer": {
      "device": "none"
    },
    "offload_param": {
      "device": "none"
    },
```

이렇게 ds_config.json에서 device 설정을 none으로 돌려두니 빠르게 실행됐다.

# 적용
기존 naive pytorch 코드에 비해 다음 수정 사항이 있다.

- `--local_rank` 설정 필요. multi process로 0, 1 이런식으로 셋팅된다. 그리고 초기화:
```python
# Initialize distributed training
deepspeed.init_distributed()
```
- device 분배, optimizer 설정도 DeepSpeed가 모두 관장한다.   
```python
# Initialize DeepSpeed without optimizer (will be created by DeepSpeed)
model_engine, optimizer, _, _ = deepspeed.initialize(
    args=args,
    model=model
)
device = model_engine.device
```
- loss, step 수정   

기존
```python
optimizer.zero_grad()
output = model(data)
output = output.view(-1, ntokens)
loss = criterion(output, targets)

loss.backward()
optimizer.step()
```

변경
```python
output = model_engine(data)
output = output.view(-1, ntokens)
loss = criterion(output, targets)

model_engine.backward(loss)
model_engine.step()
```
- save_checkpoint 모든 노드가 저장해야 진행된다.   
```
model_engine.save_checkpoint(save_dir='./checkpoints', tag='best')
```

# 설명
DeepSpeed 보다는 기초적인 딥러닝 fundamentals에 대한 정리

- loss는 정답과 출력의 차이
- optimizer는 다음 step에 가중치를 어떻게 업데이트할 지에 대한 선택, 처음에는 이게 없어도 무방하며 이 경우 grad에 lr 반영한 값을 가중치 단순 업데이트
  - 당연히 optimizer (여기서는 adamw 적용해봄)로 step한 경우 학습이 훨씬 잘 된다. loss가 더 낮음

# Trainer 실험
## axolotl
**2 GPUs:**  
FSDP (default):  
- micro_batch_size=1, ok

DeepSpeed:  
- z1 실행 실패
- z2 이상하게 cpu offload가 default로 되어 있어 너무 느려 못쓴다.
- z3_bf16 08:53 elapsed
- z3_bf16 batch 1 ok, 08:53 elapsed
- z3_bf16 batch 2 ok
- z3_bf16 batch 3 ok
- z3_bf16 batch 4 oom

**4 GPUs:**  
DeepSpeed:
- z3_bf16 batch 2 ok, 07:50 elapsed
- z3 batch 3 ok
- z3_bf16 batch 3 ok, 07:45 elapsed
- z3 batch 4 oom
- z3_bf16 batch 4 oom

FSDP:  

## LLaMA-Factory
**2 GPUs:**  
- z0: batch 1 oom
- z2: batch 1 oom
- z3: batch 1 ok, 23:40 elapsed
- z3: batch 2 oom

FSDP:  
`$ accelerate config` required(?)
- failed

LoRA는 가능, llama-factory는 z1을 지원하지 않음

**4 GPUs:**  
trainable float32(?), per_device_train_batch_size

- z0: batch 1 oom
- z2: batch 1 ok, 11:14 elapsed
- z2: batch 2 ok, 10:25 elapsed
- z2: batch 3 ok, 09:55 elapsed
- z2: batch 4 ok, 10:51 elapsed
- z2: batch 5 oom
- z3: batch 1 ok, 11:05 elapsed
  - kmmlu로 학습시 batch 1은 gpu utilization을 거의 쓰지 않아서, batch 4로 해서 100% 만들었다.
- z3: batch 4 ok, 09:35 elapsed
- z3: batch 5 oom