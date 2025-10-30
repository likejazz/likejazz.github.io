---
layout: wiki 
title: Windows Subsystem for Linux
tags: ["Productivity"]
last_modified_at: 2025/10/24 22:59:40
last_modified_history:
  - 2025/10/24 CUDA Toolkit 재설치
  - 2025/10/17 Windows 11 설치
  - 2021/06/08 Windows 10 설치
---

<!-- TOC -->

- [개인 설정 정리](#개인-설정-정리)
- [개요](#개요)
- [설치](#설치)
- [활용](#활용)
- [CUDA on WSL2](#cuda-on-wsl2)

<!-- /TOC -->

# 개인 설정 정리
- sar 설치

```bash
$ /etc/default/sysstat
ENABLED="true"

sudo systemctl enable sysstat
sudo systemctl start sysstat
```
리소스 모니터링도 `$ sar -u 1 `이 가장 직관적

- ssh

```bash
$ sudo systemctl enable ssh
```

- 설치 패키지

```bash
# Install additonal packages.
apt-get update && \
apt-get install -y \
  vim silversearcher-ag fzf file screen wget curl git htop nvtop btop tree rsync \
  telnet iputils-ping netcat-traditional openssh-server python3 python3-dev pipx npm unison inotify-tools

# Install essential CLIs associated with LLM
pipx install huggingface_hub wandb Pygments
pipx ensurepath

# Install essential NPM packages
npm install -g gnomon
npm install -g @anthropic-ai/claude-code

# Install extremely fast Python package manager
curl -LsSf https://astral.sh/uv/install.sh | sh

# pbcopy
curl -fsSLo pbcopy-linux-amd64.tar.gz \
    https://github.com/skaji/remote-pbcopy-iterm2/releases/latest/download/pbcopy-linux-amd64.tar.gz
tar xf pbcopy-linux-amd64.tar.gz
mv pbcopy /usr/bin/
```

- `.bashrc`

```bash
# Credentials: HuggingFace, GitHub, AWS, ChatGPT, Claude, Gemini, ...
export HF_TOKEN=xxx
export GITHUB_TOKEN=xxx
export OPENROUTER_API_KEY=xxx
export OPENAI_API_KEY=xxx
export ANTHROPIC_API_KEY="xxx"
export AWS_ACCESS_KEY_ID="xxx"
export AWS_SECRET_ACCESS_KEY="xxx"
export SLACK_TOKEN=xxx
export WANDB_API_KEY="xxx"

# Created by `pipx`
export PATH="$PATH:/home/xxx/.local/bin"

# z
source ~/bin/z.sh

# v
alias v="source .venv/bin/activate"

# fzf
source /usr/share/doc/fzf/examples/key-bindings.bash

# Add RVM to PATH for scripting. Make sure this is the last PATH variable change.
export PATH="$PATH:$HOME/.rvm/bin"
```

---

# 개요
cygwin은 POSIX API를 부분 지원하여 바이너리를 새로 빌드해야 하지만 WSL은 리눅스 커널 ABI 호환성을 제공하여 ELF 바이너리를 그대로 실행할 수 있다.

# 설치
- 이제 그냥 `wsl --install`로 하면 WSL 2가 설치된다.
- ~~Program Features에서 WSL을 활성화 한다. 현재 VM 방식의 WSL2가 있으나 아직 Windows Insider 프로그램을 통해서만 설치할 수 있다.~~ WSL은 기본적으로 리눅스의 ELF 바이너리를 네이티브로 실행해준다.
- ~~CMD에서 `bash`를 실행하면 Microsoft Store에서 Ubuntu를 설치하도록 연결해준다.~~
- Command Prompt가 마음에 들지 않았는데 Windows Terminal은 iTerm이 부럽지 않을 정도로 뛰어나다. (2019년 5월 공개)
- Windows Terminal의 json을 수정하여 default를 Ubuntu로 해두면 리눅스 부럽지 않다.

# 활용
- vscode도 매우 seamless하게 연동된다. ubuntu에서는 code-server를 설치하여 자동으로 연동된다. vscode에서도 wsl extention이 별도로 존재한다.

# CUDA on WSL2
- ~~WSL2가 필요하기 때문에 Windows Insider에 참여하여 dev channel의 Preview 버전 설치. 처음에는 이더넷 드라이버가 안잡혔는데, 재부팅 후 잡힌다. 아마 VM 용도로 이더넷이 추가되면서 지연이 발생하는듯.~~ 이제 WSL 2가 기본이다. CUDA Toolkit은 Linux > WSL-Ubuntu로 설치하고, 윈도우에서는 설치하지 않는다.
```
Windows NVIDIA-SMI 581.57                 Driver Version: 581.57         CUDA Version: 13.0
Linux   NVIDIA-SMI 580.102.01             Driver Version: 581.57         CUDA Version: 13.0
```
윈도우에서는 NVIDIA App으로 설치한 GPU 드라이버와 버전이 일치하고, WSL 2에서는 CUDA Toolkit의 버전을 보여준다.

- WSL 2에서 강제로 하위 버전 CUDA Toolkit을 설치했다가 WSL 강제 종료 이슈가 발생하여 한동안 고생했다. WSL 2에서 12.8 CUDA Toolkit을 설치했으나 다음과 같은 오류 발생:
```
[코드 1 (0x00000001)로 프로세스 종료됨]
이제 Ctrl+D 이 터미널을 닫거나 Enter 키를 눌러 다시 시작할 수 있습니다.
오류입니다.
오류 코드: Wsl/Service/E_UNEXPECTED
```
- jupyterlab을 0.0.0.0으로 구동해도 WSL2는 VM 구조라 별도 IP를 갖기 때문에 다른 호스트(macOS)에서 접속 불가능. 동일 윈도우에서만 접속 가능하여 불편하다.
- nvtop이 안됐는데, nvitop을 설치하면서 libnvidia-compute-580-server nvidia-firmware-580-server-580.95.05 nvidia-kernel-common-580-server같은 예전 드라이버 의존성이 있어 함께 설치하고 이 때문에 실행이 안됐다. GPU 드라이버를 윈도우쪽에서 설치하기 때문에 WSL 2에서 nvidia관련은 apt에서 모두 제거했다.