---
layout: wiki 
title: Open MPI
tags: ["MLOps & HPC"]
last_modified_at: 2022/06/12 15:34:28
---

- [개요](#개요)
  - [RPC와 차이](#rpc와-차이)
- [MPI 테스트](#mpi-테스트)
  - [mpirun 실행](#mpirun-실행)
  - [srun 실행](#srun-실행)
- [실험 방식](#실험-방식)

# 개요
MPI는 Message Passing Interface 저수준 API로 병렬 컴퓨팅을 위한 HPC에 주로 사용된다.

- MPI-1: 1994년에 정의된 인터페이스 표준
- PMIx: PMI-2 이후 PMI Exascale이란 뜻으로 등장한 표준. OpenPMIx 구현체가 있으며, Open MPI에서도 구현. slurm에서도 mpi default는 pmix를 사용한다.

## RPC와 차이
- RPC는 고수준 API
  - C API인 MPI와 gRPC로 동작하는 RPC
- MPI는 `mpirun`등의 프로세스에서 호출하고 관리한다. gRPC는 서버/클라이언트 개발 구조
- MPI는 유사한 컴퓨터 셋에서 실행, 동일 환경 공유. RPC는 환경 공유가 필요 없음.
    - 따라서 MPI는 HPC 클러스터에 사용, gRPC는 서버/클라이언트 모델로 인터넷으로도 서비스 가능.

# MPI 테스트
MPI Hands-On[^fn-mpihandson]을 참고해 다음과 같이 C++ 테스트 코드 작성:

[^fn-mpihandson]: <http://education.molssi.org/parallel-programming/04-distributed-examples/index.html>

```cpp
#include <mpi.h>
#include <iostream>

int main(int argc, char **argv) {
    // Initialize MPI
    int size, rank, name_len;
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Get_processor_name(processor_name, &name_len);

    int N = 200000000;

    // Print a identification
    std::cout << "Host: " << processor_name
              << " rank " << rank
              << " out of " << size
              << " processors." << std::endl;

    // Determine the workload of each ran
    int workloads[size];
    for (int i = 0; i < size; i++) {
        workloads[i] = N / size;
        if (i < N % size) {
            workloads[i]++;
        }
    }
    int my_start = 0;
    for (int i = 0; i < rank; i++) {
        my_start += workloads[i];
    }
    int my_end = my_start + workloads[rank];

    // Initialize a
    double start_time = MPI_Wtime();
    auto *a = new double[N];
    for (int i = my_start; i < my_end; i++) {
        a[i] = 1.0;
    }
    double end_time = MPI_Wtime();
    if (rank == 0) {
        std::cout << "# Elapsed Times" << std::endl;
        std::cout << "- Initialize a : " << end_time - start_time << std::endl;
    }

    // Initialize b
    start_time = MPI_Wtime();
    auto *b = new double[N];
    for (int i = my_start; i < my_end; i++) {
        b[i] = 1.0 + double(i);
    }
    end_time = MPI_Wtime();
    if (rank == 0) {
        std::cout << "- Initialize b : " << end_time - start_time << std::endl;
    }

    // Add the two arrays
    start_time = MPI_Wtime();
    for (int i = my_start; i < my_end; i++) {
        a[i] = a[i] + b[i];
    }
    end_time = MPI_Wtime();
    if (rank == 0) {
        std::cout << "- Add two arrays : " << end_time - start_time << std::endl;
    }

    // Calculate average
    start_time = MPI_Wtime();
    double average = 0.0;
    for (int i = my_start; i < my_end; i++) {
        average += a[i] / double(N);
    }
    if (rank == 0) {
        for (int i = 1; i < size; i++) {
            double partial_average;
            MPI_Status status;
            MPI_Recv(&partial_average, 1, MPI_DOUBLE, i, 77, MPI_COMM_WORLD, &status);
            average += partial_average;
        }
    } else {
        MPI_Send(&average, 1, MPI_DOUBLE, 0, 77, MPI_COMM_WORLD);
    }
//    double partial_average = average;
//    MPI_Reduce(&partial_average, &average, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);

    end_time = MPI_Wtime();
    if (rank == 0) {
        std::cout << "- Calculate average : " << end_time - start_time << std::endl;
    }

    // Print average of two lists
    std::cout.precision(12);
    if (rank == 0) {
        std::cout << "(Average Value : " << average << ")" << std::endl;
    }
    delete[] a;
    delete[] b;

    MPI_Finalize();
    return 0;
}
```

`mpicc`, `mpic++`로 빌드하며, Open MPI 설치 시 함께 설치된다.
```
$ mpic++ test_mpi.cpp -o mpic
```

다음과 같이 MPI를 CMake에도 적용했다. NCCL 용도로 만든 CMakeLists.txt[^fn-cmake]도 참고

[^fn-cmake]: <https://github.com/1duo/nccl-examples>

```cmake
cmake_minimum_required(VERSION 3.22)
project(mpic)

set(CMAKE_CXX_STANDARD 14)

find_package(MPI REQUIRED)

include_directories(${MPI_INCLUDE_PATH})

add_executable(mpic test_mpi.cpp)
target_link_libraries(mpic ${MPI_LIBRARIES})
```

## mpirun 실행
`mpirun`으로 실행:
```
$ mpirun --host gpu01:2,gpu02:2,gpu03:2 \
--mca btl_base_warn_component_unused 0 \
--mca btl_tcp_if_exclude docker0 \
./mpic
Host: gpu03 rank 5 out of 6 processors.
Host: gpu01 rank 0 out of 6 processors.
Host: gpu03 rank 4 out of 6 processors.
Host: gpu02 rank 2 out of 6 processors.
Host: gpu02 rank 3 out of 6 processors.
Host: gpu01 rank 1 out of 6 processors.
# Elapsed Times
- Initialize a : 0.20162
- Initialize b : 0.226402
- Add two arrays : 0.0977537
- Calculate average : 0.242178
(Average Value : 100000001.5)
```

각 노드와 ssh로 no password 접속이 가능해야 하며, 그렇지 않으면 오류도 없이 hang-up 상태로 빠지기 때문에 주의. ec2는 pem 파일 때문에 인증서 위치를 지정해줘야 하는데, `mpirun`에는 그런 옵션이 없다. 이 경우 `~/.ssh/config`에 미리 지정해주면 된다.

```
Host gpu01
HostName 10.1.10.163
User ubuntu
IdentityFile ~/xxx-common.pem

Host gpu02
HostName 10.1.4.37
User ubuntu
IdentityFile ~/xxx-common.pem

Host gpu03
HostName 10.1.15.104
User ubuntu
IdentityFile ~/xxx-common.pem
```

warning 메시지가 발생하기 때문에 `btl_base_warn_component_unused=0`으로 설정했고, docker0 interface를 보지 않도록 `btl_tcp_if_exclude=docker0`을 지정했다.

각 노드에 orted가 뜨고 이 데몬이 host와 패킷 통신을 하고, 노드에 명령도 실행한다.
```
$ sudo ss -tonap
LISTEN   0      128                   0.0.0.0:56881               0.0.0.0:*      users:(("orted",pid=8345,fd=16))
ESTAB    0      0                 10.1.10.163:38104            10.1.2.182:37869  users:(("orted",pid=8345,fd=26)) timer:(keepalive,4min39sec,0)

$ ps uxf
ubuntu    7976  0.3  0.1 4982216 25224 ?       Ssl  02:56   0:00  \_ orted --hnp-topo-sig 0N:1S:XX:x86_64 -mca ess env -mca es
ubuntu    7983  361  2.1 6401120 344376 ?      Rl   02:56   3:15      \_ ./all_reduce_perf -b 100M -e 200M -c 0 -n 1
```

## srun 실행
slurm은 pmix가 default로 동작하며, 우분투에서 `openmpi-bin`등 필요한 패키지만 골라서 설치했더니 bare-metal에서는 warning 메시지와 함께 pmix가 제대로 동작하지 않는다. pyxis를 이용해 Open MPI 이미지[^fn-docker]를 이용하니 문제가 없다.

[^fn-docker]:<https://registry.hub.docker.com/r/mfisherman/openmpi>

```
// Makefile
build: test_mpi.cpp
	mpic++ test_mpi.cpp -o mpic

// Dockerfile
FROM mfisherman/openmpi:latest

WORKDIR /project
ADD Makefile test_mpi.cpp /project/

RUN make

$ docker build . -t openmpi
$ enroot import dockerd://openmpi
```

sqsh 파일 용량도 371M으로 io2 디스크에서 이미지 전송에 1초 정도 소요된다.

```
$ srun -N2 \
--export="OMPI_MCA_btl_tcp_if_include=ens5" \
--container-image=./openmpi.sqsh \
./mpic
Host: gpu01 rank 0 out of 2 processors.
Host: gpu02 rank 1 out of 2 processors.
# Elapsed Times
- Initialize a : 0.586631
- Initialize b : 0.679502
- Add two arrays : 0.305796
- Calculate average : 0.307118
(Average Value : 100000001.5)
```

`srun` 실행시 로컬에서 각 노드에 ssh 접속한 세션은 끊어지는 문제가 있다. (기존 `mpirun`은 ssh로 통신한다) 또한 Open MPI 이미지에서 `btl_tcp_if_exclude` 옵션은 오류가 발생하므로 `btl_tcp_if_include=ens5`를 강제로 지정해야 한다.

slurm은 각 노드에 이미 구동되어 있는 slurmd가 slurmstepd를 구동하고 이 데몬은 host 서버와 소켓통신을 하고 명령도 실행한다.

```
$ ps auxf
root      9278  0.2  0.0 470292  8744 ?        Sl   03:20   0:00 slurmstepd: [836.0]
ubuntu    9288  0.0  0.0   7948   788 ?        S    03:20   0:00  \_ /bin/sleep 200

$ sudo ss -taonp
LISTEN   0      128                 127.0.0.1:47165               0.0.0.0:*      users:(("slurmstepd",pid=9278,fd=15))
LISTEN   0      4096                  0.0.0.0:36883               0.0.0.0:*      users:(("sleep",pid=9288,fd=10),("slurmstepd",pid=9278,fd=10))
ESTAB    0      0                 10.1.10.163:35528            10.1.2.182:41679  users:(("slurmstepd",pid=9278,fd=35))
```

mpirun은 ssh로 orted를 띄우고, srun은 slurmd가 slurmstepd를 띄운다. 

NCCL은 해당 프로세스(`./all_reduce_perf`의 경우 쓰레드를 생성해)가 직접 통신을 한다. PyTorch를 사용하고 있다면 python을 통해 실행하기 때문에 python 프로세스가 쓰레드(Torch의 C++ 모듈이 동작하므로 GIL 영향 밖에 있을 것)를 생성하여 통신하는 형태가 된다.

`htop`에서 쓰레드는 녹색으로 표시된다.

# 실험 방식
원할한 개발을 위해 JetBrains Gateway로 원격 서버에 CLion IDE host에 접속하는 형태로 진행했다. host는 클라이언트에서 Remote Development 진행시 자동으로 설치해주며, 원격 서버의 `~/.cache/JetBrains/RemoteDev`에 저장된다.

srun에 mca 파라미터를 전달하는 방법은 가이드[^fn-mca] 참조. 환경 변수 앞에 `OMPI_MCA_`를 부여하면 된다.

[^fn-mca]: <https://www.open-mpi.org/faq/?category=tuning#setting-mca-params>
