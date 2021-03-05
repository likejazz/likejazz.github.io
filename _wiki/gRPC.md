---
layout: wiki 
title: gRPC
last-modified: 2021/03/05 17:27:49
---

<!-- TOC -->

- [Tutorial](#tutorial)
    - [Server: Go(Linux)](#server-golinux)
    - [Client: Java](#client-java)

<!-- /TOC -->

2017년에 Thrift로 실험을 했으나 2021년 현재 gRPC가 완전히 주도권을 잡았다. 사실상 표준 RPC 프로토콜로 활약중이다.

# Tutorial
『gRPC 시작에서 운영까지』 내용에 따라 1장[^fn-1] Go 서버와 Java 클라이언트가 잘 동작한다. 

## Server: Go(Linux)
Go에서는 `go mod init` 후에 proto를 복사하고, `protoc`을 실행한다. protobuf는 `$ sudo apt install protobuf-compiler`로 설치한다. 또는 우분투에서는 `$ sudo snap install protobuf --classic`로 설치할 수 있다.

[^fn-1]: <https://github.com/switchover/grpc-up-and-running/tree/main/ch01>

## Client: Java
(주제와 다른 이슈) `/usr/libexec/java_home -V`에 JDK외에 설치가 확인되어 제대로 된 `JAVA_HOME`을 지정해주지 못하고 있음. `/usr/libexec/java_home -v 1.8.0`로 처리했다.

(마찬가지로 주제와 다른 이슈) 사내망에서 인증서 오류로 `gradle build`가 되지 않는다. 외부망에서 빌드한(서버에서 라이브러리를 받아온) 이후에 사내망 빌드가 가능하다.