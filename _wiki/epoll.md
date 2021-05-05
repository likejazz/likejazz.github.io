---
layout: wiki 
title: epoll
last-modified: 2021/05/06 08:09:10
---

<!-- TOC -->

- [개요](#개요)
- [epoll](#epoll)
- [링크](#링크)

<!-- /TOC -->

# 개요
Node.js는 원래 libev를 도입했으나 Windows를 지원하지 않아 IOCP도 함께 지원하는 libuv를 개발하고 전환했다.

# epoll
<img src="https://suchprogramming.com/wp-content/uploads/2018/01/poll-times.png" width="70%">[^fn-speed]

Linux는 모든게 file descriptor로 구성되는데, 갯수에 상관없이 일정하게 빠른 속도를 보인다는 큰 장점이 있다.

[^fn-speed]: <https://suchprogramming.com/epoll-in-3-easy-steps/>

epoll을 이용한 simple echo 서버, submodule 구조에 cmake가 적용되어 있다.[^fn-tutorial] `git clone` 후에 `git submodule update --init --recursive`로 submodule을 받았다.

[^fn-tutorial]: <https://github.com/isaacmorneau/simple-epoll>

# 링크
[epoll](https://dev.likejazz.com/post/650260566015885312/epoll)  
C++ 예제는 epoll은 아니고 std::thread를 이용해서 Event Loop를 흉내내고 MQ와 Timer를 구현했다.