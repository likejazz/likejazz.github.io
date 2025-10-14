---
layout: wiki 
title: Windows Subsystem for Linux
tags: ["Productivity"]
last_modified_at: 2025/10/10 22:17:34
---

<!-- TOC -->

- [개요](#개요)
- [설치](#설치)
- [활용](#활용)
- [CUDA on WSL2](#cuda-on-wsl2)

<!-- /TOC -->

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
- ~~WSL2가 필요하기 때문에 Windows Insider에 참여하여 dev channel의 Preview 버전 설치. 처음에는 이더넷 드라이버가 안잡혔는데, 재부팅 후 잡힌다. 아마 VM 용도로 이더넷이 추가되면서 지연이 발생하는듯.~~ 이제 WSL 2가 기본이다. CUDA Toolkit 업데이트가 WSL 내에서는 driver 설치가 안됐고, 윈도우 드라이버를 설치하니 WSL 2 드라이버도 함께 따라갔다.
- ~~수동 설치도 해보고 이후에는 `$ wsl --install`로 자동 설치도 진행. Ubuntu 20.04 설치. 버전이름이 없는 이미지는 항상 최신 버전으로 지정된다고. 기존에 WSL1이 공존하고 있어 Program Features에서 WSL 제거 후 `$ wsl --install`로 다시 설치.~~
- NVIDIA 가이드[^fn-nvidia]대로 CUDA Toolkit 설치. pytorch에서는 gpu로 인식한다.
- `$ sudo apt install nvtop`해봤으나 `Segmentation Fault` 발생. `nvidia-smi.exe`만 동작한다. 아예 윈도우의 성능 관리자가 더 보기 편했다.
- tensorflow-gpu를 설치하고 Keras로 MNIST convnet 예제를 돌려봤으나 CPU만 100%를 치고 gpu를 인식하지 못함. DirectML 버전으로 설치 진행하려다 중단.
- jupyterlab을 0.0.0.0으로 구동해도 WSL2는 VM 구조라 별도 IP를 갖기 때문에 다른 호스트(macOS)에서 접속 불가능. 동일 윈도우에서만 접속 가능하여 불편하다. Power Shell로 proxy 설정하는 방법이 있으나 복잡하다.

[^fn-nvidia]: <https://docs.nvidia.com/cuda/wsl-user-guide/index.html>
