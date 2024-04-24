---
layout: wiki 
title: NCCL
tags: ["MLOps & HPC"]
last_modified_at: 2024/04/22 13:51:02
---

- [NCCL](#nccl)
  - [Dockerfile](#dockerfile)

# NCCL
CUDA NGC에 NCCL이 설치되어 있지 않다. runtime 이상에는 설치되어 있다고 적혀 있지만 libnccl.so가 안보인다. NCCL tests[^fn-nccltests]는 NCCL과 MPI 모두 사용한다.

[^fn-nccltests]: <https://github.com/NVIDIA/nccl-tests>

NCCL Tests는 `mpicc`가 아니라 `nvcc`로 빌드하기 때문에 PyTorch 이미지에서 빌드하려면 `MPI_HOME`이 설정되어 있어야 한다. `$ make MPI=1`을 할 때 빌드 옵션을 보면 `-lmpi`가 부여된다.

```console
$ make MPI=1 NCCL_HOME=/usr/local/cuda MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi
```

`mpirun`으로 다음과 같이 실행 가능하다.
```console
$ mpirun -x NCCL_SOCKET_IFNAME=ens \
-x LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH \
--host gpu01,gpu02 \
--mca btl_base_warn_component_unused 0 \
--mca btl_tcp_if_include ens5 \
./all_reduce_perf -b 100M -e 110M -c 0 -n 1
```

NCCL은 `ncclGetUniqueID` C API를 통해 procid를 얻고 프로세스간 직접 소켓 통신을 한다. 다음과 같이 listening을 하고 있다가 established 상태에서 패킷을 주고 받는 것을 확인할 수 있다.

```
$ sudo ss -tonap
LISTEN   0      128                   0.0.0.0:1025                0.0.0.0:*      users:(("all_reduce_perf",pid=8211,fd=17))
LISTEN   0      4096              10.1.10.163:55979               0.0.0.0:*      users:(("all_reduce_perf",pid=8211,fd=44))
LISTEN   0      4096              10.1.10.163:34353               0.0.0.0:*      users:(("all_reduce_perf",pid=8211,fd=42))
...
ESTAB    0      0                 10.1.10.163:55553             10.1.4.37:48642  users:(("all_reduce_perf",pid=8211,fd=159))
ESTAB    262144 0                 10.1.10.163:39199           10.1.15.104:47862  users:(("all_reduce_perf",pid=8211,fd=116))
ESTAB    0      0                 10.1.10.163:54568           10.1.15.104:45795  users:(("all_reduce_perf",pid=8211,fd=130))
```

## Dockerfile

NGC CUDA에 NCCL을 직접 빌드하여 다음과 같이 이미지를 만들었다.

```dockerfile
FROM nvcr.io/nvidia/cuda:11.7.0-devel-ubuntu22.04

WORKDIR /workspace-container

# NCCL
ADD nccl-local-repo-ubuntu2004-2.12.12-cuda11.6_1.0-1_amd64.deb /workspace-container
RUN dpkg -i nccl-local-repo-ubuntu2004-2.12.12-cuda11.6_1.0-1_amd64.deb
RUN cp /var/nccl-local-repo-ubuntu2004-2.12.12-cuda11.6/nccl-local-90A432C3-keyring.gpg /usr/share/keyrings/
RUN apt update && apt install -y git wget libnccl2 libnccl-dev \
    && rm -rf /var/lib/apt/lists/*

# Open MPI
RUN wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.3.tar.gz \
    && tar xfz openmpi-4.1.3.tar.gz
RUN cd openmpi-4.1.3 && ./configure \
    && make all -j4 \
    && make install

# NCCL tests
RUN git clone https://github.com/NVIDIA/nccl-tests.git
RUN cd nccl-tests && make MPI=1
RUN cp -R /workspace-container/nccl-tests/build/* /usr/local/bin/

# Open MPI examples(made by me)
RUN mkdir -p openmpi
ADD Makefile test_mpi.cpp openmpi/
RUN cd openmpi && make
RUN cp -R openmpi/mpic /usr/local/bin/
```

slurm, pyxis에서는 docker0 interface를 계속해서 바라보는 문제가 있다. MPI/NCCL 모두 네트워크 인터페이스를 강제로 할당해야 정상작동하며 srun 구동시 다음과 같이 MPI/NCCL 옵션을 부여했다.

```console
$ srun --gres=gpu:1 -N2 \
--export="OMPI_MCA_btl_tcp_if_include=ens5,OMPI_MCA_pml=^ucx,NCCL_SOCKET_IFNAME=ens" \
--container-image=./pytorch-nccl-tests.sqsh \
all_reduce_perf -b 100M -e 110M -c 0 -n 1
```

- `OMPI_MCA_btl_tcp_if_include=ens5`: MPI가 `ens5`를 보도록.
- `OMPI_MCA_pml=^ucx`: PML framework가 UCX를 사용하지 않도록. 그렇지 않으면 다음과 같이 Connection refused 오류가 발생한다.
```
1: [1653901933.475958] [gpu02:5215 :0]            sock.c:325  UCX  ERROR   connect(fd=40, dest_addr=172.17.0.1:40703) failed: Connection refused
1: [gpu02:05215] pml_ucx.c:419  Error: ucp_ep_create(proc=0) failed: Destination is unreachable
1: [gpu02:05215] pml_ucx.c:472  Error: Failed to resolve UCX endpoint for rank 0
```
- `NCCL_SOCKET_IFNAME=ens`: NCCL이 ens를 사용하도록. 이렇게 하면 `ens3` 또는 `ens5`를 사용하게 된다.

PyTorch 이미지에서만 유독 UCX 오류가 나기 때문에 pml 옵션에서 ucx를 사용하지 않도록 설정했다.[^fn-ucx] 

[^fn-ucx]: <https://stackoverflow.com/questions/70794773/unknown-fatal-error-while-calling-c-programm-with-spawn-in-mpi4py>

```
$ srun --gres=gpu:1 -N3 --export="OMPI_MCA_btl_tcp_if_include=ens5,OMPI_MCA_pml=^ucx,NCCL_SOCKET_IFNAME=ens" --container-image=./pytorch-nccl-tests.sqsh all_reduce_perf -b 100M -e 110M -c 0 -n 1
# nThread 1 nGpus 1 minBytes 104857600 maxBytes 115343360 step: 1048576(bytes) warmup iters: 5 iters: 1 validation: 0
#
# Using devices
#   Rank  0 Pid  30938 on      gpu01 device  0 [0x00] Tesla T4
#   Rank  1 Pid  24859 on      gpu02 device  0 [0x00] Tesla T4
#   Rank  2 Pid  16393 on      gpu03 device  0 [0x00] Tesla T4
#
#                                                       out-of-place                       in-place
#       size         count      type   redop     time   algbw   busbw  error     time   algbw   busbw  error
#        (B)    (elements)                       (us)  (GB/s)  (GB/s)            (us)  (GB/s)  (GB/s)
   104857600      26214400     float     sum   202872    0.52    0.69    N/A   224616    0.47    0.62    N/A
   105906176      26476544     float     sum   249852    0.42    0.57    N/A   261420    0.41    0.54    N/A
   106954752      26738688     float     sum   253797    0.42    0.56    N/A   236018    0.45    0.60    N/A
   108003328      27000832     float     sum   239908    0.45    0.60    N/A   193760    0.56    0.74    N/A
   109051904      27262976     float     sum   232508    0.47    0.63    N/A   234376    0.47    0.62    N/A
   110100480      27525120     float     sum   250801    0.44    0.59    N/A   224436    0.49    0.65    N/A
   111149056      27787264     float     sum   231014    0.48    0.64    N/A   249133    0.45    0.59    N/A
   112197632      28049408     float     sum   262098    0.43    0.57    N/A   273330    0.41    0.55    N/A
   113246208      28311552     float     sum   297748    0.38    0.51    N/A   286821    0.39    0.53    N/A
   114294784      28573696     float     sum   284853    0.40    0.53    N/A   282749    0.40    0.54    N/A
   115343360      28835840     float     sum   305408    0.38    0.50    N/A   307126    0.38    0.50    N/A
# Out of bounds values : 0 OK
# Avg bus bandwidth    : 0.58536
```

MPI 예제를 CUDA로 계산하면 100배 더 빠르나, `nvprof`로 프로파일링 해보면 `cudaMemcpyDeviceToHost`와 NCCL 통신(`ncclReduce`)에서 병목이 있다. 노드가 늘어날 수록 NCCL 통신 시간도 증가한다.
