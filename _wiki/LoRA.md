---
layout: wiki 
title: LoRA
tags: ["LLM Training"]
last_modified_at: 2024/10/24 00:49:43
---

- [Train](#train)
- [Merge](#merge)

# Train
deepspeed로 `get_peft_model()`, `PeftModel.from_pretrained()`와의 차이는 trainable 여부다.

# Merge
```python
base_model = AutoModelForCausalLM.from_pretrained(base_model_path).to(0)
peft_model = PeftModel.from_pretrained(base_model, lora_path)

merged_model = peft_model.merge_and_unload()
merged_model.save_pretrained(merged_model_path)
```