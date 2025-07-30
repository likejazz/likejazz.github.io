---
layout: wiki 
title: gRPC
tags: ["Network Programming"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [Tutorial](#tutorial)
    - [Server: Go(Linux)](#server-golinux)
    - [Client: Java](#client-java)
- [Generate code from *.proto](#generate-code-from-proto)
    - [Python](#python)
    - [Go](#go)
- [grpcurl](#grpcurl)
- [기타](#기타)

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

코드 생성은 `gradle build`로 가능하다.

# Generate code from *.proto
## Python
```
$ python -m grpc_tools.protoc \
    -I. \
    --python_out=. \
    --grpc_python_out=. \
    calculator.proto

or

$ brew install protobuf
$ protoc -I. --python_out=. calculator.proto
```
현재 디렉토리에 pb가 생성된다.

## Go
Go는 다소 복잡하다. 다음은 proto 디렉토리 하위에 pb가 생성된다.
```
$ brew install protobuf protoc-gen-go
$ protoc -I. \
    --go_out=plugins=grpc:. \
    --go_opt=Mcalculator.proto=proto/ \
    calculator.proto
```
- `plugins=grpc`는 `UnimplementedXXXServer`, `RegisterXXXServer` wrapper 생성을 위해 필요하다.
- `--go_opt=Mcalculator.proto=proto/` 패키지명을 지정한다. 그렇지 않으면 proto 파일에서 `option go_package`를 지정해야 한다. `main.go`에서 해당 패키지를 `import`하여 사용하게 되며, 반드시 하나 이상의 `/`가 필요하다.
```go
import pb "XXX.com/pi/proto"
```

# grpcurl
proto 파일을 지정하여 다음과 같이 호출할 수 있다.
```
$ brew install grpcurl
$ grpcurl \
    -proto product_info.proto \
    -d '{"name": "Samsung S10", 
        "description": "Galaxy S10", 
        "price": 700.0}' \
    xxx.a.run.app:443 \
    ecommerce.ProductInfo.addProduct | gnomon
```
속도 측정을 위해 gnomon을 파이프로 연결했다.

# 기타
- Cloud Run Python 가이드[^fn-guide] 대로 잘 동작한다.
- gRPC는 docker의 python:3.8-alpine에서는 pip 설치가 진행되지 않았다. slim으로 가능.
- Cloud Run에 올릴때는 `--plaintext` 옵션에 주의한다. 로컬에서는 해당 옵션이 필요하지만 서버에서는 당연히 필요 없기 때문. 게다가 오류도 `Failed to dial target host`로 나오기 때문에 혼동하기 쉽다.

[^fn-guide]: <https://github.com/grpc-ecosystem/grpc-cloud-run-example/tree/master/python>
