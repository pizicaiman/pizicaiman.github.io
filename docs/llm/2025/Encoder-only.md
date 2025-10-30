---
layout: doc
title: Encoder-only 架构详解
date: 2025-01-09
category: llm
tags: [llm, transformer, encoder, encoder-only, bert]
excerpt: 详细介绍Transformer编码器-only（仅编码器）架构原理、应用场景及代码实现，及其在BERT等模型中的关键作用。
permalink: /docs/llm/2025/encoder-only/
---

# Encoder-only 架构（仅编码器）

Encoder-only 架构指仅包含编码器（Encoder）部分的神经网络结构，如 BERT、ERNIE、RoBERTa、ALBERT 等主流模型。这类模型专注于对输入文本的全面理解，广泛应用于分类、检索、嵌入、Masked Language Model（MLM）等任务场景。

---

## 1. 架构基础

- **纯编码器堆叠**  
  整个模型由多层 Transformer Encoder 依次堆叠组成，无解码器结构。

- **自注意力机制**  
  每一层编码器通过多头自注意力（Multi-Head Self-Attention）实现 token 间全局信息交互，能够建模上下文依赖。

- **输入全可见**  
  输入序列在处理时整体可见（区别于 decoder-only 的因果掩码），有利于捕捉文本的双向特征。

- **无需自回归**  
  只做信息理解，无需一步步生成 token，适合各种判别、表征和检索类任务。

---

## 2. 核心流程

1. **嵌入层（Embedding）**  
   - 输入文本经分词（Tokenization），映射成词向量，并加上位置（Positional）、段落（Segment）嵌入。

2. **多层编码器堆叠**  
   - 序列逐层通过 N 层 Transformer Encoder Block，每层包括：
     - 多头自注意力（Self-Attention）
     - 残差与 LayerNorm
     - 前馈全连接网络（Feed-Forward/MLP）
     - 再次残差与归一化

3. **输出表征**  
   - 通常以 `[CLS]` token 作为全局语义表征，或各 token 的上下文向量用于下游任务。

---

## 3. 应用场景

- 文本分类 / 文档分类
- 序列标注（如命名实体识别 NER）
- 句对任务（如文本蕴含）
- 检索与语义匹配
- 生成下游任务的数据嵌入

---

## 4. 简化伪代码

```python
# 伪代码：Encoder-only 主体结构
input_tokens = ...  # 输入token序列
x = Embedding(input_tokens)

for i in range(num_layers):
    x = EncoderBlock(x)     # 全局自注意力

output = x                  # 每个token的上下文表示
```

---

## 5. 总结

Encoder-only 架构凭借其强大的表征能力和高效的训练特性，成为现代 NLP 领域各类判别与理解类任务的基础结构之一。它通过堆叠编码器层，实现输入文本的深度理解，在大规模语料上预训练后，适用于多种下游任务迁移与微调。

An encoder-only architecture is a neural network model that consists exclusively of encoder blocks, without any decoder components. This design is widely used for tasks that require comprehensive understanding of input sequences, such as text classification, retrieval, embedding generation, and masked language modeling.

### Key Characteristics

- **Stacked Encoders:** The model is composed of multiple identical layers of encoder blocks (e.g., as seen in BERT).
- **Self-Attention:** Each block uses self-attention to model relationships between all tokens in the input.
- **No Auto-Regressive Decoding:** Unlike encoder-decoder or decoder-only models, encoder-only models do not generate sequences one token at a time.
- **Input Processing:** The entire input sequence is processed simultaneously, enabling full context awareness.

### Example: BERT Architecture

BERT (Bidirectional Encoder Representations from Transformers) is the most famous encoder-only model. It takes a sequence of tokens, adds positional and segment embeddings, then passes them through a stack of self-attention encoder blocks to obtain contextualized token representations.

```text
[Input Sequence] → [Tokenization & Embeddings] → [Encoder Stack] → [Output Representations]
```

### Typical Use Cases

- **Sentence/Document Classification**
- **Sentence Pair Tasks (e.g., entailment)**
- **Token Classification (e.g., NER)**
- **Retrieval and Semantic Search**
- **Embedding Generation for Downstream Tasks**

### Encoder Block (Transformer)

Each encoder block generally consists of:
1. Multi-head Self-Attention
2. Add & Norm
3. Feed Forward Network
4. Add & Norm

A high-level pseudocode for an encoder-only model:

```python
class EncoderOnlyModel(nn.Module):
    def __init__(self, ...):
        super().__init__()
        self.embedding = EmbeddingLayer(...)
        self.encoder = nn.ModuleList([EncoderBlock(...) for _ in range(num_layers)])
    
    def forward(self, x):
        x = self.embedding(x)
        for encoder_block in self.encoder:
            x = encoder_block(x)
        return x  # contextualized representations for each token
```

### Summary

Encoder-only models are powerful for tasks where deep understanding and contextualization of the full input is necessary, but not for tasks requiring generative output (e.g., text generation).
