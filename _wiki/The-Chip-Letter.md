---
layout: wiki 
title: The Chip Letter
tags: ["MLOps & HPC"]
last_modified_at: 2024/07/22 17:35:20
---

[AI Accelerators: The Cambrian Explosion
](https://thechipletter.substack.com/p/ai-accelerators-the-cambrian-explosion)
- 구글 TPU v1 2015년 출시(2016년 발표) 추론만 가능
  - TPU v2 2017년 출시 HBM, bfloat16
  - TPU v4 2021년 출시. 수냉 training 버전. 수냉 x inference only TPU v4i
  - TPU v5 2023년 출시
  - CISC → SiFive RISC-V
- Qualcomm Cloud AI 100 Ultra는 SRAM 576MB(A100은 40MB, RTX 4080S 64MB L2), DRAM은 128GB LPR4x, 100B models on a single card 홍보.

[Google's First Tensor Processing Unit: Origins](https://thechipletter.substack.com/p/googles-first-tensor-processing-unit)
- 2011년 구글 X, 2013년 힌튼 회사 인수, 2014년 딥마인드 인수
- 2013년 Alex는 기존 모델이 모두 CPU에 실행되고 있어 개인적으로 GPU 구매. 이후 $1억 3천만에 40,000개 NVIDIA GPU 구매.
- 아키텍처 → 실리콘 전환에 LSI Corporation(현재 브로드컴)에 의뢰

[The Arm Story Part 1 : From Acorns](https://thechipletter.substack.com/p/the-arm-story-part-1-from-acorns-6e2), [2](https://thechipletter.substack.com/p/the-arm-story-part-2-archimedes-to-4fd), [3](https://thechipletter.substack.com/p/the-arm-story-part-3-creating-a-global-1b0)
> There will be two types of computer company in the future, those with silicon design capability and those that are dead. - Hermann Hauser

Why did this architecture from a small, ultimately failed, British company come to be so important and to survive and prosper against much larger competition?

- 12명의 엔지니어로 시작. Acorn과 Apple이 고객.
- Arm 지분 매각이 애플을 구하는데 큰 역할. 1998년 상장, 애플은 2000년에 모두 매각.

[Motorola's Pioneering 8-bit 6800: Origins and Architecture](https://thechipletter.substack.com/p/motorolas-pioneering-8-bit-6800-origins)
  - [Leaving Arizona](https://thechipletter.substack.com/p/leaving-arizona)  

MOS Technology 는 Son of Motorola로 평가. 초기에 Atari에 납품. 이후 Commodore 사업부.
- MOS 6502 애플 I에 적용. 6800은 $175, 6502는 $25
- Intel → Zilog, Motorola → MOS

[A History of C Compilers - Part 1: Performance, Portability and Freedom](https://thechipletter.substack.com/p/a-history-of-c-compilers-part-1-performance)
- Arm은 x86에 도전, RISC-V는 Arm에 도전, GPU와 TPU 같은 AI-focused ASIC은 데이터센터의 핵심 하드웨어로 자리매김.
- 메인프레임은 컴파일러와 함께 제공
  - DEC PDP-11 이후 새로운 OS, 컴파일러를 개발할 기회. 유닉스의 탄생. 1972년 데니스 리치는 C언어와 컴파일러를 개발한 다음 유닉스를 어셈블리에서 C로 재작성.
  - [Byte Magazine 1983-08, The C Language](https://archive.org/details/byte-magazine-1983-08/mode/2up)

[Moore on Moore](https://thechipletter.substack.com/p/moore-on-moore)

[The RISC Wars Part 1 : The Cambrian Explosion
](https://thechipletter.substack.com/p/the-risc-wars-part-1-the-cambrian-c55)