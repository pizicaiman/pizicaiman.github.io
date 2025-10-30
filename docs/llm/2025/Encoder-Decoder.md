

Encoder-Decoder 架构是大规模预训练语言模型（如 T5、BART、mT5、MarianMT 等）中的经典结构类型，广泛应用于机器翻译、问答、摘要等需“输入-输出”成对建模的场景。

---

### 1. 架构基础

- **Encoder（编码器）**  
  对输入序列（如原文句子）进行理解和高层表示。由多层 Transformer Encoder 堆叠组成，每层包含自注意力（Self-Attention）、前馈网络及归一化等模块。输入序列通常全部可见。

- **Decoder（解码器）**  
  生成目标序列（如翻译后文本），由多层 Transformer Decoder 组成。每层包含掩码自注意力（Masked Self-Attention）、交叉注意力（Encoder-Decoder Attention）、前馈网络等。解码时只能访问当前及已生成的 token，并能聚合编码器输出的信息。

- **连接机制**  
  Decoder Block 中的交叉注意力机制，允许解码器针对每个生成步骤，聚焦并读取编码器处理过的输入信息。

---

### 2. 构建流程

1. **嵌入层（Embedding）**  
   - 输入 token 通过嵌入层映射为向量，加入位置编码。

2. **多层 Encoder Block**  
   - 每一输入 token 经过 N 层编码器，产生上下文相关的隐藏状态。

3. **多层 Decoder Block**  
   - Decoder 分步输入前文 token，每层包括自注意力（仅看见已生成部分）、与编码器输出的交叉注意力，以及前馈网络处理。

4. **输出层**  
   - 最后解码器输出经过线性投影，使用 Softmax 得到下一个 token 的概率。

5. **训练目标**  
   - 通常采用 teacher forcing 方法，用已知目标序列训练，最大化输出 token 的条件概率。

---

### 3. 关键特性

- **输入输出灵活**：支持不同长度的输入输出序列，适合翻译与摘要等 Seq2Seq 任务。
- **条件生成**：解码器可聚合整个输入信息，实现精准的条件生成。
- **强大上下文理解与表达能力**。

---

### 4. 示例（简化伪代码）

```python
# Encoder-Decoder 框架伪代码
input_tokens = ...      # 输入序列
target_tokens = ...     # 目标序列（如翻译结果）

encoder_out = Encoder(Embedding(input_tokens))

# 训练时通常用目标序列的前缀做为解码输入
decoder_in = Embedding(target_tokens[:, :-1])
x = decoder_in
for i in range(num_layers):
    x = DecoderBlock(x, encoder_out)
logits = Linear(x)  # 得到每个位置的下一个 token 预测概率
```

---

### 5. 总结

Encoder-Decoder 是经典的 Seq2Seq 架构，特别适合“输入到输出”成对建模场景。它以编码器捕获输入语义，解码器结合条件生成，可支持高度复杂的文本生成和理解任务，是自然语言处理领域的基础架构之一。
