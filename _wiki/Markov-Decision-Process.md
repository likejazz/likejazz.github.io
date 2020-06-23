---
layout: wiki 
title: Markov Decision Process
last-modified: 2020/06/23 14:19:09
---

<!-- TOC -->

- [MDP vs FSM](#mdp-vs-fsm)
- [Reinforcement Learning](#reinforcement-learning)

<!-- /TOC -->

강화학습은 마르코프 의사결정과정으로 정의된 문제를 푸는 것과 같으며, MDP는 다음과 같이 표시한다.
$$S, A, P, R, \gamma$$

# MDP vs FSM
Whilst a Markov chain is a finite state machine, it is distinguished by its transitions being stochastic, i.e. random, and described by probabilities. [^fn-so]

MDP는 확률적 전이라는 점이 다르다.

[^fn-so]: <https://stackoverflow.com/a/4880336>

# Reinforcement Learning
The environment is typically stated in the form of a Markov decision process (MDP), because many reinforcement learning algorithms for this context utilize dynamic programming techniques. [^fn-wiki]

강화 학습의 환경은 MDP 형태로 표현한다. 많은 강화 학습 알고리즘이 다이나믹 프로그래밍을 사용하기 때문이다.

[^fn-wiki]: <https://en.wikipedia.org/wiki/Reinforcement_learning>

The main difference between the classical dynamic programming methods and reinforcement learning algorithms is that the latter do not assume knowledge of an exact mathematical model of the MDP and they target large MDPs where exact methods become infeasible.

강화 학습을 사용하는 후자는 MDP의 정확한 수학적 모델을 가정하지 않고, 정확한 방법을 찾기 어려운 큰 MDP를 대상으로 한다.