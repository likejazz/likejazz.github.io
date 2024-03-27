---
layout: wiki 
title: Jetson
tags: ["MLOps & HPC"]
last_modified_at: 2024/03/27 19:16:45
---

<!-- TOC -->

- [설정](#설정)
- [도구](#도구)
  - [Transformers](#transformers)
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
    "default-runtime": "nvidia"
}

$ sudo systemctl restart docker
```

[l4t-pytorch](https://github.com/dusty-nv/jetson-containers/tree/master/packages/l4t/l4t-pytorch)로 docker 환경을 구성한다. 공식 NGC에 최신 버전이 없고 직원 개인 페이지에서 관리되고 있다. L4T는 Linux for Tegra이며 이를 이용해 [Docker #1](https://github.com/likejazz/private-links)을 수정해 [Docker #3](https://github.com/likejazz/private-links)으로 설정. 처음에는 기존에 사용하던 패키지를 모두 포함했으나 deepspeed가 설치되지 않아 이후 학습용 패키지는 제거했다.

이외에 dusty-nv가 제공하는 [SLM container](https://www.jetson-ai-lab.com/tutorial_slm.html)를 활용하려면 [System Setup](https://github.com/dusty-nv/jetson-containers/blob/master/docs/setup.md)부터 참고. 디스크 용량이 64G에 불과한 문제가 있으며, [3가지 확장 방식](https://developer.nvidia.com/embedded/learn/jetson-agx-orin-devkit-user-guide/developer_kit_layout.html)을 지원한다.
1. **NVMe SSD card** 가장 빠름. SK Hynix 1TB 이상
2. USB thumb-drive (on any USB port)
3. microSD card

## Transformers
```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TextIteratorStreamer
from threading import Thread

from rich import print

model_name='/dnotitia/gemma-2b-it'
model = AutoModelForCausalLM.from_pretrained(model_name, device_map='cuda')

tokenizer = AutoTokenizer.from_pretrained(model_name)
streamer = TextIteratorStreamer(tokenizer)

query = 'Can I get a recipe for French Onion soup?'

prompt = [{'role': 'user', 'content': query}]
inputs = tokenizer.apply_chat_template(
    prompt,
    add_generation_prompt=True,
    return_tensors='pt'
).to(model.device)

Thread(target=lambda: model.generate(inputs, max_new_tokens=256, streamer=streamer)).start()

print('=' * 60)
print(f'[bold magenta]Prompt:[/bold magenta] [white]{query}[/white]')
print('-' * 60)
for text in streamer:
    print(text, end='', flush=True)
print()
print('=' * 60)
```
[^fn-2]

[^fn-2]: <https://www.jetson-ai-lab.com/tutorial_api-examples.html>

# 기타
- [NVIDIA Jetson Generative AI Lab Tutorial](https://www.jetson-ai-lab.com/tutorial-intro.html) 공식 가이드가 업데이트 되지 않고 이 곳에 별도로 정리
- [Benchmarks](https://www.jetson-ai-lab.com/benchmarks.html)