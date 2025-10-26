---
layout: doc
title: Transformer之注意力机制
date: 2025-01-10
category: llm
tags: [llm, transformer, attention, self-attention, multi-head]
excerpt: 解析Transformer模型中注意力机制（含Self-Attention与Multi-Head Attention）的原理、公式与实现。
permalink: /docs/llm/2025/transformer-attn/
---

本文系统解析了Transformer结构的核心——注意力机制，包括自注意力（Self-Attention）、缩放点积注意力（Scaled Dot-Product Attention）、多头注意力（Multi-Head Attention）等原理、公式与典型流程。帮助读者理解模型如何高效建模序列中各位置关系，是后续理解Encoder、Decoder与输入实现的基础。

Transformer模型的核心创新之一就是**注意力机制**（Attention Mechanism）。在Transformer中，Attention机制赋予了模型在处理序列数据时灵活地聚焦于输入序列中的不同部分，使得每一个输出都能够参考整个输入序列的相关信息，而不是像RNN那样只关注局部上下文。

---

## 1. 什么是注意力机制？

注意力机制最初受到人类视觉关注机制的启发——人类在感知世界时会对一些重要信息给予更多注意。机器学习中的注意力机制模拟了这种“选择性关注”能力，它使模型在做决策时对输入的不同部分赋予不同的权重。

---

## 2. Self-Attention（自注意力）

Transformer中最核心的注意力机制是**自注意力（Self-Attention）**。

给定输入序列 $X = (x_1, x_2, ..., x_n)$，自注意力机制在计算每个位置 $i$ 的表示 $z_i$ 时，会考虑序列中所有位置 $j$ 的输入，并分配一个权重（注意力分数）：

$$
z_i = \sum_{j=1}^{n} \text{Attention}(x_i, x_j) \cdot x_j
$$

---

## 3. Scaled Dot-Product Attention（缩放点积注意力）

在实践中，Transformer采用了缩放点积注意力，其计算方式如下：

1. 将输入 $X$ 通过三组可学习参数线性变换得到 Queries (Q), Keys (K), Values (V)：
   $$
   Q = X W^Q \\
   K = X W^K \\
   V = X W^V
   $$
2. 计算每对Query-Key之间的点积，除以缩放因子 $\sqrt{d_k}$（$d_k$ 是K的维度），再通过softmax归一化，得到权重：

   $$
   \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{Q K^\top}{\sqrt{d_k}}\right) V
   $$

---

## 4. Multi-Head Attention（多头注意力）

多头注意力机制将Q、K、V分别分成 $h$ 组（即$h$个“头”），每组经过独立的线性变换，进行不同的自注意力计算，最后将所有头的输出拼接再映射出去：

- 优点：多头机制让模型可以关注不同子空间的信息，提升模型表征能力。

---

## 5. 图解Transformer Attention机制

```mermaid
graph LR
    A[输入序列] --> B[线性变换生成Q/K/V]
    B --> C[缩放点积注意力]
    C --> D[多头拼接]
    D --> E[输出]
```

---

## 6. 注意力机制的意义

- **全局依赖建模**：自注意力可以打破序列的“距离”限制，使每个位置直接看见整个序列的信息。
- **并行运算**：不同于RNN串行处理，Transformer每个位置的注意力可以并行计算，大大提高效率。

---

## 7. 参考代码（PyTorch实现核心 Attention）

```python
import torch
import torch.nn.functional as F

def scaled_dot_product_attention(Q, K, V, mask=None):
    d_k = Q.size(-1)
    scores = Q @ K.transpose(-2, -1) / (d_k ** 0.5)
    if mask is not None:
        scores = scores.masked_fill(mask == 0, float('-inf'))
    attn = F.softmax(scores, dim=-1)
    output = attn @ V
    return output, attn
```

---

## 参考资料

- Vaswani et al. "Attention is All You Need" (2017)
- [The Illustrated Transformer (Jay Alammar)](http://jalammar.github.io/illustrated-transformer/)


