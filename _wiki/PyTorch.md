---
layout: wiki 
title: PyTorch
tags: ["Deep Learning"]
last_modified_at: 2024/01/06 23:42:12
---

<!-- TOC -->

- [Distributed Data Parallel(DDP)](#distributed-data-parallelddp)
- [torchrun](#torchrun)
- [C10d(Distributed communication package)](#c10ddistributed-communication-package)
- [distributed Tutorials](#distributed-tutorials)
  - [기타](#기타)
- [apex](#apex)
- [CUDA\_VISIBLE\_DEVICES](#cuda_visible_devices)

<!-- /TOC -->

# Distributed Data Parallel(DDP)
기존 DP는 multithreading이어서 파이썬의 GIL에 영향을 받았다. 따라서, multiprocessing인 DDP가 등장했고, 권장한다. env 모드일 경우 c10d가 gloo,mpi,nccl을 백엔드로 하여 동작한다. pytorch에서 mpi는 built-in 되어 있지 않기 때문에 소스 컴파일이 아니면 사용할 수 없다.

- `torch.distributed as dist`: `torch.multiprocessing as mp`와 함께 Child Proccesses를 구동하여 `tensor()`를 주고받을 수 있다. srun을 이용하면 mp가 필요없으며 multi-nodes 또한 srun 또는 mpirun만 가능하다. c10d library가 gloo또는 nccl을 이용해 통신한다.

DDP 모델로 초기화 할 때 params를 브로드캐스팅[^fn-broadcasting]하여 동일하게 맞추고 시작한다. 초기화 할 때 Reducer를 호출하고 이때 autograd_hook을 걸어둔다. 이후 autograd 엔진이 동작할 때 hook이 작동된다. 모든 노드의 grads를 합산하고 평균을 구한 동일한 grads를 갖게되며 버킷 단위로 패러럴하게 효율적으로 동작한다.

[^fn-broadcasting]: <https://github.com/pytorch/pytorch/blob/164029f783ba52d206862925e9341e6b851179ff/torch/nn/parallel/distributed.py#L647-L654>

# torchrun
`torch.multiprocessing.spawn`이 필요 없이 multiprocessing 처리를 한다. 기존 `python -m torch.distributed.launch` 실행 방식이 `torchrun`으로 개선됐다.
```console
$ python -m torch.distributed.launch --use_env train_script.py

$ torchrun --nproc_per_node=2 train_script.py
```
기존 distributed.launch, 지금의 torchrun은 os 환경변수로 다음 값을 셋팅해준다. PyTorch distributed communication package의 기본 디자인이다.

```
{
    'MASTER_ADDR': 'gpu01', 
    'MASTER_PORT': '29500', 
    'RANK': '1', 
    'WORLD_SIZE': '2'
}
```

default가 `env://`이며 `dist.init_process_group()`로 초기화하려면 반드시 `MASTER_ADDR` 등이 셋팅되어 있어야 한다. 또는 pyxis로 PyTorch 이미지를 이용하면 torchrun이 하는 환경변수 셋팅을 `SLURM_*`에서 읽어서 해준다. 

mpirun에서 torchrun을 실행했더니 multiprocessing에서 hang이 걸린다. mpirun이 multiprocessing 처리를 하므로, 필요한 환경변수를 적용하고 바로 python으로 구동해도 nccl은 동작한다.

# C10d(Distributed communication package)
DDP는 TCP Store가 default로 `torch.distributed`가 기본 29500를 **C10d TCP store** for the rendezvous among worker processes로 열고 해당 포트로 통신한다. 내부적으로는 `std::unordered_map`에 추가/삭제 하는 구조로 되어 있으며, 통신은 `sys/socket.h`를 랩핑하여 사용한다.

다른 kv store와는 다르게, 특정 key의 value가 업데이트 되면 watch key 중인 모든 worker들에게 broadcasting하는 형태로 동작한다. workers는 get을 하면 값이 들어올때 까지 waiting 한다.

NCCL은 프로세스간 직접 소켓통신을 하기 때문에 C10d TCP Store는 초반에 시작할 때와 종료할 때 통신을 주고 받고 중간 과정에 통신을 주고받지 않는다.

# distributed Tutorials
- Single-Machine Model Parallel Best Practices[^fn-single]에서 batch size를 나누는 형태인 Pipelining Model Parallel을 하면 속도가 더 빨라야 하지만 실험할 때는 오히려 훨씬 더 느린 속도가 나왔다.
- Writing Distributed Applications with PyTorch[^fn-writing] 가이드에서는 MNIST Data Parallel로, 매 epoch 단위로 grad까지 구한 후 all reduce, 이후 world_size로 평균을 구한다. 예제는 CPU 예제다.

[^fn-single]: <https://pytorch.org/tutorials/intermediate/model_parallel_tutorial.html>
[^fn-writing]: <https://pytorch.org/tutorials/intermediate/dist_tuto.html>
[^fn-mnistddp]: <https://github.com/kubeflow/examples/blob/master/pytorch_mnist/training/ddp/mnist/mnist_DDP.py>

## 기타
- Basic MNIST Example[^fn-mnistbasic] learning rate를 동적으로 변경하는 `StepLR` 코드를 확인할 수 있다.
- MNIST DDP[^fn-mnistddp] KubeFlow 예제 중에 MNIST를 DDP로 구현한 예제가 있다. GPU로 torchrun으로 잘 동작한다. 그런데, `backend='nccl'`이 동작하지 않아서 all-reduce가 `gloo`로만 동작한다.

[^fn-mnistbasic]: <https://github.com/pytorch/examples/tree/main/mnist>

# apex
pip install로는 import시 error가 발생하여, 소스를 받은 후 `$ pip install -v --disable-pip-version-check .`로 설치했다. 이 경우 nvcc로 직접 빌드를 진행한다.

# CUDA_VISIBLE_DEVICES
How to do check if PyTorch is using the GPU:
```
$ export CUDA_VISIBLE_DEVICES=0,1,2,3
$ python

>>> [torch.cuda.get_device_name(i) for i in range(torch.cuda.device_count())]
['NVIDIA A100-SXM4-80GB', 'NVIDIA A100-SXM4-80GB', 'NVIDIA A100-SXM4-80GB', 'NVIDIA A100-SXM4-80GB']
```