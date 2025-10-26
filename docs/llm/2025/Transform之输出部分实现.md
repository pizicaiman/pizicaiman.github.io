---
layout: doc
title: Transform之输出部分实现
date: 2025-01-10
category: llm
tags: [llm, transformer, output, softmax, vocab, logits]
excerpt: 解析Transformer输出头与词概率映射的实现（PyTorch为例）
permalink: /docs/llm/2025/transformer-output/
---

# Transformer输出部分实现（以PyTorch为例）

Transformer的输出通常是解码器(decoder)堆叠层的最终输出，通过线性变换和softmax映射为词概率分布。下面给出标准流程示例：

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class TransformerOutputLayer(nn.Module):
    def __init__(self, d_model, vocab_size):
        super().__init__()
        self.linear = nn.Linear(d_model, vocab_size, bias=False)
        # 有时tie weights到embedding: self.linear.weight = embedding.weight

    def forward(self, x):
        # x: (batch, seq_len, d_model)
        logits = self.linear(x)           # (batch, seq_len, vocab_size)
        probs = F.softmax(logits, dim=-1) # (batch, seq_len, vocab_size)
        return probs, logits
```

**说明：**

- `x` 是解码器输出(hidden states)，形状为(batch, seq_len, d_model)。
- `self.linear` 将特征维映射到词表大小，通常不加bias。
- 有时为了性能和参数共享，`linear.weight`与embedding层权重共享（即weight tying）。
- 返回的 `probs` 可以用于推理/采样等。

### 示例用法

```python
d_model = 512
vocab_size = 32000
output_head = TransformerOutputLayer(d_model, vocab_size)

dummy_x = torch.randn(8, 20, d_model)  # batch=8, seq_len=20
probs, logits = output_head(dummy_x)
print(probs.shape)   # (8, 20, 32000)
```

---

**补充：采样/解码流程常见方案**

- 贪心(`argmax`): `next_id = torch.argmax(logits, dim=-1)`
- Top-k/Top-p采样，可见huggingface或fairseq实现
- 通常只输出 logit 或 prob，由上层调用采样/解码策略

---

**参考资料：**
- [PyTorch nn.Linear 文档](https://pytorch.org/docs/stable/generated/torch.nn.Linear.html)
- [Huggingface Output heads](https://github.com/huggingface/transformers/blob/main/src/transformers/models/gpt2/modeling_gpt2.py)
- Vaswani et al., "Attention is All You Need" (2017)

