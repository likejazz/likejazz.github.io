---
layout: wiki 
title: NeMo
tags: ["LLM Training"]
last_modified_at: 2024/10/24 00:51:06
---

- [이미지 다운로드](#이미지-다운로드)
- [GPT LLM 학습 과정](#gpt-llm-학습-과정)
  - [CKPT to NeMo](#ckpt-to-nemo)
  - [Inferences](#inferences)
  - [P-tuning](#p-tuning)
  - [P-tuning Eval](#p-tuning-eval)
- [NeMo-Megatron을 이용한 multi-nodes 학습](#nemo-megatron을-이용한-multi-nodes-학습)
- [기타](#기타)

# 이미지 다운로드
NGC[^fn-ngc]

[^fn-ngc]: <https://ngc.nvidia.com>
```
$ docker login nvcr.io

Username: $oauthtoken
Password: <Your Key>

$ docker pull nvcr.io/ea-bignlp/bignlp-training:23.01-py3
```

# GPT LLM 학습 과정
[^fn-gpt]

NeMo에서 GPT2 LLM

1. NeMo-Megatron 이미지 사용 `nvcr.io/ea-bignlp/bignlp-training:23.01-py3`
1. 데이터 다운로드
  ```
$ wget https://dumps.wikimedia.org/enwiki/latest/enwiki-latest-pages-articles.xml.bz2  
```
  데이터 파일 크기는 20G
1. 데이터 가공
  ```
$ pip install wikiextractor
$ python -m wikiextractor.WikiExtractor enwiki-latest-pages-articles.xml.bz2 --json
$ find text -name 'wiki_*' -exec cat {} \; > train_data.jsonl
```
  가공 디렉토리  `/HXXXXXXX/NeMo-experiment`
1. HF의 BPE(유니코드 문자가 아닌 바이트 단위 구성으로 간주) Tokenizer 사용. 
    - `gpt2-vocab.json`: 각 문자별 코드 맵핑 테이블
    - `gpt2-merges.txt`: 문자만 배치. 이 파일 기준으로 먼저 tokenize
  아래 실행시 설정하지 않으면 자동으로 hf에서 다운로드하여 `.cache`에 저장한다.
1. 학습 데이터 into memory map format  
  데이터 디렉토리에서 진행
```
$ python /opt/bignlp/NeMo/scripts/nlp_language_modeling/preprocess_data_for_megatron.py \
--input=train_data.jsonl \
--tokenizer-library=megatron \
--tokenizer-type GPT2BPETokenizer \
--json-keys=text \
--dataset-impl mmap \
--append-eod \
--output-prefix=hfbpe_gpt_training_data \
--workers=128
```
  bin, idx 두 바이너리 파일 생성. 학습 데이터 tokenized, 크기도 1/3. DGX Station은 CPU 128개
1. GPT2(124M) 학습 진행(.npy 데이터셋을 자동 생성한다)  
  conf 위치는 `/opt/bignlp/NeMo/examples/nlp/language_modeling/conf/megatron_gpt_config.yaml`
  
```
$ cd /HXXXXXXX/NeMo-experiment
$ CUDA_VISIBLE_DEVICES=0,1,2,3 python /opt/bignlp/NeMo/examples/nlp/language_modeling/megatron_gpt_pretraining.py  \
  --config-path=/opt/bignlp/NeMo/examples/nlp/language_modeling/conf \
  --config-name=megatron_gpt_config \
  trainer.max_steps=300000 \
  model.micro_batch_size=6 \
  model.global_batch_size=192 \
  model.encoder_seq_length=1024 \
  model.data.data_prefix=[1.0,hfbpe_gpt_training_data_text_document]
```
  Display가 4번 device. A100-80G 4장, 55시간 소요  
  재학습 진행 시 checkpoint는 파일명 맨 뒤 last를 인식한다. 2번 이상 resume시 empty 오류 발생하는 문제가 있음.
1. `$ tensorboard --logdir nemo_experiments --bind_all` 6006 포트
2. PyTorch Lightning의 `.ckpt` 체크포인트로 저장

[^fn-gpt]: <https://docs.nvidia.com/deeplearning/nemo/user-guide/docs/en/main/nlp/nemo_megatron/gpt/gpt_training.html>

## CKPT to NeMo
`.nemo` files include the necessary tokenizer models and/or vocabulary files, etc.
```
$ CUDA_VISIBLE_DEVICES=0,1,2,3 python -m torch.distributed.launch --nproc_per_node=1 \
     /opt/bignlp/NeMo/examples/nlp/language_modeling/megatron_ckpt_to_nemo.py \
     --checkpoint_folder=/HXXXXXXX/NeMo-experiment/nemo_experiments/megatron_gpt/checkpoints \
     --checkpoint_name='megatron_gpt--val_loss=2.69-step=301001-consumed_samples=57791808.0.ckpt' \
     --nemo_file_path=/HXXXXXXX/NeMo-experiment/nemo_experiments/megatron_gpt/gpt.nemo \
     --tensor_model_parallel_size=1 \
     --pipeline_model_parallel_size=1 \
     --gpus_per_node=4 \
     --model_type=gpt
```

`torch.distributed.launch`로 실행하지 않으면 `WORLD_SIZE`가 0이 되어 실행되지 않는다. ckpt를 바로 eval에 사용하려고 하니 mismatched input 오류가 발생하여 동작하지 않는다. tokenizer로 인해 다른 곳에서 학습한 nemo를 가져와도 사용이 어렵다. 예전 예제이므로 지금은 torchrun으로 실행하면 될 것이다.

## Inferences
```
$ python /opt/bignlp/NeMo/examples/nlp/language_modeling/megatron_gpt_eval.py \
  gpt_model_file=/HXXXXXXX/NeMo-experiment/gpt.nemo \
  trainer.devices=1 \
  trainer.num_nodes=1 \
  tensor_model_parallel_size=1 \
  pipeline_model_parallel_size=1 \
  server=True
```

curl로 확인 가능:
```
$ curl -X PUT http://localhost:5555/generate \
     -H 'Content-Type: application/json' \
     -d '{
     "sentences": ["The first 200 years of the Joseon era were marked by relative peace. During this period, the Korean alphabet was created by "],
     "tokens_to_generate": 300,
     "temperature": 1.0,
     "add_BOS": true,
     "top_k": 0,
     "top_p": 0.9,
     "greedy": false,
     "all_probs": false,
     "repetition_penalty": 1.2,
     "min_tokens_to_generate": 2
     }' | jq '.sentences'
```

## P-tuning
MLP, LSTM으로 프롬프트 인코더 구성, 같은 크기의 BERT보다 NLU 성능이 더 좋아질 수 있다.

노트북[^fn-prompt-notebook] 참조하여 데이터 가공. 노트북에서는 1 gpu 사용. 콘솔에서 multi gpu 가능.
```
$ python /opt/bignlp/NeMo/examples/nlp/language_modeling/megatron_gpt_prompt_learning.py
```

동일 name으로 실행한 상태라면 restore 하므로 모델 크기가 다르거나 할 때 주의

[^fn-prompt-notebook]: <https://github.com/NVIDIA/NeMo/blob/stable/tutorials/nlp/Multitask_Prompt_and_PTuning.ipynb>

## P-tuning Eval
```python
from nemo.collections.nlp.models.language_modeling.megatron_gpt_prompt_learning_model import MegatronGPTPromptLearningModel
from nemo.collections.nlp.parts.nlp_overrides import NLPDDPStrategy
from omegaconf import OmegaConf
from pytorch_lightning.trainer.trainer import Trainer

cfg = OmegaConf.load('/HXXXXXXX/NeMo-experiment/megatron_gpt_prompt_learning_inference.yaml')

# trainer required for restoring model parallel models
trainer = Trainer(strategy=NLPDDPStrategy(), **cfg.trainer)
assert (
        cfg.trainer.devices * cfg.trainer.num_nodes
        == cfg.tensor_model_parallel_size * cfg.pipeline_model_parallel_size
), "devices * num_nodes should equal tensor_model_parallel_size * pipeline_model_parallel_size"

model = MegatronGPTPromptLearningModel.restore_from(
    restore_path=cfg.virtual_prompt_model_file, trainer=trainer)

test_examples = [
    {"taskname": "sentiment", "sentence": "The company is well positioned in Brazil and Uruguay ."},
    {"taskname": "sentiment", "sentence": "Profit before taxes decreased by 9 % to EUR 187.8 mn in the first nine months of 2008 , compared to EUR 207.1 mn a year earlier ."},
    {"taskname": "sentiment", "sentence": "Finlan 's listed food industry company HKScan Group controlled companies in the Baltics improved revenues by EUR 3.5 mn to EUR 160.4 mn in 2010 from EUR 156.9 mn in the year before ."},
]

response = model.generate(inputs=test_examples, length_params=None)

print("=" * 70)
print('The prediction results of some sample queries with the trained model:')
print("=" * 70)
for result in response['sentences']:
    print(result)
    print("-" * 30)
```

# NeMo-Megatron을 이용한 multi-nodes 학습

1. 컨테이너의 bignlp-scripts를 로컬에 복사(추가로 NeMo도 복사, 수정이 가능하도록) 23.04 기준 NeMo-Megatron-Launcher로 변경됨.
  ```
$ srun -p c-vision -N 1 \
--container-image /bcmgpfs/hyperai/nvcr.io+ea-bignlp+bignlp-training+23.01-py3.sqsh \
--container-mounts /bcmgpfs:/bcmgpfs \
bash -c "cp -r /opt/bignlp/bignlp-scripts /bcmgpfs/hyperai/"
```
2. config 수정
  ```
- conf/config.yaml : container_mounts
- conf/cluster/bcm.yaml : partition
- conf/training/gpt3/126m.yaml : num_nodes(노드 몇 대 쓸지), 데이터셋 지정, create_wandb_logger
  ```
  - `/bcmgpfs/hyperai/bignlp-scripts`가 bignlp_path
  - 노드 갯수는 Batching[^fn-batching] 참고: global_batch_size = micro_batch_size * data_parallel_size(all GPU count)
3. 하드 코딩된 디렉토리 수정
   - `bignlp-scripts/bignlp/core/stages.py` NeMo 디렉토리 변경(로컬로)
   - `NeMo/nemo/collections/nlp/data/language_modeling/megatron/blendable_dataset.py` 데이터셋 크기 assert 실패, 건너뛰도록
   - `biglnlp-scripts/bignlp/core/stages.py` wandb 설정 변경
4. 데이터 쓰기 권한이 필요하여(rw로 오픈) 데이터와 동일 계정으로 진행
5. `python main.py` 실행, sbatch 파일 생성
6. wandb로 모니터링

[^fn-batching]: <https://docs.nvidia.com/deeplearning/nemo/user-guide/docs/en/stable/nlp/nemo_megatron/batching.html#batching>

Troubleshooting 요약:
1. bin,idx 생성 계정과 동일 계정으로 실행해야함. .npy 파일 생성.
2. ~~8노드일 때만 정상적으로 그래프 구성, 12노드로 global_batch_size 조정시 GPU 100% 상태에서 더 이상 진행되지 않음~~(이전에 8노드로 만든 checkpoint을 읽어서 발생한 문제) 그러나 일부 데이터셋 추가시 12노드에서 실행 안되는 문제는 있다.

# 기타
Proxy를 이용한 ssh 접속:  
`~/.ssh/config`

```
Host sshuser-vpn
  Hostname 10.17.XXX.XXX
  Port 22XX
  User sshuser
  IdentityFile /Users/HXXXXXXX/.ssh/id_rsa_rapids_docker
  ProxyCommand ssh -W 10.17.XXX.XXX:22XX airlab-XXX@10.12.XXX.XXX -p 30XXX

$ ssh sshuser-vpn
```