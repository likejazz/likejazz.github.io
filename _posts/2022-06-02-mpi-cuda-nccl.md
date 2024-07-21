---
layout: post
title: MPI, SLURM, CUDA, NCCL의 구조와 관계
tags: ["MLOps & HPC"]
last_modified_at: 2022/06/02 00:00:00
---

<small>
*2022년 6월 2일 초안 작성*  
</small>

- [요약](#요약)
- [MPI](#mpi)
- [MPI vs. RPC](#mpi-vs-rpc)
- [CUDA-Aware MPI](#cuda-aware-mpi)
- [MPI 코드 데모](#mpi-코드-데모)
- [SLURM vs. MPI](#slurm-vs-mpi)
- [AWS 동작 실험 결과](#aws-동작-실험-결과)
- [NCCL](#nccl)
- [NCCL Tests](#nccl-tests)
- [NCCL Tests(Slurm + Pyxis)](#nccl-testsslurm--pyxis)
- [3가지 주요 옵션](#3가지-주요-옵션)
- [Build Tools](#build-tools)
- [CUDA 코드 데모](#cuda-코드-데모)
- [정리](#정리)
- [References](#references)

# 요약
1. MPI
1. RPC 비교
1. MPI 상세 및 데모
1. Slurm 비교 및 실행 방법
1. NCCL
1. CUDA 데모 + NCCL
1. 정리

# MPI
- Message Passing Interface
- Low-level API로 병렬 컴퓨팅을 위한 HPC에 주로 사용
- MPI-1: 1994년에 정의된 인터페이스 표준
- PMIx: PMI-2 이후 PMI Exascale이란 뜻으로 등장한 표준. OpenPMIx 구현체가 있으며, Open MPI에서도 구현
  - Slurm의 MPI default도 PMIx, `--mpi=pmix`

# MPI vs. RPC
- 원격으로 프로세스를 실행한다는 점에서 비슷
  - 그러나 MPI: Low-level C API
  - RPC: HTTP + JSON like(실제로는 Binary 포맷) 이용
    - 대표적으로 gRPC
- MPI는 `mpirun`등의 프로세스에서 호출하고 관리
  - RPC는 서버/클라이언트 개발 구조
- MPI는 유사한 컴퓨터셋의 병렬 컴퓨팅에 이용
  - RPC는 환경을 공유하지 않으며 인터넷으로도 서비스 가능

# CUDA-Aware MPI
NVIDIA에서 2013년 3월에 CUDA-Aware MPI에 대해 소개[^fn-1][^fn-2]
- MPI 구현체 여럿
  - 지금은 Open MPI가 대세인듯
- SLURM과 비슷한 환경
- 아직 NCCL이 존재하지 않던 시대
  - 2015년에 첫 커밋

[^fn-1]: <https://developer.nvidia.com/blog/introduction-cuda-aware-mpi/>
[^fn-2]: <https://www.open-mpi.org/faq/?category=runcuda>

# MPI 코드 데모

[test_mpi.cpp](https://github.com/likejazz/mpi-nccl-examples/blob/master/src/test_mpi.cpp)
- 병렬 실행
- 성능 비교
- MPI Reduce  
  - <https://mpitutorial.com/tutorials/mpi-reduce-and-allreduce/>  
  - <https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/operations.html#reduce>
  - <https://pytorch.org/tutorials/intermediate/dist_tuto.html#collective-communication>

# SLURM vs. MPI
- Slurm은 통신 프로토콜로 MPI를 사용한다. `srun`은 `mpirun`을 대체
  - MPI는 ssh로 `orted` 구동, Slurm은 `slurmd`가 `slurmstepd` 구동
- Slurm은 스케쥴링 제공
- Slurm은 리소스 제한(GPU 1장만, CPU 1장만 등) 가능
- Slurm은 Pyxis가 있어서 enroot를 이용해 docker 이미지 실행 가능

# AWS 동작 실험 결과
docker interface 문제로 고생했으나 다음과 같이 대부분 실험 완료
<img src="https://user-images.githubusercontent.com/1250095/173693110-b123e034-b16b-4c2e-8b29-f955ca3a024c.png" width="70%">

# NCCL
NVIDIA GPU와 네트워크에 최적화된 멀티 GPU, 멀티 노드 통신 라이브러리[^fn-nccl]
- Simple C API
- MPI API 호환
- TensorFlow, PyTorch등에서 이미 지원
- NVIDIA가 초기에는 MPI 밀다가 방향전환?

[^fn-nccl]: <https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/operations.html#reduce>

# NCCL Tests
```
$ mpirun -x NCCL_SOCKET_IFNAME=ens \
	-x LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH \
	--host gpu01,gpu02,gpu03 \
	--mca btl_base_warn_component_unused 0 \
	--mca btl_tcp_if_include ens5 \
	./all_reduce_perf -b 100M -e 110M -c 0 -n 1
```

# NCCL Tests(Slurm + Pyxis)
```
$ srun --gres=gpu:1 \
	--export="OMPI_MCA_btl_tcp_if_include=ens5,NCCL_SOCKET_IFNAME=ens" \
	--nodelist=gpu01,gpu02,gpu03 \
	--container-image=./nccl.sqsh \
	all_reduce_perf -b 100M -e 110M -c 0 -n 1
```

nccl.sqsh는 NGC CUDA에 Open MPI, NCCL 따로 설치한 것(5GB)

# 3가지 주요 옵션
- `OMPI_MCA_btl_tcp_if_include=ens5`: MPI는 ens5를 보라
- `OMPI_MCA_pml=^ucx`: PML framework로 UCX를 사용하지 말라
- `NCCL_SOCKET_IFNAME=ens`: NCCL은 ens로 시작하는 네트워크 인터페이스를 보라

`srun` 실행시 기존 MPI 옵션은 `OMPI_MCA_` 접두사 부여하는 형태

# Build Tools
- `gcc`, `g++`
  - MPI: `mpicc`, `mpic++`
  - CUDA: `nvcc` (LLVM 기반) MPI가 필요한 경우 빌드 옵션에 `-lmpi` 부여

# CUDA 코드 데모
[test_nccl.cu](https://github.com/likejazz/mpi-nccl-examples/blob/master/src/test_nccl.cu)
- CPU 계산을 CUDA로 했을때 100배 빠름
- 병렬 계산[^fn-easier]
- MPI와 함께 사용
- 병목 구간 프로파일링(nvprof)
  - GPU to Host 복사
  - NCCL 통신

[^fn-easier]: <https://developer.nvidia.com/blog/even-easier-introduction-cuda/>

# 정리
1. Slurm을 사용해야 하는 가장 중요한 이유:
   - Pyxis를 이용한 docker 이미지 바로 실행
2. 예전에는 NVIDIA도 MPI를 지지했지만 이제 GPU 통신은 NCCL 추천
   - 그럼에도 NCCL은 여전히 MPI와 함께 쓰인다.
   - 특히 rank/size 정보는 MPI을 통해 얻는다.
3. CUDA는 엄청난 병렬 연산이 가능하다.
   - 하지만 메모리 복사, NCCL 통신에 병목
4. MPI, NCCL 모두 네트워크 설정 문제 겪음, SuperPOD은 문제 없나

# References