---
layout: wiki 
title: NVIDIA
tags: ["MLOps & HPC"]
last_modified_at: 2024/10/14 15:47:40
---

- [NVIDIA Data Center GPUs](#nvidia-data-center-gpus)
  - [V100, A100, H100](#v100-a100-h100)
  - [Streaming Multiprocessor (SM), Compute Unit (AMD)](#streaming-multiprocessor-sm-compute-unit-amd)
  - [SuperPOD](#superpod)

# NVIDIA Data Center GPUs

<img src="/images/2024/293452912-bf1ac3b7-1c4c-4ee5-8036-44c3e73f13c7.png" width="100%">

| Name | bits/s | Bytes/s |
| ------ | ----- | ------ |
| SanDisk Extreme SSD | | 550MB/s read |
| USB 3.1 Gen 2 | 10Gb/s | 1.25GB/s |
| HDMI | 10Gb/s | 1.25GB/s |
| HDMI 4K | 18Gb/s | 2.25GB/s |
| M1 Macbook Pro 1TB SSD | | 7.4GB/s read |
| M2 Air 1TB SSD | | 2.8GB/s read |
| M2 Air Memory | | 100GB/s |
| M1 Pro Memory | | 200GB/s |
| M1 Max Memory | | 400GB/s |
| NVLink(A100) | | 600GB/s |
| HBM(A100 40G) | | 1.5TB/s |
| HBM(A100 80G) | | 1.9TB/s |
| HBM(H100 SXM 80G) | | 3.35TB/s |

## V100, A100, H100
<img src="/images/2024/Screenshot 2024-04-22 at 2.04.22 PM.png" width="70%"> 
<img src="/images/2024/Screenshot 2024-04-22 at 2.08.45 PM.png" width="70%">
<img src="/images/2024/Screenshot 2024-04-22 at 2.12.01 PM.png" width="70%">

## Streaming Multiprocessor (SM), Compute Unit (AMD)

<img src="/images/2024/Screenshot 2024-04-22 at 2.04.42 PM.png" width="45%">

<img src="/images/2024/Screenshot 2024-04-22 at 2.09.17 PM.png" width="45%" style="float: left; margin-right: 5px">
<img src="/images/2024/Screenshot 2024-04-22 at 2.12.26 PM.png" width="45%">

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==" style="clear: both">

## SuperPOD
Compute Nodes: 40ea DGX A100 system(8x A100)

<img width="80%" src="/images/2024/163540882-1069b3b7-4aa1-4c81-b572-65cbd7d4e033.png">

A100 80GB부터는 VRAM이 80GB. V100까지는 16/32GB까지만 제공되어 insufficient memory error가 잦았다.

```console
$ nvidia-smi topo --matrix
	GPU0	GPU1	GPU2	GPU3	GPU4	GPU5	GPU6	GPU7	CPU Affinity	NUMA Affinity
GPU0	 X 	NV1	NV1	NV2	NV2	PHB	PHB	PHB	0-63	0-1
GPU1	NV1	 X 	NV2	NV1	PHB	NV2	PHB	PHB	0-63	0-1
GPU2	NV1	NV2	 X 	NV2	PHB	PHB	NV1	PHB	0-63	0-1
GPU3	NV2	NV1	NV2	 X 	PHB	PHB	PHB	NV1	0-63	0-1
GPU4	NV2	PHB	PHB	PHB	 X 	NV1	NV1	NV2	0-63	0-1
GPU5	PHB	NV2	PHB	PHB	NV1	 X 	NV2	NV1	0-63	0-1
GPU6	PHB	PHB	NV1	PHB	NV1	NV2	 X 	NV2	0-63	0-1
GPU7	PHB	PHB	PHB	NV1	NV2	NV1	NV2	 X 	0-63	0-1

Legend:

  X    = Self
  SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
  NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing at most a single PCIe bridge
  NV#  = Connection traversing a bonded set of # NVLinks
```