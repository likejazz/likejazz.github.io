---
layout: wiki
title: RunPod
tags: ["Cloud"]
last_modified_at: 2026/02/19 14:38:27
last_modified_history:
  - 2024/10/28
---

- [설치](#설치)
- [생성](#생성)
- [접속](#접속)
- [Serverless](#serverless)
- [속도 비교](#속도-비교)
- [AMD](#amd)

# 설치

기본 프로그램 설치 및 설정, vLLM 구동
```bash
apt update && apt install -y vim screen nvtop htop
pip install vllm huggingface_hub[cli]
mkdir -p /workspace/hub
echo 'export HF_TOKEN=xxx' >> /root/.bashrc
echo 'export HF_HUB_CACHE=/workspace/hub' >> /root/.bashrc
export HF_TOKEN=xxx
export HF_HUB_CACHE=/workspace/hub
screen -dmS vllm bash -c 'vllm serve dnotitia/dna-1.0-8B-instruct-preview-r241024 --max-model-len 8192'
```
20G는 `8192`, RTX 4090 (24G)에서 `16384`로 구동

테스트
```bash
$ curl https://xxx.proxy.runpod.net/v1/chat/completions -i \
  -H "Content-Type: application/json" \
  -d '{
     "model": "dnotitia/run-20241010-070618",
     "stream": true,
     "max_tokens": 512,
     "frequency_penalty": 1.5,
     "messages": [{"role": "user", "content": "우리나라 대통령이 누구야?"}]
   }'
```

# 생성
Volume Disk는 40G, Edit Pod에서 Expose 포트는 8000으로 수정

# 접속

외부 포트 노출은 Edit Pod에서 HTTP Expose 기입한 다음,  
`https://xxx.proxy.runpod.net` 형태로 호출한다.

HTTP Expose 추가시 Pod을 재시작한다.

# Serverless
매우 흥미롭고 Scale Out 문제에서 해방될 것 같지만 아쉽게도 Network Latency 문제가 있어 응답이 한꺼번에 뭉쳐서 나온다. 아무리 찾아봐도 이 문제를 해결할 수 없다. 따라서 Serverless는 사용하지 않는다.

어차피 vLLM은 Continuous Batching을 지원하기 때문에 `max_num_batched_tokens=2048`에 따라 실시간 인퍼런스를 지원한다. 기본 프롬프트로 10명 동시 접속해도 속도 저하가 없었다.

# 속도 비교
Llama 3.1 8B BF16 (default), vLLM

**Workstation GPUs**  
- RTX A4500 (20G, $0.35): **34 t/s**
- ~~RTX A5000 (24G, $0.43)~~: **42 t/s**
- RTX A6000 (48G, $0.76): **41 t/s**
- RTX 4000 (16G, $0.32): OOM
- RTX 4000 Ada (20G, $0.38): **21 t/s**
- RTX 6000 Ada (48G, $1.03): **52 t/s**

**Desktop GPUs (Gaming)**  
- RTX 3090 (24G, $0.43): **48 t/s**
- RTX 4090 (24G, $0.69): **56 t/s**

**Data Center GPUs**  
- A40 (48G, $0.39): **34 t/s**
- L4 (24G, $0.43): **16 t/s**
- L40 (48G, $1.03): **44 t/s**
- L40S (48G, $1.03): **45 t/s**
- A100 PCIe (80G, $1.64): **79 t/s**
- A100 SXM (80G, $1.89): **81 t/s**
- H100 PCIe (80G, $2.69): **91 t/s**
- H100 NVL (96G, $2.79): **123 t/s**
- H100 SXM (80G, $2.99): **130 t/s**

**AMD**  
- MI300X (192G, $3.49): Installation Failed

AMD MI300X은 vllm의 pip 설치로 안되고 소스 설치해야 하는데, 가이드의 ROCm 버전은 6.2로 RunPod의 설치된 버전 5.7과 다르다. 또한 triton등 사전 패키지 설치에 한참 걸린다. 이후 vllm을 ROCm 버전으로 빌드하는데 실패. pytorch도 2.0.1이 설치되어 있어 최신 버전을 설치해야 하는데 이전 ROCm 버전의 다운로드 링크가 모두 깨져있다.

H100 PCIe와 SXM의 속도 차이는 인터페이스의 차이라기 보다는 메모리 속도의 차이다. H100 PCIe는 HBM2e 2TB/s이고 SXM은 HBM3로 3.35TB/s다. H100 NVL에도 HBM3가 들어 있다. 반면 A100은 PCIe와 SXM의 속도 차이가 거의 없는데 둘 다 동일한 HBM2e로 대역폭이 5% 정도만 차이가 난다.

라마 3.1 8B는 16G에서는 default bf16으로 구동되지 않는다. 20G 이상에서 가능하며, Context Length는 8k, 24G에서는 16k까지 가능했다.

# AMD
일단 진행했던 내용을 그대로 남긴다.
```
runpod/pytorch:2.4.0-py3.10-rocm6.1.0-ubuntu22.04

apt update && apt install -y vim screen htop &&
echo 'export HF_TOKEN=xxx' >> /root/.bashrc &&
echo 'export HF_HUB_CACHE=/workspace/hub' >> /root/.bashrc &&
pip install numba scipy huggingface-hub[cli] setuptools_scm transformers pydantic_core annotated_types cloudpickle distro jiter

cd workspace/vllm
# pip install -r requirements-rocm.txt

export ROCM_HOME="/opt/rocm-6.1.0"
export PYTORCH_ROCM_ARCH="gfx90a;gfx942"
VLLM_TARGET_DEVICE=rocm python3 setup.py develop

python -c 'import torch;print(torch.cuda.is_available())'

VLLM_TARGET_DEVICE=rocm vllm serve dnotitia/run-20241010-070618 --max-model-len 8192

config.py 소스 수정
```