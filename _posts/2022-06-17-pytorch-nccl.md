---
layout: post
title: PyTorch의 랑데뷰와 NCCL 통신 방식
tags: ["Deep Learning"]
last_modified_at: 2022/06/17 00:00:00
---

<div class="message">
torch.distributed는 어떤 방식으로 초기화 하며 랑데뷰란 무엇인지, NCCL 통신은 어떤 방식을 통해 진행되는지, 코드와 패킷 검출, 프로세스를 조회하며 원리를 직접 살펴보도록 한다.
</div>

<small>
*2022년 6월 17일 초안 작성*  
</small>

- [내용](#내용)
  - [초기화](#초기화)
  - [랑데뷰(Rendezvous)](#랑데뷰rendezvous)
  - [NCCL 통신](#nccl-통신)
  - [부록: GLOO 통신](#부록-gloo-통신)
- [정리](#정리)
- [References](#references)

# 내용

`torch.distributed` 패키지는 Distributed Data-Parallel Training(DDP)를 비롯해 분산 학습을 가능케 하는 여러 기능을 제공한다. 그렇다면 이들은 어떻게 각 노드를 인식하고 어떤 방법을 이용해 서로 통신을 주고 받을까?

## 초기화

가장 먼저 이들이 어떻게 초기화하는지 살펴보자. `torch.distributed`의 초기화 과정은 다음 함수 호출로 시작된다.

```
dist.init_process_group(backend="nccl", init_method='env://')
```

백엔드는 NCCL, GLOO, MPI를 지원하는데 이 중 MPI는 PyTorch에 기본으로 설치되어 있지 않기 때문에 사용이 어렵고 GLOO는 페이스북이 만든 라이브러리로 CPU를 이용한(일부 기능은 GPU도 지원) 집합 통신(collective communications)을 지원한다. NCCL은 NVIDIA가 만든 GPU에 최적화된 라이브러리로, 여기서는 NCCL을 기본으로 알아보도록 한다. 또한 `init_method` 파라미터는 생략 가능하지만 여기서는 default인 `env://`를 명시적으로 기술해보았다. 

`env://`는 OS 환경변수로 설정을 읽어들인다. 즉 `RANK`, `WORLD_SIZE`, `LOCAL_RANK`, `MASTER_IP`, `MASTER_PORT`라는 이름의 OS 환경변수를 읽어들여 랑데뷰(rendezvous)를 진행하는데, 저 값은 임의로 지정할 수도 있기 때문에 별도의 클러스터 관리도구가 없다면 그냥 소스코드에 하드 코딩으로도 무방하다. 만약 pyxis를 사용한다면 enroot의 hook에 PyTorch 이미지인 경우 저 환경변수를 자동으로 설정해주도록 되어 있기 때문에 신경 쓸 필요가 없다. 또한 PyTorch Lightning을 사용한다면  현재 실행 환경을 스스로 인식하여 적절한 값을 찾아오는 기능이 구현되어 있기 때문에 마찬가지로 신경 쓸 필요가 없다. 아울러 PyTorch Lightning은 Slurm으로 실행한 경우 Slurm의 OS 환경변수(`SLURM_PROCID` 등)를 읽어 저 값에 대응되도록 하여 랑데뷰를 진행하는 기능도 있다.

## 랑데뷰(Rendezvous)

그렇다면 랑데뷰란 무엇인가? PyTorch 공식문서에 따르면[^fn-1] 다음과 같이 정의한다.

[^fn-1]: <https://pytorch.org/docs/stable/elastic/rendezvous.html>

> functionality that combines a <strong>distributed synchronization</strong> primitive with <strong>peer discovery</strong>.

각 노드를 찾는 분산 동기화의 기초 과정인데, 이 과정은 `torch.distributed`의 기능 중 일부로 PyTorch의 고유한 기능 중 하나다. `torch.distributed`는 `MASTER_IP`, `MASTER_PORT`에 저장소로 활용할 데몬을 구동하는데, 저장소에는 여러 형태가 있으나 distributed는 원격으로 접속이 가능해야 하기 때문에 TCP Store의 형태로 열린다. 구체적으로는 C10d TCP Store라고 부르는데 여기서 C10d는 <strong>C</strong>ore <strong>TEN</strong>sor Distributed[^fn-2]의 약자로, TCP Store는 Redis의 역할과 비슷한 Key-Value Store의 형태다. 그러나 Redis와는 용도와 역할이 조금 다른데 TCP Store의 기능을 구체적으로 살펴보면 다음과 같다.

[^fn-2]: <https://github.com/pytorch/pytorch/issues/14850>

1. Key-Value를 C++의 `std::unordered_map`에 저장하며 전체 코드를 리뷰한 결과, 메모리 초과 등 오류 상황에 대한 대응은 구현되어 있지 않았다.
1. `sys/socket.h`를 단순 랩핑하여 소켓 통신하며, 이벤트 방식 등 고성능을 위한 네트워크 기능은 구현되어 있지 않다.
1. 브로드캐스팅 기능이 있다. 특정 키의 밸류가 업데이트 되면 해당 키를 물고 기다리고 있는 모든 클라이언트에게 변경된 값을 브로드캐스팅 하는 기능이 있다.[^fn-3]클라이언트는 특정 키를 물고 있는 별도의 데몬을 띄우고 해당 키의 밸류가 업데이트 될 때까지 기다리고 있다가 값이 넘어오면 그 값을 받아낸다.  

[^fn-3]: <https://www.cnblogs.com/rossixyz/p/15553670.html>

<img src="https://user-images.githubusercontent.com/1250095/231056945-0fcc0818-28dc-41c6-b0d2-e0c9d1179205.png" width="70%">

즉 C10d TCP Store는 Redis와 달리 고성능, 고효율의 Key-Value Store가 아니라 각 노드간 초기화를 위한 최소한의 기능과 이에 더해 브로드캐스팅이 지원되는 특수한 형태의 저장소 데몬으로 볼 수 있다. PyTorch에서는 파이썬 랩퍼도 제공하기 때문에 다음과 같이 파이썬에서 직접 TCP Store에 접속하여 값을 받아올 수도 있다. 만약 `torch.distributed`를 이용해 학습을 진행 중인 상태라면 반드시 TCP Store가 열리게 될 것이고, 이때 `MASTER_IP`와 `MASTER_PORT`를 조회하여 다음과 같이 접속할 수 있다.

(i.e. 여기서는 `MASTER_IP=10.1.10.XXX`, `MASTER_PORT=29500`, `WORLD_SIZE=2`로 가정한다)

```
>>> import torch.distributed as dist
>>> client_store = dist.TCPStore("10.1.10.XXX", 29500, 2, False)
>>> client_store.get("first_key")
```

만약 first_key라는 키에 값이 없다면 값이 들어올 때까지 대기하고, 값이 추가되는 순간 브로드캐스팅을 통해 값을 받아올 수 있게 된다.

또한 랑데뷰 과정이 초기에만 필요한 과정인 만큼 초기에만 서로 통신을 주고 받고, 학습이 진행 중일때는 서로 통신을 전혀 하지 않는다. 이는 실제 학습 진행 중인 서버에서 `MASTER_PORT`의 패킷을 tcpdump로 조회함으로써 검증할 수 있다.

<img src="https://user-images.githubusercontent.com/1250095/231057391-8c83c541-3c63-4492-80de-854762701707.png" width="70%">

학습을 시작할 시점에는 서로 패킷을 주고 받았지만 막상 학습이 시작되면 아무런 패킷도 주고 받지 않는다.

## NCCL 통신

앞서 살펴본 바와 같이 TCP Store는 랑데뷰에 특화된 데몬이자 저장소로, 이외의 다른 작업은 수행하기 어렵다. 또한 패킷 덤프를 통해 살펴본 바와 같이 초기에 랑데뷰를 위한 패킷 교환외에는 중간에 아무런 패킷도 교환하지 않는다. 그렇다면 본격적으로 학습이 시작되면 각 노드간에는 어떻게 서로 NCCL 통신을 주고 받을까?

정답은, <strong>NCCL은 프로세스가 쓰레드를 생성하고 랜덤하게 포트를 열어 1:1로 프로세스 간 직접 통신을 한다</strong>.

<img src="https://user-images.githubusercontent.com/1250095/231098531-7351a358-15df-4c5f-86e7-2f43ec55de4d.png" width="80%">

이와 같이 여러 쓰레드가(pid가 모두 동일하며, srun은 slurmstepd를 통해 자식 프로세스로 python을 실행시킨다) 랜덤하게 포트를 열고 원격 프로세스와 직접 통신한다. 10.1.15.XXX와 10.1.4.XXX는 모두 학습에 참여하는 노드의 IP로, 연결된(ESTAB 상태) 커넥션 수만 100개가 넘는다.

그렇다면 NCCL은 어떻게 서로를 알고 있을까? NCCL의 C API를 살펴보면 맨 처음에 초기화 하는 부분이 있고, 이때 마스터 프로세스를 포인터 변수 id에 담아서 브로드캐스팅 하는 과정이 있다. 이로 인해 각 노드는 시작시점에 이미 마스터의 id를 알고 있다.

```cpp
// Generating NCCL unique ID at one process and broadcasting it to all
if (rank == 0) {
    NCCLCHECK(ncclGetUniqueId(&id));
}
MPICHECK(MPI_Bcast((void *) &id, sizeof(id), MPI_BYTE, 0, MPI_COMM_WORLD));

// Initializing NCCL, group API is required around ncclCommInitRank
// as it is called across multiple GPUs in each thread/process
NCCLCHECK(ncclGroupStart());
for (int i = 0; i < deviceCount; i++) {
    CUDACHECK(cudaSetDevice(0));
    CUDACHECK(cudaStreamCreate(&s[0]));
    NCCLCHECK(ncclCommInitRank(&comm, size, id, rank));
    ...
```

위 코드는 `nccGetUniqueId()`로 추출한 id를 MPI로 브로드캐스팅 하는 샘플 코드인데, PyTorch는 MPI가 기본으로 설치되어 있지 않기 때문에 아마 MPI가 아니라 랑데뷰 과정 중에 id를 브로드캐스팅 할 것이다. 이렇게 브로드캐스팅 이후에는 서로를 알게 되기 때문에 이제 직접 1:1로 소켓 통신이 가능하다. NCCL은 CPU를 거치지 않고 GPU간 고속 통신을 컨셉으로 하고 있음을 떠올려보면 이처럼 프로세스간 직접 통신은 어찌 보면 당연하다고 할 수 있다.

> Each NCCL ranks connects to the process which called ncclGetUniqueId. Then they also connect directly to each other.

실제로 NVIDIA의 직원이자 NCCL을 개발한 Sylvain Jeaugey가 깃헙에 남긴 코멘트를 보면[^fn-5], 각 NCCL rank는 `ncclGetUniqueId()`를 통해 알게된 프로세스에 직접 연결하여 통신한다고 언급한다. 앞서 코드를 통해 설명한 내용과 일치함을 확인할 수 있다.

[^fn-5]: <https://github.com/NVIDIA/nccl/issues/388#issuecomment-694459445>

<img src="https://user-images.githubusercontent.com/1250095/231098945-ec541799-0f89-45e7-b020-362a23971e49.png" width="70%">

각 노드의 htop을 살펴보면 이렇게 십수개의 쓰레드(녹색은 쓰레드를 의미한다)가 생성되어 NCCL 통신에 참여하는 것을 확인할 수 있다. 멀티 프로세스가 아닌 멀티 쓰레드 방식으로 파이썬 프로세스를 통해서 실행되지만 NCCL은 모두 C++ 모듈이므로 GIL의 영향은 받지 않을 것이다. 또한 여기서는 NCCL을 PyTorch가 랩핑하고 이를 또 파이썬이 호출하는 구조여서 python의 이름으로 쓰레드가 구동되지만 만약 C++로 별도 프로그램을 만든다면 해당 프로세스의 이름으로 쓰레드가 생성될 것이다.

그렇다면 이처럼 랜덤으로 포트를 열어 직접 통신을 하는데, 방화벽 문제는 없을까? 당연히 방화벽 문제가 있을 수 있다.[^fn-4] 해당 이슈를 보면 포트를 지정할 수 없는지를 문의하는데, 현재는 포트를 지정하는 기능이 없다. 따라서 서버끼리 방화벽을 거쳐 연동되어 있다면 해당 노드 간에는 NCCL 통신이 어렵다.

[^fn-4]: <https://github.com/NVIDIA/nccl/issues/388>

## 부록: GLOO 통신

그렇다면 GLOO로 통신할 때는 어떨까? NCCL을 테스트 했던 동일한 코드를 백엔드만 GLOO로 바꿔서 실행해보면 다음과 같다.

<img src="https://user-images.githubusercontent.com/1250095/231099615-57724295-6f2f-406d-9794-01abdb74f13c.png" width="80%">

NCCL과 유사하게 쓰레드를 생성하면서 1:1로 포트를 열어 소켓 통신을 하는 것까지는 동일하지만 NCCL과 달리 GLOO는 불과 10여개의 포트만 오픈하며 이 마저도 로컬에서 스스로 소켓을 열어 통신하는게 4개나 되어 전체의 40%에 달한다. NCCL이 모두 원격 노드를 대상으로 하여 100개가 넘는 포트를 열어서 대규모로 통신하는 것과는 대조적이다. 그렇지만 GLOO도 문제 없이 DDP가 동작하며 GPU로 집합 통신도 가능하다. 그 이유는 GLOO가 GPU 기능으로 broadcast와 all-reduce 딱 이 2가지를 지원하는데 DDP도 이 2가지 기능만 이용하기 때문이다. 물론 NCCL 만큼 고속 성능(실험한 DDP 샘플의 경우 NCCL이 1.5배 더 빠름)을 내지는 못하지만 GLOO만으로도 DDP는 충분히 잘 동작한다.

# 정리

`torch.distributed`는 default로 OS 환경변수를 이용해 초기화 하며 `MASTER_IP`, `MASTER_PORT`에 C10d TCP Store를 구동하여 랑데뷰를 진행한다. 이후 학습이 시작되면 각 노드의 프로세스는 쓰레드를 생성하고 포트를 오픈하여 서로 1:1 소켓 통신으로 NCCL 통신을 진행한다.

GLOO 통신도 가능하다. 기본적으로 CPU 통신을 지원하고 GPU로도 DDP가 사용하는 기능은 지원하기 때문에 가능하지만, DDP 샘플을 만들어서 실험해본 결과 NCCL쪽이 1.5배 더 빨랐다. 따라서 부득이한 경우가 아니라면 GPU 백엔드는 당연히 NCCL을 권장한다.

# References