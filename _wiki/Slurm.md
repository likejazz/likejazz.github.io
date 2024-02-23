---
layout: wiki 
title: Slurm
tags: ["MLOps & HPC"]
last_modified_at: 2024/02/23 13:29:58
---

- [DeepOps](#deepops)
  - [Slurm](#slurm)
    - [DRAIN 노드 복구](#drain-노드-복구)
    - [계속 DRAIN 상태로 빠지는 경우](#계속-drain-상태로-빠지는-경우)
    - [운영 명령](#운영-명령)
    - [sbatch](#sbatch)
  - [Pyxis](#pyxis)
    - [Pyxis로 각 노드 테스트](#pyxis로-각-노드-테스트)
    - [Pyxis에서 환경변수](#pyxis에서-환경변수)
    - [sshd in Pyxis](#sshd-in-pyxis)
- [실험 환경](#실험-환경)
  - [CPU](#cpu)
  - [GPU](#gpu)
  - [MPI](#mpi)
  - [NCCL](#nccl)

# DeepOps
NVIDIA가 공개한 자사 DGX 서버에 Slurm clusters를 구성하는 툴의 이름이다. aws에 Ubuntu 환경(공식 Deep Learning AMI 18.04)에서도 설치를 지원한다. 그러나 ansible로 설치하는 스크립트의 특성상 많은 시행착오가 뒤따르며, 한 번에 제대로 설치되진 않는다. 처음에는 몇 대가 failed로 나왔다가 한 번 더 해보면 정상으로 진행되기도 한다.

## Slurm
srun 실행 후 Pyxis 플러그인으로 컨테이너 이미지(sqsh)를 얹어 실행해보면 각 노드에 네트워크로 전송하는 것을 확인할 수 있다. nload 에서 볼 수 있다. `nvcr.io/nvidia/pytorch:22.03-py3`는 10GB 이므로 전송에 aws g4dn에서 1m 40s 이상 소요된다. 디스크를 좀 더 좋게 설정해(AWS gp3 → io2) 40s까지 줄였다. 각 노드 로컬에서 직접 `$ enroot create`를 할 경우 30s까지 줄어든다.

```console
$ srun -N2 --container-image=alpine grep PRETTY /etc/os-release # import 후 OS 출력
$ srun -N2 --container-image=./alpine.sqsh grep PRETTY /etc/os-release # sqsh 전달하여 OS 출력
```

NCCL Tests[^fn-nccl]를 위한 dockerfile은 다음과 같다.

[^fn-nccl]: <https://github.com/NVIDIA/nccl-tests>
```docker
FROM nvcr.io/nvidia/pytorch:22.03-py3

RUN apt-get update
RUN apt install net-tools

ENV NCCL_SOCKET_IFNAME=en
ENV NCCL_TESTS_COMMITISH=f773748b46

WORKDIR /opt/nccl_tests
RUN wget -q -O - https://github.com/NVIDIA/nccl-tests/archive/${NCCL_TESTS_COMMITISH}.tar.gz | \
        tar --strip-components=1 -xzf - \
        && CC=mpicc CXX=mpicxx make MPI=1
RUN cp -R /opt/nccl_tests/build/* /usr/local/bin/
```

### DRAIN 노드 복구
노드가 `drain` 상태일때 다음과 같이 복구한다.
```console
$ sudo scontrol update nodename=bcm-dgxa100-[0006-0007] state=resume

# 각 서버 별 상태 나열
$ sinfo -Nl
Mon May 23 11:43:57 2022
NODELIST   NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON
gpu01          1    batch*        idle 64     2:16:2 467190        0      1   (null) none
gpu02          1    batch*        idle 64     2:16:2 467190        0      1   (null) none
gpu03          1    batch*        idle 64     2:16:2 467190        0      1   (null) none
gpu04          1    batch*        idle 64     2:16:2 467190        0      1   (null) none
gpu10          1    batch*        idle 64     2:16:2 467190        0      1   (null) none
gpu11          1    batch*        idle 64     2:16:2 467190        0      1   (null) none
gpu12          1    batch*        idle 64     2:16:2 467190        0      1   (null) none
gpu13          1    batch*        idle 64     2:16:2 467190        0      1   (null) none

# Tasks 조회
$ watch -n 1 squeue -l
```

전체 노드 스펙은 `/etc/slurm/slurm.conf`에 있고, 각 노드는 `/etc/nhc/nhc.conf`에 정의되어 있다.

deepops에 slurm-validation.yml이 포함되어 있으므로 다음과 같이 nccl_tests를 수행할 수 있다.
```console
$ ansible-playbook -l slurm-cluster playbooks/slurm-cluster/slurm-validation.yml \
-e '{srun_exports: "OMPI_MCA_btl_tcp_if_include=ens5,OMPI_MCA_pml=^ucx,NCCL_SOCKET_IFNAME=ens"}' \
-vvv
```

Interactive 모드는 다음과 같이 실행한다.
```
$ srun --gres=gpu:8 \
 --partition=c-nlp \
 --nodelist=bcm-dgxa100-0019 \
 --container-image /bcmgpfs/hyperai/hyperai+vicuna+latest.sqsh \
 --container-mounts /bcmgpfs:/bcmgpfs \
 --unbuffered \
 --pty \
 --job-name=hyperai-interactive \
 bash -i
```

### 계속 DRAIN 상태로 빠지는 경우
gpu03 노드에만 별도 패키지를 설치했더니 그 서버가 명령을 실행하면 자꾸만 drain 상태로 빠진다.
```
$ sudo tail -f /var/log/slurm/slurmctld.log
[2022-06-10T05:06:29.819] sched: _slurm_rpc_allocate_resources JobId=591 NodeList=gpu03 usec=470
[2022-06-10T05:06:29.909] error: Prolog launch failure, JobId=591
```
Prolog launch failure 에러가 발생하며, `/etc/slurm/slurm.conf`에서 `Prolog=/etc/slurm/prolog.sh`를 찾았다. 해당 스크립트는 `/etc/slurm/prolog.d` 디렉토리를 실행한다.
```
-rwxr-xr-x 1 slurm slurm  734 May 24 06:29 50-all-enroot-dirs*
-rwxr-xr-x 1 slurm slurm  169 May 24 06:29 50-exclusive-cpu*
-rwxr-xr-x 1 slurm slurm  156 May 24 06:29 50-exclusive-gpu*
-rwxr-xr-x 1 slurm slurm  196 May 24 06:29 50-exclusive-ssh*
-rwxr-xr-x 1 slurm slurm   93 Jun 10 05:11 95-all-rootless*
```
Slurm의 리소스 제약을 보조하는 스크립트와 함께 `95-all-rootless`는 singularity를 실행하는 명령인데, 원래 설치되어 있지 않기 때문에 실행되지 않아야 하나 얼마전 gpu03 노드만 별도로 singularity를 설치한 적이 있다. 이 노드에만 명령이 실행되면서 이후 명령에 오류가 발생. gpu03 노드도 실행되지 않도록 조치하여 복구.

NVIDIA BCM 설치에서는 slurm GPU 노드의 로그는 `/var/log/slurm/slurmd.log`에 위치

### 운영 명령
```
$ sacct                   # Job History
$ sacctmgr list user      # 사용자 조회
$ sacctmgr show account   # 그룹(account) 조회

# 특정 파티션 전체 노드 테스트
$ srun --partition=a-all --gres=gpu:8 --nodes=40 nvidia-smi -L
$ sacctmgr list event     # Drained 이벤트 조회

# 특정 노드 테스트
$ srun --gres=gpu:8 \
-u \  # unbuffered, 버퍼링하지 않도록
-l \  # label, remote task id 출력
--partition=c-failover \
--nodelist=bcm-dgxa100-0006 \
--container-image=./nvidia+pytorch+22.05-py3.sqsh \
nvidia-smi
```

다음은 `/usr/local/bin/sll`을 만들고 `sll`로 실행하여 Job 상태를 실시간으로 확인하는 스크립트다.
```
#!/bin/bash

SLURM_TIME_FORMAT="%d %H:%M:%S" watch -n 1 "sacct --allusers -X --format=JobID,Partition,User,JobName%40,Start%12,Elapsed,NNodes,State | sort -r"
```

### sbatch
sbatch를 이용해 torchrun 대신 srun으로 작업을 실행하는 예제:
```bash
#!/bin/bash

# Parameters
#SBATCH --dependency=singleton
#SBATCH --exclusive
#SBATCH --output=/bcmgpfs/hyperai/FastChat/output/hyperai-vicuna.out
#SBATCH --error=/bcmgpfs/hyperai/FastChat/output/hyperai-vicuna.err
#SBATCH --gpus-per-node=8
#SBATCH --ntasks-per-node=8
#SBATCH --nodes=2
#SBATCH --nodelist=bcm-dgxa100-[0020,0021]
#SBATCH --partition=c-nlp
#SBATCH --mem=0
#SBATCH --time=30-00:00:00
#SBATCH --job-name=hyperai-vicuna

export HTTP_PROXY=http://XXX:3128
export HTTPS_PROXY=http://XXX:3128

srun \
--output /bcmgpfs/hyperai/FastChat/output/hyperai-vicuna.out \
--error /bcmgpfs/hyperai/FastChat/output/hyperai-vicuna.err \
--container-image /bcmgpfs/hyperai/hyperai+vicuna+latest.sqsh \
--container-mounts /bcmgpfs:/bcmgpfs \
--no-container-mount-home bash -c '
  cd /bcmgpfs/hyperai/FastChat;
  python \
    fastchat/train/train_mem.py \
    --model_name_or_path /bcmgpfs/hyperai/llama-7b \
    --data_path /bcmgpfs/hyperai/sg_90k_merged.json \
    --bf16 True \
    --output_dir output \
    --num_train_epochs 3 \
    --per_device_train_batch_size 2 \
    --per_device_eval_batch_size 2 \
    --gradient_accumulation_steps 16 \
    --evaluation_strategy "no" \
    --save_strategy "steps" \
    --save_steps 1200 \
    --save_total_limit 10 \
    --learning_rate 2e-5 \
    --weight_decay 0. \
    --warmup_ratio 0.03 \
    --lr_scheduler_type "cosine" \
    --logging_steps 1 \
    --fsdp "full_shard auto_wrap" \
    --fsdp_transformer_layer_cls_to_wrap "LlamaDecoderLayer" \
    --tf32 True \
    --model_max_length 2048 \
    --gradient_checkpointing True \
    --lazy_preprocess True'
```

Pyxis의 hook이 사실상 torchrun의 역할을 대행해준다.

제출:
```console
$ sbatch runme.sh
```

## Pyxis
이미지 배포에 오랜 시간이 걸리는데, 각 노드에 `$ enroot list`로 이미지가 존재하면 `container-name` 지정으로 호출할 수 있다. 이 경우 이미지 전송을 하지 않으므로 매우 빠르게 실행 가능하다.

```
$ srun --gres=gpu:1 -N2 -l --container-name=pytorch-slurm nvidia-smi
```

Pyxis 플러그인은 `pyxis_pytorch-slurm`를 찾는다. 따라서 `enroot`로 미리 생성할때는 접두어 `pyxis_`를 붙여야 한다. 그러나 slurm을 실행할 때 Pyxis가 각 노드의 enroot 이미지를 날려버리는 이슈가 있다.

### Pyxis로 각 노드 테스트
enroot는 기본적으로 home이 mount되고, Pyxis는 기본적으로 home이 mount 되어 있지 않다. home이 mount 되면 conda 환경 등 많은 부분이 영향을 받기 때문에 주의. 다음은 특정 dir만 맵핑되어 테스트 가능하다.

```console
$ srun --gres=gpu:8 \
-u \
-l \
--partition=c-failover \
--nodelist=bcm-dgxa100-0006 \
--container-mounts=/autohome/hyperai/skpark:/workspace/skpark \
--container-image=./nvidia+pytorch+22.05-py3.sqsh \
ls -al ./skpark
```

### Pyxis에서 환경변수
Pyxis는 `WORLD_SIZE`, `RANK`, `LOCAL_RANK`를 `SLURM_*`에서 읽어서 OS 환경 변수로 설정해주는 후처리 훅이 있다.

만약 GPU 4장, 2 노드가 있다면,
- `WORLD_SIZE`: `4 * 2 = 8`
- `RANK`: `[0, 1, 2, 3, 4, 5, 6, 7]`
- `LOCAL_RANK`: 각 노드에서 `[0, 1, 2, 3]`

### sshd in Pyxis
ssh 접속시 connection refused 오류[^fn-ssh]가 발생한다. 권한 문제로 openssh가 연결을 종료하는 문제. Pyxis에서 `--no-container-remap-root`로 일반 사용자로 srun 실행하여 해결.

[^fn-ssh]: <https://github.com/NVIDIA/pyxis/issues/85>

# 실험 환경

|    | CPU 실행 | GPU 실행 | MPI | NCCL |
| -- | ------- | ------- | --- | ---- |
| mpirun | o | o | o | o |
| srun | o | o | x | o |
| srun + Pyxis | o | o | o | o |
| srun + Pyxis(PyTorch) | | | o | o |

## CPU
```console
# mpirun
$ mpirun --host gpu01,gpu02 cat /etc/os-release | grep PRETTY

# srun
$ srun -N2 -l cat /etc/os-release | grep PRETTY

# srun + Pyxis
$ srun -N2 -l --container-image=./openmpi.sqsh cat /etc/os-release | grep PRETTY
```

## GPU
```console
# mpirun
$ mpirun --host gpu01,gpu02 nvidia-smi

# srun
$ srun --gres=gpu:1 -N2 -u -l nvidia-smi -L

# srun + Pyxis
$ srun --gres=gpu:1 -N2 -l --container-image=./nccl.sqsh nvidia-smi
```

## MPI
mpirun  
conda 환경에서는 다른 `mpirun`이 실행될 수 있으므로 유의
```console
$ mpirun --host gpu01,gpu02 \
--mca btl_base_warn_component_unused 0 \
--mca btl_tcp_if_exclude docker0 \
./mpic
```
srun(x)  
`orte_ess_init failed`가 발생한다. slurm은 slurmstepd가 실행하는데, 여기서는 MPI 통신을 위해 굳이 orted를 실행하려다 발생하는 오류로 보인다.
```console
$ srun --gres=gpu:1 -N2 \
--export="OMPI_MCA_btl_tcp_if_include=ens5" \
./mpic
```
srun + Pyxis
```console
$ srun --gres=gpu:1 -N2 \
--export="OMPI_MCA_btl_tcp_if_include=ens5" \
--container-image=./openmpi.sqsh \
./mpic
```
srun + Pyxis(PyTorch)  
15GB, 40s 소요
```console
$ srun --gres=gpu:1 -N2 \
--export="OMPI_MCA_btl_tcp_if_include=ens5,OMPI_MCA_pml=^ucx" \
--container-image=./pytorch-nccl-tests.sqsh \
./openmpi/mpic
```

## NCCL
mpirun  
conda 환경에서는 다른 `mpirun`이 실행될 수 있으므로 유의
```console
$ mpirun -x NCCL_SOCKET_IFNAME=ens \
-x LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH \
--host gpu01,gpu02 \
--mca btl_base_warn_component_unused 0 \
--mca btl_tcp_if_include ens5 \
./all_reduce_perf -b 100M -e 110M -c 0 -n 1
```
srun
```console
$ srun --gres=gpu:1 \
-u \
-l \
--nodelist=gpu01,gpu02 \
--export=ALL,"NCCL_SOCKET_IFNAME=ens" \
python nccl.py
```
srun + Pyxis
```console
$ srun --gres=gpu:1 -N2 \
--export=ALL,"OMPI_MCA_btl_tcp_if_include=ens5,NCCL_SOCKET_IFNAME=ens" \
--container-image=./nccl.sqsh \
all_reduce_perf -b 100M -e 110M -c 0 -n 1
```
srun + Pyxis(PyTorch)  
`mpirun`에 비해 1/2 속도밖에 나오지 않는다.
```console
$ srun --gres=gpu:1 -N2 \
--export=ALL,"OMPI_MCA_btl_tcp_if_include=ens5,OMPI_MCA_pml=^ucx,NCCL_SOCKET_IFNAME=ens" \
--container-image=./pytorch-nccl-tests.sqsh \
all_reduce_perf -b 100M -e 110M -c 0 -n 1
```
