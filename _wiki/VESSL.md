---
layout: wiki 
title: VESSL
tags: ["Cloud"]
last_modified_at: 2026/02/19 14:40:59
last_modified_history:
  - 2024/10/29
---

- [개요](#개요)
- [실험](#실험)
  - [Run](#run)
  - [Workspace](#workspace)
    - [llama-factory](#llama-factory)
    - [vLLM](#vllm)
- [요약](#요약)
  - [장점](#장점)
  - [단점](#단점)
- [결론](#결론)

# 개요
VESSL을 테스트 해보고 장단점을 정리한다.

job을 실행하는 Run과 서버를 제공하는 Workspace가 있다. 이외에 Service, Pipeline이 있는데 Service는 생성되지 않는다.

# 실험
총 4가지 태스크가 제공된다. <https://app.vessl.ai/Dnotitia>

- Run
- Workspace
- Service
- Pipeline

이 중 Service는 생성되지 않는다. 아울러 Pipeline은 Kubeflow의 Pipeline과 유사해보이나 현재는 필요하지 않아 굳이 실험하지 않았다. Run과 Workspace를 중심으로 살펴보고 장단점을 정리한다.

## Run
Run은 yaml을 이용해 서비스를 구성하고 이를 바로 deploy 해주는 기능이다. 샘플 Template을 제공하는데 이 중 Training 관련은 보이지 않는다. 모두 Gradio를 이용한 데모 구동 예제로 그나마도 2개가 모두 실행에 실패했다. 아마 패키지 버전이 올라가면서 설정이 바뀐거 같은데 제대로 버전 관리가 되지 않는것 같다.
<img src="https://lh3.googleusercontent.com/pw/AP1GczOvkcJM7FpVrG1JzOXXaPy2sNl1SFbv7famB0OkN2jSMTzTi9929VeEk53guEqcJO4y6oOC-X0ohBNtivbBIlURDwXyeNCJhhGFvkrpe7FYc2Z7PTE_pDuBPmvNtv0eYEfA05YUec2xBrxFnGUkz_-piA=w1740-h702-s-no-gm?authuser=0" width="80%">

다행히 solar-pro-chatbot은 실행이 됐고, 데모를 구동할 수 있었다. 하지만 이후에는 알수없는 이유로 더 이상 실행되지 않았고, 수동으로 터미널로 접속해서 설정을 건드리니 세션이 강제로 종료됐다. 편리한 기능이지만 yaml 관리가 번거롭고, 데모 구동 외에는 용도가 제한적이다.

## Workspace
### llama-factory
다음과 같이 기본 설치 진행:
```bash
$ apt update
$ apt install tree nvtop vim
```

llama-factory 설치:
```bash
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e ".[torch,metrics]"
```

실행:
```
$ HF_TOKEN=xxx llamafactory-cli train examples/train_full/llama3_full_sft_ds3.yaml
```

학습 성료:
```
***** train metrics *****
  epoch                    =     2.9939
  total_flos               =      730GF
  train_loss               =     0.5927
  train_runtime            = 0:23:55.39
  train_samples_per_second =       2.05
  train_steps_per_second   =      0.512
```

RunPod이 제공해주는 기능과 거의 동일하다. 문제 없이 학습이 잘 된다. 

### vLLM
설치:
```bash
apt update && apt install -y vim screen nvtop htop tree
pip install vllm huggingface_hub[cli]
mkdir -p /root/.cache/huggingface/hub
echo 'export HF_TOKEN=xxx' >> /root/.bashrc
echo 'export HF_HUB_CACHE=/workspace/hub' >> /root/.bashrc
export HF_TOKEN=xxx
screen -dmS vllm bash -c 'vllm serve dnotitia/dna-1.0-8B-instruct-preview-r241024 --max-model-len 8192'
```

문제 없이 서빙이 잘 됐고, 외부 포트도 노출해 데모에도 연결할 수 있었다.

# 요약
## 장점
- VESSL은 RunPod과 동일하거나 또는 조금 더 저렴한 비용으로 RunPod과 동일한 기능을 제공한다. 이 기능이 Workspace이며, 특별히 문제 없이 잘 동작했다. 사실상 RunPod의 괜찮은 대안으로 보인다.
- VESSL은 서버가 모두 국내에 있어서인지 매우 빠르게 동작한다. RunPod은 서버가 미국이나 유럽으로 나오며 당연히 Latency가 느리다.

## 단점
- Run은 아직 불안하게 동작하며 실행이 안되는 경우도 많았으며, 버그도 있었다. 게다가 Serverless도 아니기 때문에 굳이 이렇게 실행할 이유는 없어 보였다.
- RunPod의 경우 외부 포트 노출시 별도의 인코딩 문자열을 제공하는데 반해 VESSL은 특정 IP의 포트를 바로 노출 시키는 보안 문제가 있다.
- 접속 방식 또한 IP에 그대로 접속하는 형태이기 때문에 취약점 발생시 바로 뚫릴 수 있는 잠재적 위험을 안고 있다.
  - RunPod: `ssh xxx@ssh.runpod.io -i ~/.ssh/id_ed25519`
  - VESSL: `ssh -p 30686 root@xx.xx.xx.xx`
- RunPod은 종료해도 삭제되지 않는 디스크를 추가 비용을 받고 제공해서 start/stop이 비교적 자유로운 편인데, VESSL은 종료하면 일부 설정만 남아 있고 기본적으로 데이터가 모두 삭제된다. 매 번 다운로드 해야하므로 상당히 번거롭다.

# 결론
여러 단점에도 불구하고 가장 중요한 Workspace 기능은 문제 없이 동작했으며 RunPod보다 훨씬 더 쾌적하게 이용이 가능했다. 사실상 이 기능 하나 때문에라도 RunPod의 훌륭한 대안으로 보이며 RunPod보다 더 편리하게 사용할 수 있을 것 같다.