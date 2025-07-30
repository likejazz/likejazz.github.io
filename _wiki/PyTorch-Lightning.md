---
layout: wiki 
title: PyTorch Lightning
tags: ["Deep Learning"]
last_modified_at: 2024/01/07 00:11:15
---

- [Lightning in 15 minutes](#lightning-in-15-minutes)
- [Trainer](#trainer)

# Lightning in 15 minutes
MNIST를 3차원 임베딩으로 나타내는 오토인코더 예제[^fn-15min]

[^fn-15min]: <https://lightning.ai/docs/pytorch/stable/starter/introduction.html>

```python
trainer = L.Trainer(max_steps=1000, devices=4)
```

- `optimizer.step()`, `loss.backward()`, `optimizer.zero_grad()` calls
- Calling of `model.eval()`, enabling/disabling grads during evaluation
- Checkpoint Saving and Loading
- Tensorboard
- Multi-GPU support 기본은 DDP로 동작한다. 주피터에서는 되지 않고 CLI에서만 가능.

# Trainer
Trainer를 실행할 때 멀티 노드인 경우 `num_nodes`를 기입해주면 다른 노드를 기다린다. 싱글 노드인데 2이상 기입하면 무한 대기상태에 빠지므로 유의.

```python
trainer = pl.Trainer(gpus=1, num_nodes=2, max_epochs=5, strategy="ddp")
```

slurm 환경에서는 Multiprocessing is handled by SLURM. 문구가 표시된다. `mpirun`으로 실행은 되지만 서로 통신을 못하고 대기한다. `RANK` 값을 얻어올 수 없기 때문으로 보이며, lightning은 OS 환경 설정을 하지 않고(enroot가 진행) 따로 MPI 통신을 하지 않는다.
