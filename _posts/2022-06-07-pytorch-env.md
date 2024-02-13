---
layout: post
title: PyTorch 이미지가 RANK 환경변수를 얻는 법
tags: ["Deep Learning"]
last_modified_at: 2022/06/07 00:00:00
---

<div class="message">
Slurm + Pyxis를 이용해 PyTorch 이미지를 실행(srun)하면 RANK, WORLD_SIZE등의 OS 환경변수가 자동으로 셋팅된다. 덕분에 torchrun이 필요하지 않고, MASTER_ADDR을 찾는 torch.distributed.init_process_group()도 문제없이 바로 실행이 되는데, 어디서 어떤 기술이 이런 역할을 수행하는지 주요 기술을 하나씩 분석하고 해당 기술을 찾아내본다.
</div>

<small>
*2022년 6월 7일 초안 작성*  
</small>

- [내용](#내용)
  - [의문점](#의문점)
  - [주요 기술](#주요-기술)
    - [torchrun](#torchrun)
    - [Slurm](#slurm)
    - [MPI](#mpi)
    - [Enroot](#enroot)
- [정리](#정리)
- [References](#references)

# 내용

## 의문점

PyTorch 이미지를 Slurm + Pyxis로 실행하면 `RANK`, `WORLD_SIZE`, `MASTER_ADDR`등이 OS 환경변수로 자동으로 셋팅된다.

- 이 작업은 누가하는 것인지?
- Docker 이미지 내에 그런 역할을 포함해서 생성했는지?
- Pyxis나 Enroot가 그런 역할을 하는지?
- Slurm이 그런 역할을 하는지?

이 의문점을 해결하기 위해 주요 기술을 하나씩 분석해보도록 한다.

## 주요 기술

### torchrun

먼저 torchrun은 과거 `python -m torch.distributed.launch`의 기능을 모두 포함하며 [Elastic Launch](https://pytorch.org/docs/stable/elastic/run.html)라는 이름으로 새롭게 밀고 있는 기능이다. 기존 기능을 모두 포함하며 이에 더해 fail-over, 노드 갯수를 변경하는 기능 뿐만 아니라 `RANK`, `WORLD_SIZE`등의 환경변수도 셋팅해준다. 그러나 현재 `RANK`는 torchrun을 사용하지 않고도 셋팅이 되고 있으며, torchrun의 역할은 사실상 Slurm의 srun으로도 수행할 수 있다. PyTorch 게시판에서는[^fn-here] sbatch에서도 torchrun을 권장하고 있으나 결국 Pyxis로 실행하려면 맨 마지막 쓰레드처럼 torchrun 대신 srun 실행이 필요하다. torchrun을 sbatch & srun으로 적용하는 방법[^fn-ddp] 참고. Pyxis를 사용하면 환경변수 설정 과정은 hook 코드에서 하기 때문에 굳이 저런식으로 코드에서 처리해줄 필요는 없다. 임의로 srun을 `--ntasks-per-node=1`로 실행하고 torchrun에서 `--nproc_per_node=8`과 같은 식으로 실행했더니 랑데뷰가 되지 않았다.

[^fn-here]: <https://discuss.pytorch.org/t/distributed-training-on-slurm-cluster/150417/2>
[^fn-ddp]: <https://gist.github.com/TengdaHan/1dd10d335c7ca6f13810fff41e809904>

### Slurm

Slurm은 srun으로 실행할 때 각각의 노드에 `SLURM_*`으로 시작하는 다양한 환경변수를 셋팅해준다. 여기에 필요한 대부분의 값이 셋팅되지만 이 값은 Slurm에서만 알아볼 수 있으며, PyTorch distributed 모듈은 이와는 다른 환경변수를 확인한다. 결정적으로 `torch.distributed.init_process_group()` 함수는 `MASTER_ADDR`를 확인하도록 되어 있기 때문에 `SLURM_*` 환경변수를 `RANK`, `WORLD_SIZE`, `MASTER_ADDR`등으로 변환해줄 수 있는 코드 또는 장치가 필요하다.

### MPI

MPI 통신을 이용하면 `MPI_Comm_rank()`, `MPI_Comm_size()` 함수를 이용해 rank, size를 받아올 수 있다.

```
#include <mpi.h>
int main(int argc, char **argv) {
    int size, rank;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
 
    std::cout << rank << "/" << size << stdl::endl;
    ...
```

그러나 Slurm을 이용하면 OS 환경변수로 필요한 값을 셋팅해주기 때문에 사실상 MPI 통신이 필요없고, PyTorch 또한(아마 라이센스 등의 문제로) MPI가 built-in 되어 있지 않다. PyTorch에서 MPI를 사용하려면 소스 컴파일을 해야 하는데, 아마 그렇게까지 해서 사용하는 경우는 드물 것 같고 CPU 통신은 GLOO, GPU 통신은 NCCL을 기본으로 하고 있어 PyTorch에서는 굳이 MPI를 사용하지 않아도 문제가 없다.

### Enroot

Enroot에서는 Hooks[^fn-hooks]라는 기능이 있어서 이미지를 시작할 때 여러가지 환경변수나 설정을 자동으로 구성하도록 할 수 있다. 그런데 PyTorch-Ignite 코드의 주석[^fn-ignite]에서 PyTorch 이미지가 이 hook 기능을 사용한다는 힌트를 발견할 수 있었다.

[^fn-hooks]: <https://github.com/NVIDIA/enroot/blob/master/doc/standard-hooks.md>
[^fn-ignite]: <https://github.com/pytorch/ignite/blob/202df21aa568d07c1266fe8ec4140d01435f350a/ignite/distributed/comp_models/native.py#L512-L519>

```bash
# 1) Tools like enroot can have hooks to translate slurm env vars to RANK, LOCAL_RANK, WORLD_SIZE etc
# See https://github.com/NVIDIA/enroot/blob/v3.1.0/conf/hooks/extra/50-slurm-pytorch.sh
# 2) User can use torch.distributed.launch tool to schedule on N local GPUs using 1 node, 1 task by SLURM
# To cover case 1), let's ensure that defined RANK == SLURM_PROCID, LOCAL_RANK == SLURM_LOCALID,
#   WORLD_SIZE == SLURM_NTASKS. We will use defined MASTER_ADDR and MASTER_PORT instead of defining
#   them by our means
# To cover case 2), let's check that defined RANK >= SLURM_PROCID, LOCAL_RANK >= SLURM_LOCALID,
#   WORLD_SIZE >= SLURM_NTASKS, SLURM_JOB_NUM_NODES == 1
```

주석에는 Enroot가 `SLURM_*` 환경변수를 `RANK`, `WORLD_SIZE`등으로 바꿔준다고 기술되어 있다. 실제로 Enroot 코드를 보면 50-slurm-pytorch.sh[^fn-50]라는 hook 코드를 찾을 수 있다.

[^fn-50]: <https://github.com/NVIDIA/enroot/blob/master/conf/hooks/extra/50-slurm-pytorch.sh>

```bash
if ! grep -q "^PYTORCH_VERSION=" "${ENROOT_ENVIRON}"; then
    exit 0
fi
 
if [ -n "${SLURM_STEP_NODELIST-}" ] && ! grep -q "^MASTER_ADDR=" "${ENROOT_ENVIRON}" && command -v scontrol > /dev/null; then
    printf "MASTER_ADDR=%s\n" "$(scontrol show hostname "${SLURM_STEP_NODELIST}" | head -n1)" >> "${ENROOT_ENVIRON}"
fi
if [ -n "${SLURM_JOB_ID-}" ] && ! grep -q "^MASTER_PORT=" "${ENROOT_ENVIRON}"; then
    printf "MASTER_PORT=%s\n" "$((${SLURM_JOB_ID} % 16384 + 49152))" >> "${ENROOT_ENVIRON}"
fi
if [ -n "${SLURM_NTASKS-}" ] && ! grep -q "^WORLD_SIZE=" "${ENROOT_ENVIRON}"; then
    printf "WORLD_SIZE=%s\n" "${SLURM_NTASKS}" >> "${ENROOT_ENVIRON}"
fi
if [ -n "${SLURM_PROCID-}" ] && ! grep -q "^RANK=" "${ENROOT_ENVIRON}"; then
    printf "RANK=%s\n" "${SLURM_PROCID}" >> "${ENROOT_ENVIRON}"
fi
if [ -n "${SLURM_LOCALID-}" ] && ! grep -q "^LOCAL_RANK=" "${ENROOT_ENVIRON}"; then
    printf "LOCAL_RANK=%s\n" "${SLURM_LOCALID}" >> "${ENROOT_ENVIRON}"
fi
```

특정 이미지에 OS 환경변수로 `PYTORCH_VERSION`이라는 값이 셋팅되어 있으면 이를 PyTorch를 포함한 이미지로 간주하고 `scontrol show hostname`등 다양한 명령을 이용해 `SLURM_*` 환경변수를 `torch.distributed`가 사용할 수 있는 환경변수로 변환해주는 코드를 확인할 수 있다. 아울러 이 코드는 `extra/` 디렉토리에 있기 때문에 기본적으로 실행되지 않지만 NVIDIA DeepOps로 클러스터를 구성할 때 Pyxis 설치 과정에서 이 extra hooks가 기본으로 실행될 수 있도록 링크[^fn-link]를 걸어준다.

[^fn-link]: <https://github.com/NVIDIA/deepops/blob/master/roles/pyxis/tasks/main.yml>

```yaml
---
- name: install dependencies (Ubuntu)
  apt:
    name: "{{ item }}"
    state: present
  with_items: "{{ pyxis_ubuntu_deps }}"
  when: ansible_distribution == "Ubuntu"
 
- name: install dependencies (EL)
  package:
    name: "{{ item }}"
    state: present
  with_items: "{{ pyxis_el_deps }}"
  when: ansible_os_family == "RedHat"
 
- name: install slurm-pmi hook
  file:
    path: /etc/enroot/hooks.d/50-slurm-pmi.sh
    state: link
    src: /usr/share/enroot/hooks.d/50-slurm-pmi.sh
  when: is_compute
 
- name: install slurm-pytorch hook
  file:
    path: /etc/enroot/hooks.d/50-slurm-pytorch.sh
    state: link
    src: /usr/share/enroot/hooks.d/50-slurm-pytorch.sh
  when: is_compute
 
- name: pyxis tasks
  include_tasks: pyxis.yml
  when: slurm_install_pyxis
  tags: pyxis
```

따라서 DeepOps를 이용해 클러스터를 구성했다면 Enroot 실행시 PyTorch 이미지는 `SLURM_*` 환경변수를 이용해 `RANK`, `WORLD_SIZE`등을 자동으로 셋팅해주게 된다. 실제로 각 서버 `/etc/enroot/hooks.d`에는 extra hooks 링크가 걸려 있는 것을 확인할 수 있다.

<img src="https://user-images.githubusercontent.com/1250095/231106517-249b8b59-dc32-4824-b2b9-e80cba5bdb05.png" width="70%">

# 정리

Slurm + Pyxis를 이용해 PyTorch 이미지를 실행(srun)하면 `RANK`, `WORLD_SIZE`등의 OS 환경변수를 자동으로 셋팅해준다. Enroot의 Hooks 기능을 이용하는 것이며, PyTorch 관련은 default가 아니지만 NVIDIA DeepOps로 클러스터를 구성하면 default로 실행되도록 링크를 걸어준다. 이렇게 되면 사실상 torchrun과 MPI 통신의 역할을 모두 대체할 수 있기 때문에 두 기능이 더 이상 필요하지 않다. 아울러 이 내용은 공식문서에는 기입되어 있지 않고 코드에만 기술되어 있기 때문에 코드 레벨의 꼼꼼한 확인이 필요하다.

# References