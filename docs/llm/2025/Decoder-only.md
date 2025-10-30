Decoder-only 架构（即仅有解码器的架构）是当前主流大规模语言模型（如 GPT 系列、LLaMA、Bloom 等）的核心技术路线之一。其基本构建方法如下：

---

### 1. 架构基础

- **Transformer 结构**  
  Decoder-only 架构本质上是 Transformer 的解码器（Decoder）堆叠。与 Encoder-Decoder 结构不同，它仅由多层解码器（通常十几到数百层）组成，无需编码器。

- **自回归生成**  
  在训练和生成阶段，模型通过 Masked Self-Attention（掩码自注意力）机制，仅能访问当前位置及其之前的 token，实现逐步输出下一个词元。

---

### 2. 构建流程

1. **嵌入层（Embedding）**  
   输入的 tokens 经过词表查找，被映射为稠密的向量。

2. **多层 Decoder Block 堆叠**  
   每个 block 包含：
   - 掩码自注意力（Masked Multi-Head Self Attention）
   - 残差连接与 LayerNorm
   - 前馈全连接层（MLP/FFN）
   - 再次残差与归一化

   Block 堆叠的层数决定了模型规模（如 GPT-3 有 96 层）。

3. **输出层**  
   最后一个 decoder 输出经线性映射，Softmax 得到每个 token 的概率分布。

4. **训练目标**  
   - 使用大规模语料进行自回归语言建模（即最大化下一个 token 的概率）。

---

### 3. 关键特性

- **仅用解码器，省去编码器结构，专注生成能力。**
- **掩码自注意力保证生成过程中未来信息不可见。**
- **结构高度可扩展，可并行高效训练。**

---

### 4. 示例（简化伪代码）

```python
# 伪代码描述 Decoder-only 大模型主结构
input_tokens = ...  # 输入 token 序列
x = Embedding(input_tokens)

for i in range(num_layers):
    x = DecoderBlock(x, mask=True)  # 掩码保证仅看见前文

logits = Linear(x) # 得到各 token 概率
```

---

### 5. 总结

Decoder-only 架构是现代大规模语言生成模型的核心方案，具备训练和推理高效、生成文本自然流畅等优点，是大模型发展的主流形式。
