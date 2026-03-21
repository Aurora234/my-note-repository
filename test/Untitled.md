GPT（Generative Pre-trained Transformer）是由 OpenAI 提出的一系列**自回归语言模型**，其核心思想是：**通过预测下一个词的方式，在大规模无标注文本上进行预训练，学习通用的语言表示能力**。下面我们以 **GPT-1**（2018）为基础，结合 GPT-2/3 的演进，详细解释 GPT 的预训练机制。

---

## 🔷 一、核心预训练任务：**标准语言建模**（Standard Language Modeling）

> **给定前面的词，预测下一个词**（Next Token Prediction）

这是最经典的**自回归**（autoregressive）任务。

### 📌 数学形式：
给定一个词序列 $ x_1, x_2, ..., x_T $，最大化似然：
$$
\mathcal{L} = \sum_{t=1}^{T} \log P(x_t \mid x_1, x_2, ..., x_{t-1})
$$

模型只使用**左侧上下文**（单向），不能看到未来词。

---

## 🔷 二、GPT 的预训练流程详解

### ✅ 1. **模型架构：Transformer 解码器**（Decoder-only）
- 仅使用 **Transformer 的解码器部分**（但去掉 encoder-decoder attention 和 masking for future tokens 已内置）
- 每一层包含：
  - **Masked Multi-Head Self-Attention**（关键！防止看到未来）
  - **Position-wise Feed-Forward Network**
- 使用 **残差连接 + LayerNorm**（Post-LN 结构）

> ⚠️ 注意：GPT 的“解码器”在预训练阶段**没有编码器输入**，只是一个纯自回归生成模型。

---

### ✅ 2. **输入表示**
- **Token Embedding**：将词映射为向量（使用 BPE 分词）
- **Positional Embedding**：加入绝对位置编码（与原始 Transformer 相同）
- 最终输入：  
  $$
  h_0 = W_e x + W_p
  $$
  其中 $ x $ 是 token ID 序列，$ W_e $ 是词嵌入矩阵，$ W_p $ 是位置嵌入。

---

### ✅ 3. **训练目标：预测下一个词**
- 对每个位置 $ t $，模型输出一个词汇表大小的 logits 向量
- 使用 **交叉熵损失**（Cross-Entropy Loss）计算预测与真实下一个词的误差
- **所有位置都参与损失计算**（不像 BERT 只算 masked 位置）

```python
# 伪代码
logits = model(input_ids)          # [batch, seq_len, vocab_size]
loss = cross_entropy(
    logits[:, :-1, :].reshape(-1, vocab_size),   # 预测 x2~xT
    input_ids[:, 1:].reshape(-1)                 # 真实标签 x2~xT
)
```

---

### ✅ 4. **关键机制：因果掩码**（Causal Mask / Look-ahead Mask）
为了保证模型**只能看到当前及之前的词**，在 self-attention 中使用**上三角掩码**：

| Query \ Key | x₁   | x₂   | x₃   | x₄   |
| ----------- | ---- | ---- | ---- | ---- |
| x₁          | ✓    | ✗    | ✗    | ✗    |
| x₂          | ✓    | ✓    | ✗    | ✗    |
| x₃          | ✓    | ✓    | ✓    | ✗    |
| x₄          | ✓    | ✓    | ✓    | ✓    |

- 掩码将未来位置的 attention score 设为 `-inf`
- softmax 后，未来位置权重为 0

> 这就是 `torch.nn.MultiheadAttention` 中 `attn_mask` 或 `is_causal=True` 的作用。

---

## 🔷 三、GPT-1 / GPT-2 / GPT-3 的预训练演进

| 版本      | 发布时间 | 参数量 | 数据集               | 关键特点                                             |
| --------- | -------- | ------ | -------------------- | ---------------------------------------------------- |
| **GPT-1** | 2018     | 117M   | BooksCorpus (≈8亿词) | 首次提出“预训练+微调”范式                            |
| **GPT-2** | 2019     | 1.5B   | WebText (≈40GB网页)  | **去微调**（zero-shot learning），更大规模           |
| **GPT-3** | 2020     | 175B   | Common Crawl + 多源  | **上下文学习**（in-context learning），few-shot 能力 |

> 📌 共同点：**始终使用标准语言建模作为唯一预训练任务**，没有 MLM、NSP 等辅助任务。

---

## 🔷 四、预训练数据与工程细节

| 项目           | GPT-1 示例                                            |
| -------------- | ----------------------------------------------------- |
| **分词器**     | Byte Pair Encoding (BPE)，词表大小 40,000             |
| **最大长度**   | 512 tokens                                            |
| **优化器**     | Adam（β₁=0.9, β₂=0.999），学习率 2.5e-4，warmup 2k 步 |
| **Batch Size** | 64 sequences                                          |
| **训练步数**   | 100 万步                                              |

GPT-2/3 则使用更大 batch、更长训练、更高质量数据。

---

## 🔷 五、预训练 vs 微调（GPT-1 的做法）

虽然 GPT-2/3 主打 zero-shot/few-shot，但 **GPT-1 仍采用“预训练 + 微调”**：

1. **预训练**：在 BooksCorpus 上训练语言模型
2. **微调**：
   - 在下游任务（如分类、蕴含）上继续训练
   - 输入格式统一为：  
     ```
     [x1, x2, ..., xn, [CLS]]
     ```
   - 用 `[CLS]` 位置的输出接分类头
   - **联合优化**：语言模型 loss + 任务 loss

```python
total_loss = task_loss + λ * language_modeling_loss
```

> 后续 GPT 系列发现：只要模型足够大，**无需微调也能完成任务**（通过 prompt 引导）。

---

## 🔷 六、GPT 预训练的优势与局限

### ✅ 优势：
- **简单统一**：只有一个任务，易于扩展到更大规模
- **天然适合生成**：自回归结构可直接用于文本生成
- **可扩展性强**：GPT-3 证明“规模带来智能”

### ❌ 局限：
- **单向上下文**：无法像 BERT 那样利用右侧信息（对某些理解任务不利）
- **预训练-微调不一致**：微调时没有 `[MASK]`，但 GPT 不存在这个问题（因为它不用 mask）

> 🔄 后来出现的 **T5、BART** 等模型尝试融合双向和生成能力。

---

## ✅ 总结：GPT 预训练的核心思想

> **GPT 通过在海量文本上训练一个“预测下一个词”的自回归语言模型，让 Transformer 解码器学会通用的语言知识。它仅依赖标准语言建模任务，利用因果掩码保证单向性，最终获得强大的生成和零样本推理能力。**

从 GPT-1 到 GPT-4，这一范式被不断放大，成为当前大语言模型（LLM）的主流路线之一。