---
layout: wiki 
title: Golang
tags:  ["Languages & Framework"]
last_modified_at: 2024/04/19 19:00:00
---

<!-- TOC -->

- [가이드](#가이드)
  - [go generate](#go-generate)
- [빌드 및 실행](#빌드-및-실행)
  - [dep](#dep)
  - [Live Reload](#live-reload)
  - [바이너리](#바이너리)
- [설치](#설치)
  - [GoLand에서 특정 팩키지가 인식되지 않음](#goland에서-특정-팩키지가-인식되지-않음)
- [gopkg.in go get 처리 되지 않는 문제](#gopkgin-go-get-처리-되지-않는-문제)
- [언어](#언어)
  - [Method Overloading in Go](#method-overloading-in-go)
  - [struct 없이 JSON parsing](#struct-없이-json-parsing)
- [트러블슈팅](#트러블슈팅)
  - [EasyJSON](#easyjson)

<!-- /TOC -->

# 가이드
Go 1.13 이후로 모듈구조가 디폴트로 채택되면서 가이드[^fn-guide]가 많이 변경됐다.  

[^fn-guide]: <https://golang.org/doc/code>

`go install`은 `$GOPATH/bin`에 설치된다. `go build`는 해당 디렉토리에 빌드된다. 예전에는 시스템에 `$GOPATH`를 잡고 했는데 이제 `go env`로 확인 가능하다. 변경시 `go env -w` 기능도 2019년에 추가됨. 시스템 환경설정은 필요 없다.

기본 프로그램도 모듈이기 때문에,
```
$ go mod init hyundai.com/hello
```
형태로 셋업한다.

리모트 모듈을 import 했을때는 `go mod tidy`로 자동 설치.

## go generate
```go
//go:generate go run gen.go arg1 arg2
```


# 빌드 및 실행
## dep
그간 glide를 팩키지 매니지먼트 툴로 사용했으나 dep 이용 중. 코드를 분석해서 자동으로 팩키지 관리를 해주는게 인상적. 프로토타입 단계이긴 하나 구글에서 직접 만드는 툴이라 신뢰도도 높다. 하지만 완전 자동으로 문제 없이 동작하진 않았고 몇 가지 손봐줘야 할 부분들이 있다. `dep ensure`를 해도 `Operation not permitted` 같은 에러가 발생하면서 `/var/folders` 밑에 삭제도 안되는 이상한 폴더에 오류가 계속 발생하는 이슈도 있었는데 `Gopkg.toml`을 모두 삭제하고, `dep init -v`로 새로 생성하니 잘 된다.

몇 가지 손봐줘야 할 부분 중 하나는, go-redis-cluster는 옛날 버전이 태깅되어 있는데 최신 버전을 사용하고자 할 경우 `branch = "master"`로 지정해주어야 한다. `-v`를 사용해서 계속 확인하니 이후에는 잘 되는 것으로 보임. 아무 메시지가 안나와서 `-v` 없이 사용은 힘들다.

```
dep ensure -v
```

## Live Reload
Go에서는 데몬 형태로 구동할때 수정 후 매 번 재시작해야 하는 번거로움이 있는데 예전에는 fresh를 사용했고 최근에는 업데이트가 거의 없는듯 하여 [gin](https://github.com/codegangsta/gin)으로 교체. gin은 에러가 날때 외에는 아무 메시지가 나오지 않고 verbose도 없어서 다소 혼동은 있으나 문제없이 잘 동작함. 예전에 fresh는 오래 띄워 놓으면 스스로 메모리 오류로 죽는 문제가 있었음.

2017년 8월 현재 몇몇 제대로 live reloading이 되지 않는 문제로 인해 gin은 더 이상 사용하지 않고 주로 수동으로 재시작하나 가끔은 fresh를 여전히 이용한다.

## 바이너리

| OS | `$GOARCH` | `$GOOS` |
| Mac | arm64 | darwin |
| Linux | amd64 | linux |
| Jetson | arm64 | linux | 

# 설치
## GoLand에서 특정 팩키지가 인식되지 않음
`$GOPATH/src/PROJECT/vendor`에 있는 팩키지였고 Project Structure를 보니 vendor가 excluded되어 있었음(설정한 적 없는거 같은데)
 제거하니까 다시 indexing하면서 인식된다.

# gopkg.in go get 처리 되지 않는 문제
`glide get gopkg.in/yaml.v2`이 안되는 문제가 있는데 https://github.com/Masterminds/glide/issues/449 에 따르면 git 자체의 문제로 보인다. `git config --global http.sslVerify true`를 하면 된다고 되어 있으나 SSL 처리는 그렇게 되는게 맞지만(`false`인 경우,
```
➜  sandbox git clone https://gopkg.in/yaml.v2
Cloning into 'yaml.v2'...
fatal: unable to access 'https://gopkg.in/yaml.v2/': Unknown SSL protocol error in connection to gopkg.in:-9838
```
이렇게 오류 발생) true로 해도 301 오류로 처리되지 않았다.
```
➜  sandbox git clone https://gopkg.in/yaml.v2
Cloning into 'yaml.v2'...
error: RPC failed; HTTP 301 curl 22 The requested URL returned error: 301
fatal: The remote end hung up unexpectedly
```
즉, 뭔가 redirect 처리가 되지 않는 문제이며 https://github.com/niemeyer/gopkg/issues/50 이 문서에서 제시한 `git config --global http.https://gopkg.in.followRedirects true` workaround로 해결 가능했다. 이 설정이 되어 있지 않아도 다른 사람은 문제 없는데 왜 내 경우에만 발생하는지는 별도로 추가 확인이 필요하다.

최종 .gitconfig는 아래와 같은 형태로 구성되었다.
```
[user]
        name = Sang Park
        email = likejazz@gmail.com
[http]
        sslVerify = true
[http "https://gopkg.in"]
        followRedirects = true
```

# 언어
## Method Overloading in Go
Go에는 메소드 오버로딩이 없다. 자바에서 매우 편리하게 사용하던 기능인데 [Go FAQ에 지원하지 않음이 명시](https://golang.org/doc/faq#overloading)되어 있음  
- 타입 매칭(파라미터)이 없으면 메소드 전달이 단순화됨
- 실제로는 혼란스럽고 깨지기 쉽다.
- 이름으로만 매칭되는 타입 일관성 유지는 Go 타입 시스템의 주요 결정 사항

Workaround: [Function and Method Overloading in Golang](http://changelog.ca/log/2015/01/30/golang)

## struct 없이 JSON parsing
```
var v map[string]string
err := json.Unmarshal([]byte(res), &v)
assert.Equal(t, "ERROR", v["STATUS"])
```

# 트러블슈팅
## EasyJSON
JSON을 빠르게 파싱한다고 되어 있으나 이걸로 `unmarshall` 했을때 아무리 struct를 변경해도 반영이 안되어 살펴보니 Auto-Generated 한 wrapper가 있어서 항상 여길 바라보는 문제가 있었다. JSON 포맷을 수정했다면 반드시 wrapper를 재생성 해야 반영된다.
```
# install
go get -u github.com/mailru/easyjson/...

# run
easyjson -all <file>.go
```
https://github.com/mailru/easyjson

*템플릿에서 struct의 데이타가 출력되지 않는 문제가 있어 하나씩 다 확인해보았더니 마지막에 핸들러에서 맵핑할때 반드시 필요한 데이타만 필터링 하는 최종 절차가 있었다. 거기에 필드를 추가하여 템플릿에서 사용할 수 있도록 함. 상기 EasyJSON 과는 관련이 없었던 문제였다.*
