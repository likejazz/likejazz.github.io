---
layout: wiki 
title: Windows Subsystem for Linux
last-modified: 2021/04/01 13:30:36
---

<!-- TOC -->

- [개요](#개요)
- [설치](#설치)

<!-- /TOC -->

# 개요

# 설치
- Program Features에서 WSL을 활성화 한다. 현재 VM 방식의 WSL2가 있으나 아직 Windows Insider 프로그램을 통해서만 설치할 수 있다. WSL은 기본적으로 리눅스의 ELF 바이너리를 네이티브로 실행해준다.
- CMD에서 `bash`를 실행하면 Microsoft Store에서 Ubuntu를 설치하도록 연결해준다.
- Command Prompt가 마음에 들지 않았는데 Microsoft Store에 Windows Terminal이 있고, iTerm이 부럽지 않을 정도로 뛰어나다.
- Windows Terminal의 json을 수정하여 default를 Ubuntu로 해두면 리눅스 부럽지 않게 활용 가능하다.
- vscode도 매우 seamless하게 연동된다. ubuntu에서는 code-server를 설치하여 자동으로 연동된다. vscode에서도 wsl extention이 별도로 존재한다.