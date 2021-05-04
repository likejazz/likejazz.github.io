---
layout: wiki 
title: Deno
last-modified: 2021/05/04 12:37:47
---

<!-- TOC -->

- [선택 이유](#선택-이유)
- [runtime](#runtime)
    - [deno](#deno)
    - [node](#node)

<!-- /TOC -->

# 선택 이유
Deno는 Ryan Dahl의 새로운 프로젝트, secure runtime for *TypeScript*.

C++로 구현했던 Node.js와 달리 초기에는 Go로 구현하려 했고, V8 Go binding 구현 흔적도 남아 있다. 하지만 Go에서 별도의 GC 문제로 인해 Rust로 전환. 애초에 Go로 구현하려고 했던 만큼 Deno에는 Go의 철학이 많이 스며들어 있다. 대표적인게 URL 모듈 구조.

Standard Library도 Go의 영향을 많이 받았다. 2년여간 오픈소스로 진행해오다 드디어 법인을 설립[^fn-deno]했다고. 시드머니로 50억원을 투자 받았다고.

[^fn-deno]: <https://deno.com/blog/the-deno-company>

# runtime
## deno
- `deno`: ts 실행
- `denon`: nodemon처럼 파일 변경시 바로 재시작하는 데몬. deno는 npm 필요 없이 url을 기입하여 바로 모듈 설치가 가능하다. ryan이 발표에서 자랑했던 부분 중 하나.

## node
- `node`: js 실행
- `ts-node`: `npm install -g ts-node`로 설치. `tsc`가 변환 후 `node`로 실행. 원래 `tsc`는 js 변환 역할만 수행.

별개로 노드 모듈 중 하나인 `gnomon` 매우 좋다. global로 설치하여 유용하게 활용 중이다.