---
layout: wiki 
title: Jetson
tags: ["MLOps & HPC"]
last_modified_at: 2024/12/17 12:51:01
---

<!-- TOC -->

- [설정](#설정)
- [도구](#도구)
	- [Transformers](#transformers)
	- [SSH](#ssh)
	- [NVMe SSD](#nvme-ssd)
	- [Container](#container)
	- [llama.cpp](#llamacpp)
		- [convert](#convert)
	- [ollama](#ollama)
- [성능](#성능)
- [Flash Jetson Linux](#flash-jetson-linux)
- [기타](#기타)

<!-- /TOC -->

# 설정
[가이드](https://developer.nvidia.com/embedded/learn/get-started-jetson-agx-orin-devkit)대로 초기 설정에 10분 이상 소요. headless configuration으로 monitor, input device없이 진행이 가능하다. USB-C 케이블을 연결해 USB 모뎀으로 진행:
```shell
$ screen /dev/cu.usbmodem1420xxxx 115200
```
가이드에는 `sudo`로 안내하고 있으나 굳이 sudo 할 필요는 없었다.

터미널에서 cols/rows 조정을 위해 `.profile`에 추가:
```shell
...
# Fix for screen session
export TERM=xterm-256color
resize > /dev/null
...
```

`sudo apt install nvidia-jetpack`로 [JetPack](https://docs.nvidia.com/jetson/jetpack/install-jetpack/index.html#package-management-tool)을 설치한다. 9.1G이며 20분 소요. CUDA Toolkit과 cuDNN, TensorRT, NVIDIA container runtime 등을 패키지로 설치해준다.

# 도구
처음에 nvidia-smi가 없어서 당황할 수 있다.
> It is not possible to use nvidia-smi on a Jetson. This program/utility requires a PCI bus. Jetsons have an integrated GPU (iGPU) which is wired directly to the memory controller.[^fn-1]

[^fn-1]: <https://forums.developer.nvidia.com/t/nvidia-smi-not-present-in-jetson-linux/239757>

`jtop` 설치:
```shell
$ sudo apt install python3-pip
$ sudo pip3 install -U jetson-stats
$ sudo systemctl restart jtop.service
$ sudo reboot
```

`/run/jtop.sock`로 host에서 docker로 연동할 수 있다.

docker without root 설정:
```shell
$ sudo usermod -a -G docker $USER
$ sudo chmod 666 /var/run/docker.sock
```

nvidia-docker를 default로 하기 위해 다음 설정:
```shell
# /etc/docker/daemon.json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "default-runtime": "nvidia",
    "data-root": "/models/docker"
}

$ sudo systemctl restart docker
```

처음에는 l4t-pytorch로 docker 환경을 구성했다. 공식 NGC에 최신 버전이 없고 직원 개인 페이지에서 관리되고 있다. L4T는 Linux for Tegra이며 이를 이용해 [#1-3. Dockerfile for L4T PyTorch (ssh version)](/wiki/Private-Links) 설정

이후에 dusty-nv가 제공하는 [jetson-container](https://github.com/dusty-nv/jetson-containers) 활용. [System Setup](https://github.com/dusty-nv/jetson-containers/blob/master/docs/setup.md)부터 참고. 디스크 용량이 64G에 불과한 문제가 있다. [3가지 확장 방식](https://developer.nvidia.com/embedded/learn/jetson-agx-orin-devkit-user-guide/developer_kit_layout.html)을 지원한다.
1. **NVMe SSD card** 가장 빠름. SK Hynix 1TB 이상
2. USB thumb-drive (on any USB port)
3. microSD card

## Transformers
```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TextIteratorStreamer
from threading import Thread

from rich import print

model_name='/gemma-2b-it'
model = AutoModelForCausalLM.from_pretrained(model_name, device_map='cuda')

tokenizer = AutoTokenizer.from_pretrained(model_name)
streamer = TextIteratorStreamer(tokenizer)

prompt = [{'role': 'user', 'content': 'Can I get a recipe for French Onion soup?'}]
inputs = tokenizer.apply_chat_template(
    prompt,
    add_generation_prompt=True,
    return_tensors='pt'
).to(model.device)

Thread(target=lambda: model.generate(inputs, max_new_tokens=256, streamer=streamer)).start()

print('=' * 60)
print(prompt)
print('-' * 60)
for text in streamer:
    print(text, end='', flush=True)
print()
print('=' * 60)
```
[^fn-2]

[^fn-2]: <https://www.jetson-ai-lab.com/tutorial_api-examples.html>

## SSH
외부 접속이 가능하도록 SSH 설정을 진행한다.
```shell
$ sudo apt install openssh-server
```

## NVMe SSD
<img src="/images/2024/IMG_4550.jpg" width="50%">

```shell
$ sudo mkfs.ext4 /dev/nvme0n1
$ sudo mkdir /models
$ sudo chown sangpark /models
$ sudo mount /dev/nvme0n1 /models
# /etc/fstab 수정
# <file system> <mount point>             <type>          <options>                               <dump> <pass>
/dev/root            /                     ext4           defaults                                     0 1
/dev/nvme0n1         /models               ext4           defaults                                     0 2
/models/16GB.swap    none                  swap           sw                                           0 0
```

## Container
다음과 같이 Dockerfile에 먼저 반영 시도:
```Dockerfile
ENV HF_HOME=/models/huggingface \
    HF_HUB_CACHE=/models/huggingface/hub \
    HF_TOKEN=hf_xxxx

# It will be replaced in v5.
ENV TRANSFORMERS_CACHE=/models/huggingface
```

`$ huggingface-downloader google/gemma-2b-it`[^fn-3]

[^fn-3]: <https://github.com/dusty-nv/jetson-containers/tree/master/packages/llm/huggingface_hub>

이후 `run.sh`을 수정해서 매번 필요한 컨테이너를 실행하는 형태로 활용한다. 컨테이너 시작 시간 딜레이가 거의 없다.
```bash
# extra flags
EXTRA_FLAGS="$EXTRA_FLAGS --env HF_HOME=/models/huggingface"
EXTRA_FLAGS="$EXTRA_FLAGS --env HF_HUB_CACHE=/models/huggingface"
EXTRA_FLAGS="$EXTRA_FLAGS --env HF_TOKEN=xxxx"
EXTRA_FLAGS="$EXTRA_FLAGS --env TRANSFORMERS_CACHE=/models/huggingface"
```

base model은 `nvcr.io/nvidia/l4t-jetpack:r35.4.1`를 사용한다. 이외에 최신 버전 활용을 위해 패키지를 직접 빌드해서 사용한다.
```shell
$ ./build.sh --name=xxx/base \
	build-essential \
	additional-pkgs \
	python \
	cmake \
	huggingface_hub
```

전체 실행 스크립트는 다음과 같다.
```bash
#!/usr/bin/env bash
# pass-through commands to 'docker run' with some defaults
# https://docs.docker.com/engine/reference/commandline/run/
ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"

# extra flags
EXTRA_FLAGS="$EXTRA_FLAGS --env HF_HOME=/models/huggingface"
EXTRA_FLAGS="$EXTRA_FLAGS --env HF_HUB_CACHE=/models/huggingface"
EXTRA_FLAGS="$EXTRA_FLAGS --env HF_TOKEN=xxxx
EXTRA_FLAGS="$EXTRA_FLAGS --env TRANSFORMERS_CACHE=/models/huggingface"

# this file shows what Jetson board is running
# /proc or /sys files aren't mountable into docker
cat /proc/device-tree/model > /tmp/nv_jetson_model

set -x

docker run --runtime nvidia -it --rm --network host \
	--volume /tmp/argus_socket:/tmp/argus_socket \
	--volume /etc/enctune.conf:/etc/enctune.conf \
	--volume /etc/nv_tegra_release:/etc/nv_tegra_release \
	--volume /tmp/nv_jetson_model:/tmp/nv_jetson_model \
	--volume /var/run/dbus:/var/run/dbus \
	--volume /var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket \
	--volume /var/run/docker.sock:/var/run/docker.sock \
	--volume /run/jstop.sock:/run/jtop.sock \
	--volume $ROOT/data:/data \
	--volume /home/sangpark:/xxxx \
	--volume /models:/models \
	--device /dev/snd \
	--device /dev/bus/usb \
	$EXTRA_FLAGS \
	"$@"
```

특정 Repository 모두 삭제:
```shell
$ docker rmi $(docker images | grep 'xxxx/transformers' | awk '{print $1":"$2}')
```

## llama.cpp
기본 제공 컨테이너는 버전이 낮아 gemma 모델이 구동되지 않는다. `CUDA_ARCHITECTURE` 버전은 87이다.[^fn-4] 다음과 같이 직접 빌드:
```shell
$ git checkout tags/b2581
$ cmake .. -DGGML_CUDA=on -DGGML_CUDA_F16=1 -DCMAKE_CUDA_ARCHITECTURES=87
$ cmake --build . --config Release --parallel 8
```

[^fn-4]: <https://developer.nvidia.com/cuda-gpus>

실행:
```shell
$ ./main -m /models/gguf/gemma-2b-it-q4_k_m.gguf \
--escape \
--in-prefix "<start_of_turn>user\n" \
--in-suffix "<end_of_turn>\n<start_of_turn>model\n" \
--temp 0 \
--n-gpu-layers 99 \
--instruct \
--ctx-size 4096 \
--verbose-prompt
```

컨테이너 접속 방법: `$ ./run-xx.sh xxxx/llama_cpp`

### convert
EEVE-Korean-Instruct-10.8B-v1.0를 gguf로 convert할 때 missing tokenizer.model 에러가 발생하는데, [다음과 같이 convert-hf-to-gguf.py를 패치](https://github.com/ggerganov/llama.cpp/pull/6443/files)하여 에러를 무시하도록 한다.

## ollama
다음과 같이 llama.cpp를 받는다.
```shell
$ git submodule init
$ git submodule update
```

해당 SHA-1은 llama.cpp에서 다음과 같이 조회 가능하다.
```shell
$ git log --oneline
52604860 (HEAD -> master, tag: b2586, origin/master, origin/HEAD) [SYCL] Disable iqx on windows as WA (#6435)
f87f7b89 flake.lock: Update (#6402)
33a52448 compare-llama-bench.py: fix long hexsha args (#6424)
226e8193 ci: server: verify deps are coherent with the commit (#6409)
c50a82ce readme : update hot topics
37e7854c (tag: b2581) ci: bench: fix Resource not accessible by integration on PR event (#6393)
c342d070 Fedora build update (#6388)
```

빌드:
```shell
# Add `safe.directory` to github repositories.
git config --global --add safe.directory /xxx/ollama
git config --global --add safe.directory /xxx/ollama/llm/llama.cpp

# Build
go generate ./...
go build .
```

ollama에서 한글 처리에 문제가 있어 보인다. 간혹 출력 결과에 알 수 없는 줄바꿈이 포함되며, 한글 입력 또한 완전히 삭제되지 않고 일부 문자가 남아 있다. llama.cpp는 괜찮은데 ollama에서만 문제가 발생한다. 이는 CLI 문제로 보인다.

# 성능
[#2-1. GPU comparison: Jetson Orin, RTX 4080 SUPER](/wiki/Private-Links)

# Flash Jetson Linux
Host PC로 반드시 리눅스가 필요하다. 22.04는 Jetson 6.0 DP만 설치 가능하다. 그 이하 버전은 20.04가 필요하며, 6.0은 여전히 프리뷰 버전이다. [sdkmanager를 이용](https://developer.ridgerun.com/wiki/index.php/NVIDIA_Jetson_Orin/JetPack_5.0.2/Getting_Started/Wizard_Flashing#Step_1:_Set_Board_in_Recovery_Mode)해 모든게 자동으로 진행되지만 USB Read error가 발생. Jetson을 재시작하고 recovery mode 버튼 누른 후 reset 버튼으로 다시 복구 모드에 들어가서 `$ lsusb` 상태가 `Bus 005 Device 003: ID 0955:7023 NVIDIA Corp. APX`인 것을 확인하고 다시 진행하니 해결됐다. 가이드에는 전원을 끈 상태에서 하라고 했는데, 전원이 켜져 있어야 했다.

# 기타
- [NVIDIA Jetson Generative AI Lab Tutorial](https://www.jetson-ai-lab.com/tutorial-intro.html) 공식 가이드에 업데이트 되지 않고 이 곳에 별도 정리