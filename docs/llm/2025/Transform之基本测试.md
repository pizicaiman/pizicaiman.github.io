---
layout: doc
title: Transformer之基本单元测试（编码器、解码器层的PyTorch验证）
date: 2025-01-11
category: llm
tags: [llm, transformer, test, encoder, decoder, unit-test]
excerpt: PyTorch实现的Transformer编码器/解码器层的最小测试代码，验证forward流程和对齐。
permalink: /docs/llm/2025/transformer-basic-test/
---

本节以最小可运行的代码，测试`EncoderLayer`和`TransformerDecoderLayer`的对齐输入输出和整体forward流程，帮助大家自行验证模型模块拼装正确性。

```python
import torch
import torch.nn as nn

# 假定已实现的核心组件：MultiHeadAttention, PositionwiseFeedForward
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, nhead):
        super().__init__()
        self.attn = nn.MultiheadAttention(d_model, nhead, batch_first=True)
    def forward(self, q, k, v, mask=None):
        # q, k, v: (B, L, D)
        attn_output, attn_weights = self.attn(q, k, v, attn_mask=mask)
        return attn_output, attn_weights

class PositionwiseFeedForward(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super().__init__()
        self.ff = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(d_ff, d_model),
        )
    def forward(self, x):
        return self.ff(x)

# 引自正文的编码器层和解码器层（见相关章节）
class EncoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        self.self_attn = MultiHeadAttention(d_model, num_heads)
        self.ffn = PositionwiseFeedForward(d_model, d_ff, dropout)
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout1 = nn.Dropout(dropout)
        self.dropout2 = nn.Dropout(dropout)
        
    def forward(self, x, mask=None):
        attn_output, _ = self.self_attn(x, x, x, mask)
        x = x + self.dropout1(attn_output)
        x = self.norm1(x)
        ffn_output = self.ffn(x)
        x = x + self.dropout2(ffn_output)
        x = self.norm2(x)
        return x

class TransformerDecoderLayer(nn.Module):
    def __init__(self, d_model, nhead, d_ff, dropout=0.1):
        super().__init__()
        self.self_attn = MultiHeadAttention(d_model, nhead)
        self.cross_attn = MultiHeadAttention(d_model, nhead)
        self.ffn = PositionwiseFeedForward(d_model, d_ff, dropout=dropout)
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.norm3 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, memory, self_mask=None, cross_mask=None):
        x2, _ = self.self_attn(self.norm1(x), self.norm1(x), self.norm1(x), mask=self_mask)
        x = x + self.dropout(x2)
        x2, _ = self.cross_attn(self.norm2(x), memory, memory, mask=cross_mask)
        x = x + self.dropout(x2)
        x2 = self.ffn(self.norm3(x))
        x = x + self.dropout(x2)
        return x

# =============================
# 测试用例（Encoder/Decoder层）
# =============================
def test_transformer_layers():
    batch = 2
    src_len = 8
    tgt_len = 5
    d_model = 32
    nhead = 4
    d_ff = 64

    x = torch.randn(batch, src_len, d_model)              # encoder输入
    tgt = torch.randn(batch, tgt_len, d_model)            # decoder输入
    encoder_layer = EncoderLayer(d_model, nhead, d_ff)
    decoder_layer = TransformerDecoderLayer(d_model, nhead, d_ff)

    # 编码器层正向
    memory = encoder_layer(x)
    print("EncoderLayer输出 shape:", memory.shape)         # (batch, src_len, d_model)

    # 解码器层正向
    out = decoder_layer(tgt, memory)
    print("DecoderLayer输出 shape:", out.shape)            # (batch, tgt_len, d_model)

if __name__ == "__main__":
    test_transformer_layers()
```

**输出示例：**
```
EncoderLayer输出 shape: torch.Size([2, 8, 32])
DecoderLayer输出 shape: torch.Size([2, 5, 32])
```

> 补充建议：可以根据上述结构自由增加mask，或堆叠多层封装整体Transformer结构，形成端到端的验证环境。
