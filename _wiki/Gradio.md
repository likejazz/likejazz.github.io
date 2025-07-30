---
layout: wiki 
title: Gradio
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/12 12:53:09
---

<!-- TOC -->

- [예제](#예제)

<!-- /TOC -->

# 예제
```python
def inference(inputs):
    return 'a', 'b', 'c'
    
demo = gr.Interface(
    fn=inference,
    inputs=[
        gr.Textbox(lines=3),
    ],
    outputs=[
        gr.Textbox(label='model1'),
        gr.Textbox(label='model2'),
        gr.Textbox(label='model3'),
    ],
    examples=[
        'example1',
        'example2',
        'example3',
    ]
)

demo.launch(server_name='0.0.0.0', server_port=8888)
```