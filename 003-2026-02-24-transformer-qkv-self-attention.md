# 003-2026-02-24 Transformer 架构详解：从 QKV 到 Self-Attention

## 引言

2017年，Google 在论文《Attention Is All You Need》中提出了 Transformer 架构，这个创新彻底改变了自然语言处理领域。从机器翻译到文本生成，从 BERT 到 GPT，当今最强大的 AI 模型都建立在 Transformer 的基础之上。

但对于初学者来说，Transformer 的原理常常显得晦涩难懂。Self-Attention 是什么？Q、K、V 又是什么意思？Encoder 和 Decoder 如何协作？本文将从零开始，用通俗易懂的方式带你深入理解 Transformer 的核心架构，帮助你从"会用模型"进阶到"懂原理"。

---

## 为什么需要 Transformer？

在 Transformer 出现之前，处理序列数据的主流方法是 RNN（循环神经网络）及其变体 LSTM（长短期记忆网络）。RNN 的核心思想是按顺序处理输入：当前时刻的输出依赖于上一时刻的隐藏状态。

```python
# RNN 的伪代码
hidden_state = initial_state
for word in input_sequence:
    hidden_state = rnn_cell(word, hidden_state)
    output.append(hidden_state)
```

这种设计看似合理，但存在两个致命问题：

**第一，无法并行计算**。由于每个时间步都依赖于前一个时间步，RNN 必须顺序处理，这使得训练速度非常慢。想象一下，如果你要处理一篇 1000 字的文章，RNN 需要逐字处理，无法利用 GPU 的并行计算能力。

**第二，长距离依赖问题**。当序列很长时，早期的信息在传递过程中会逐渐"丢失"。比如在句子"我出生在法国，...（很长一段话）...所以我能说流利的法语"中，RNN 很难将开头的"法国"与结尾的"法语"联系起来。

Transformer 的出现解决了这两个问题。它抛弃了循环结构，完全基于注意力机制，可以并行处理整个序列，并且能够直接捕获序列中任意两个位置之间的关系，无论它们相距多远。

---

## Self-Attention：理解"注意力"的本质

### 什么是注意力？

在理解 Self-Attention 之前，先想想人类是如何阅读的。当你阅读这句话时，你的眼睛并不是平均分配注意力的。读到"Transformer"这个词时，你会更关注它；读到"的"这种虚词时，你可能一扫而过。

这就是注意力的核心：**在处理信息时，不是同等对待所有部分，而是动态地分配不同的权重，关注更重要的部分**。

在自然语言中，一个词的含义往往依赖于上下文。比如"苹果"这个词：
- "我吃了一个苹果" → 这里指水果
- "我买了一台苹果" → 这里指苹果公司

人类可以根据上下文轻松区分，但机器如何做到？答案就是 Self-Attention。

### Self-Attention 的工作原理

Self-Attention 的核心思想是：**让序列中的每个词都能够"看到"其他所有词，并根据相关性分配不同的权重**。

举个例子，考虑句子："那只猫没有穿过街道，因为它太累了"。

当我们处理"它"这个词时，Self-Attention 机制会计算"它"与句子中其他词的相关性：
- 与"猫"的相关性很高（因为"它"指代"猫"）
- 与"街道"的相关性较低
- 与"累了"的相关性很高（因为累所以没穿过街道）

这样，在表示"它"这个词时，模型会更多地参考"猫"和"累了"的信息，而不是"街道"。

用矩阵形式表示，假设输入序列是 X ∈ ℝ^(n×d)，其中 n 是序列长度，d 是嵌入维度：

```
输入矩阵 X:
[词1的向量]
[词2的向量]
[词3的向量]
...
[词n的向量]
```

Self-Attention 会计算一个注意力矩阵 A ∈ ℝ^(n×n)，其中 A[i,j] 表示第 i 个词对第 j 个词的关注程度：

```
注意力矩阵 A (归一化后):
    [词1]  [词2]  [词3]  ...  [词n]
[词1]  0.15   0.30   0.05  ...  0.10
[词2]  0.20   0.25   0.10  ...  0.15
[词3]  0.05   0.10   0.35  ...  0.08
...
[词n]  0.10   0.15   0.08  ...  0.40
```

### 为什么要"自"注意力？

你可能好奇，为什么叫"自"注意力？因为查询、键、值都来自同一个输入序列。换句话说，**序列在关注自己**。

这与传统的注意力机制不同。在早期的 Seq2Seq 模型中，注意力是 Decoder 中的输出序列关注 Encoder 中的输入序列，是跨序列的。而 Self-Attention 是在同一个序列内部进行注意力计算。

---

## QKV：深度解析三剑客

你一定听说过 Q、K、V 这三个字母，它们是 Transformer 中最核心的概念。让我们深入理解它们的含义和作用。

### Q、K、V 分别代表什么？

**Q（Query，查询）**：表示当前词想要"查询"什么信息。可以理解为当前词发出的"请求"。

**K（Key，键）**：表示其他词能够提供的"索引"或"标签"。可以理解为每个词的"名片"。

**V（Value，值）**：表示词的实际"内容"或"信息"。可以理解为每个词的"简历"。

### 为什么需要三个矩阵？

用一个类比来理解：假设你在图书馆查找资料。

- **Query**：你的查询需求（比如"机器学习入门"）
- **Key**：每本书的标签或目录（比如"人工智能"、"机器学习"、"Python"）
- **Value**：书的内容本身

当你查询时，系统会比较你的 Query 和每本书的 Key，计算匹配度（相似度），然后根据匹配度从每本书的 Value 中提取信息，最终得到综合的结果。

在 Transformer 中：
1. 每个词生成一个 Query（我要查询什么）
2. 每个词生成一个 Key（我能提供什么）
3. 每个词生成一个 Value（我的实际内容）

### Attention 计算过程

Self-Attention 的计算可以分为四个步骤：

**步骤1：线性变换**

对于输入序列中的每个词的表示向量 x ∈ ℝ^d，通过三个不同的权重矩阵进行线性变换：

```
Q = X · W_Q    W_Q ∈ ℝ^(d×d_k)
K = X · W_K    W_K ∈ ℝ^(d×d_k)
V = X · W_V    W_V ∈ ℝ^(d×d_v)
```

其中：
- X 是输入矩阵，形状 (n, d)，n 是序列长度，d 是模型维度
- W_Q, W_K, W_V 是可学习的权重矩阵
- d_k 和 d_v 是 Query/Key 和 Value 的维度（通常 d_k = d_v = d / h，h 是头数）

在代码中：

```python
import torch
import torch.nn as nn

class SelfAttention(nn.Module):
    def __init__(self, d_model, d_k, d_v):
        super().__init__()
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_v, bias=False)
    
    def forward(self, X):
        # X: (batch_size, seq_len, d_model)
        Q = self.W_Q(X)  # (batch_size, seq_len, d_k)
        K = self.W_K(X)  # (batch_size, seq_len, d_k)
        V = self.W_V(X)  # (batch_size, seq_len, d_v)
        return Q, K, V
```

**步骤2：计算相关性得分**

用当前词的 Query 与所有词的 Key 做点积，得到相关性得分：

```
Scores = Q · K^T

形状变换：
Q: (batch_size, seq_len, d_k)
K^T: (batch_size, d_k, seq_len)
Scores: (batch_size, seq_len, seq_len)
```

点积可以衡量两个向量的相似度：如果 Q 和 K 的方向一致，点积就大，说明相关性强。

```python
# 计算 Query 和 Key 的点积
scores = torch.matmul(Q, K.transpose(-2, -1))  # (batch_size, seq_len, seq_len)
```

**步骤3：缩放与归一化**

将得分除以缩放因子 √d_k，然后通过 Softmax 归一化：

```
Attention_Weights = softmax(Scores / √d_k)
```

为什么要缩放？当 d_k 很大时，点积的数值会很大，导致 Softmax 进入饱和区，梯度极小。缩放可以缓解这个问题。

```python
import torch.nn.functional as F

d_k = Q.size(-1)
scaled_scores = scores / torch.sqrt(torch.tensor(d_k, dtype=torch.float32))
attention_weights = F.softmax(scaled_scores, dim=-1)  # 在最后一个维度上归一化
```

Softmax 确保每行的权重之和为 1：

```
归一化后的注意力权重:
[0.1, 0.05, 0.6, 0.15, 0.1]  → 总和 = 1.0
```

**步骤4：加权求和**

用得到的权重对所有词的 Value 进行加权求和：

```
Output = Attention_Weights · V

形状：
Attention_Weights: (batch_size, seq_len, seq_len)
V: (batch_size, seq_len, d_v)
Output: (batch_size, seq_len, d_v)
```

```python
# 加权求和
output = torch.matmul(attention_weights, V)  # (batch_size, seq_len, d_v)
```

### 完整的 Self-Attention 代码

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SelfAttention(nn.Module):
    def __init__(self, d_model, d_k, d_v):
        super().__init__()
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_v, bias=False)
        self.d_k = d_k
    
    def forward(self, X):
        batch_size = X.size(0)
        
        # 步骤1：线性变换
        Q = self.W_Q(X)  # (batch_size, seq_len, d_k)
        K = self.W_K(X)  # (batch_size, seq_len, d_k)
        V = self.W_V(X)  # (batch_size, seq_len, d_v)
        
        # 步骤2：计算相关性得分
        scores = torch.matmul(Q, K.transpose(-2, -1))  # (batch_size, seq_len, seq_len)
        
        # 步骤3：缩放与归一化
        scaled_scores = scores / torch.sqrt(torch.tensor(self.d_k, dtype=torch.float32))
        attention_weights = F.softmax(scaled_scores, dim=-1)
        
        # 步骤4：加权求和
        output = torch.matmul(attention_weights, V)  # (batch_size, seq_len, d_v)
        
        return output, attention_weights
```

### 一个具体的例子

考虑句子："The animal didn't cross the street because it was too tired"。

假设我们处理"it"这个词：

```python
# 简化的数值示例
it_query = [0.2, 0.5, 0.3]  # "it" 的 Query

# 其他词的 Key
animal_key = [0.8, 0.1, 0.1]   # "animal"
street_key = [0.1, 0.1, 0.8]   # "street"
tired_key   = [0.3, 0.6, 0.1]   # "tired"

# 计算点积
score_animal = sum(it_query * animal_key) = 0.2*0.8 + 0.5*0.1 + 0.3*0.1 = 0.24
score_street = sum(it_query * street_key) = 0.2*0.1 + 0.5*0.1 + 0.3*0.8 = 0.31
score_tired  = sum(it_query * tired_key)  = 0.2*0.3 + 0.5*0.6 + 0.3*0.1 = 0.39

# 归一化后（假设）
weight_animal = 0.30  # 高权重，因为"it"指代"animal"
weight_street = 0.15  # 低权重
weight_tired  = 0.55  # 最高权重，因为"tired"解释了原因
```

这样，"it"的最终表示会更多地融合"animal"和"tired"的信息，帮助模型理解"it"指的是"animal"，并且因为"tired"所以没有"cross"。

---

## Multi-Head Attention：多头机制

单个 Self-Attention 可能只能捕获一种类型的关系。为了让模型从多个"子空间"理解序列，Transformer 引入了多头机制。

### 为什么需要多头？

想象一下，你在理解一句话时，会同时关注多个方面：
- 语法结构：主语、谓语、宾语的关系
- 语义关系：指代、因果、时态
- 语境信息：情感、话题、领域

单个注意力头难以同时捕获所有这些信息。多头机制让模型可以并行地关注不同的方面。

### Multi-Head 的实现

假设有 h 个头，每个头独立计算 Self-Attention：

```python
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, num_heads):
        super().__init__()
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        # 为每个头创建 Q、K、V 的权重矩阵
        self.W_Q = nn.Linear(d_model, d_model, bias=False)
        self.W_K = nn.Linear(d_model, d_model, bias=False)
        self.W_V = nn.Linear(d_model, d_model, bias=False)
        self.W_O = nn.Linear(d_model, d_model, bias=False)
    
    def forward(self, X):
        batch_size, seq_len, _ = X.size()
        
        # 线性变换并分割成多个头
        Q = self.W_Q(X).view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_K(X).view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_V(X).view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        
        # 每个头独立计算注意力
        # Q, K, V: (batch_size, num_heads, seq_len, d_k)
        scores = torch.matmul(Q, K.transpose(-2, -1)) / torch.sqrt(torch.tensor(self.d_k, dtype=torch.float32))
        attention_weights = F.softmax(scores, dim=-1)
        head_output = torch.matmul(attention_weights, V)  # (batch_size, num_heads, seq_len, d_k)
        
        # 拼接所有头
        concat_output = head_output.transpose(1, 2).contiguous().view(batch_size, seq_len, self.d_model)
        
        # 线性变换得到最终输出
        output = self.W_O(concat_output)  # (batch_size, seq_len, d_model)
        
        return output, attention_weights
```

用矩阵表示：

```
输入 X: (batch_size, seq_len, d_model)
    ↓
[头1, 头2, ..., 头h] 并行计算
    ↓
头1输出: (batch_size, seq_len, d_model/h)
头2输出: (batch_size, seq_len, d_model/h)
...
头h输出: (batch_size, seq_len, d_model/h)
    ↓
拼接: (batch_size, seq_len, d_model)
    ↓
线性变换: (batch_size, seq_len, d_model)
```

---

## Encoder-Decoder 架构

Transformer 采用了经典的 Encoder-Decoder 架构，但与传统的 Seq2Seq 模型不同，它完全基于注意力机制。

### Encoder 的结构

Encoder 由 N 个相同的层堆叠而成（N=6）。每一层包含两个子层：

```python
class EncoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        self.self_attn = MultiHeadAttention(d_model, num_heads)
        self.feed_forward = PositionwiseFeedForward(d_model, d_ff)
        self.sublayer1 = SublayerConnection(d_model, dropout)
        self.sublayer2 = SublayerConnection(d_model, dropout)
    
    def forward(self, X, mask=None):
        # 子层1：Multi-Head Self-Attention + 残差连接 + 层归一化
        X = self.sublayer1(X, lambda x: self.self_attn(x, x, x, mask))
        
        # 子层2：Feed-Forward Network + 残差连接 + 层归一化
        X = self.sublayer2(X, self.feed_forward)
        
        return X
```

**SublayerConnection** 实现残差连接和层归一化：

```python
class SublayerConnection(nn.Module):
    def __init__(self, d_model, dropout):
        super().__init__()
        self.norm = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, X, sublayer):
        # 残差连接：X + sublayer(X)
        # 层归一化：Norm(X + Dropout(sublayer(X)))
        return X + self.dropout(sublayer(self.norm(X)))
```

**Feed-Forward Network**：

```python
class PositionwiseFeedForward(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super().__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, X):
        # FFN(x) = max(0, xW1 + b1)W2 + b2
        return self.linear2(self.dropout(F.relu(self.linear1(X))))
```

用公式表示：

```
FFN(x) = ReLU(x · W_1 + b_1) · W_2 + b_2

其中：
- x: (batch_size, seq_len, d_model)
- W_1: (d_model, d_ff)  通常 d_ff = 4 × d_model
- W_2: (d_ff, d_model)
- 输出: (batch_size, seq_len, d_model)
```

### Decoder 的结构

Decoder 也由 N 层组成，但每层有三个子层：

```python
class DecoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        self.self_attn = MultiHeadAttention(d_model, num_heads)
        self.cross_attn = MultiHeadAttention(d_model, num_heads)
        self.feed_forward = PositionwiseFeedForward(d_model, d_ff)
        self.sublayer1 = SublayerConnection(d_model, dropout)
        self.sublayer2 = SublayerConnection(d_model, dropout)
        self.sublayer3 = SublayerConnection(d_model, dropout)
    
    def forward(self, X, encoder_output, tgt_mask, memory_mask):
        # 子层1：Masked Self-Attention（只能看到已生成的词）
        X = self.sublayer1(X, lambda x: self.self_attn(x, x, x, tgt_mask))
        
        # 子层2：Encoder-Decoder Attention（Query来自Decoder，KV来自Encoder）
        X = self.sublayer2(X, lambda x: self.cross_attn(x, encoder_output, encoder_output, memory_mask))
        
        # 子层3：Feed-Forward Network
        X = self.sublayer3(X, self.feed_forward)
        
        return X
```

**Masked Self-Attention** 的实现：

```python
def generate_tgt_mask(seq_len):
    # 创建一个下三角矩阵，上三角为负无穷大
    mask = torch.triu(torch.ones(seq_len, seq_len) * float('-inf'), diagonal=1)
    return mask

# 在计算注意力时应用 mask
scores = scores + mask  # 被mask的位置得分极低，softmax后接近0
```

用矩阵表示：

```
Seq_len = 4 的 Mask 矩阵:
      词1   词2   词3   词4
词1   0.0  -inf  -inf  -inf
词2   0.0   0.0  -inf  -inf
词3   0.0   0.0   0.0  -inf
词4   0.0   0.0   0.0   0.0

解释：处理词3时，只能看到词1、词2、词3，看不到词4
```

### Encoder-Decoder Attention

这是 Transformer 的关键创新之一：

```python
# Encoder-Decoder Attention 中
# Query 来自 Decoder 的上一层输出
# Key 和 Value 来自 Encoder 的最终输出

# Query: (batch_size, tgt_seq_len, d_model)
# Key: (batch_size, src_seq_len, d_model)
# Value: (batch_size, src_seq_len, d_model)
# Output: (batch_size, tgt_seq_len, d_model)

# 这样，Decoder 在生成每个目标词时，都能参考整个输入序列
```

### 完整的 Transformer 代码框架

```python
class Transformer(nn.Module):
    def __init__(self, src_vocab_size, tgt_vocab_size, d_model=512, num_heads=8, 
                 num_layers=6, d_ff=2048, dropout=0.1):
        super().__init__()
        self.encoder = Encoder(d_model, num_heads, d_ff, num_layers, dropout)
        self.decoder = Decoder(d_model, num_heads, d_ff, num_layers, dropout)
        self.src_embedding = nn.Embedding(src_vocab_size, d_model)
        self.tgt_embedding = nn.Embedding(tgt_vocab_size, d_model)
        self.generator = nn.Linear(d_model, tgt_vocab_size)
    
    def forward(self, src, tgt, src_mask, tgt_mask):
        # 编码
        encoder_output = self.encode(src, src_mask)
        
        # 解码
        decoder_output = self.decode(tgt, encoder_output, tgt_mask, src_mask)
        
        # 生成概率分布
        output = self.generator(decoder_output)
        
        return output
    
    def encode(self, src, src_mask):
        # 嵌入 + 位置编码
        src_embedded = self.src_embedding(src)
        src_embedded = positional_encoding(src_embedded)
        
        # 通过 Encoder 层
        return self.encoder(src_embedded, src_mask)
    
    def decode(self, tgt, encoder_output, tgt_mask, src_mask):
        # 嵌入 + 位置编码
        tgt_embedded = self.tgt_embedding(tgt)
        tgt_embedded = positional_encoding(tgt_embedded)
        
        # 通过 Decoder 层
        return self.decoder(tgt_embedded, encoder_output, tgt_mask, src_mask)
```

---

## 从原理到应用

理解 Transformer 的原理后，你会发现它是当今最强大模型的基础：

**BERT**（Bidirectional Encoder Representations from Transformers）：仅使用 Encoder 部分，通过双向上下文理解，在文本分类、命名实体识别等任务上表现出色。

**GPT**（Generative Pre-trained Transformer）：仅使用 Decoder 部分，通过自回归生成，在文本生成、对话系统等领域大放异彩。

**T5、BART**：同时使用 Encoder 和 Decoder，在机器翻译、文本摘要等 Seq2Seq 任务中领先。

实际应用中，你可以使用 Hugging Face 的 Transformers 库：

```python
from transformers import BertModel, GPT2Model

# 加载预训练的 BERT
bert = BertModel.from_pretrained('bert-base-uncased')
output = bert(input_ids)

# 加载预训练的 GPT-2
gpt2 = GPT2Model.from_pretrained('gpt2')
output = gpt2(input_ids)
```

对于初学者，建议的学习路径是：
1. 理解本文介绍的核心概念（Self-Attention、QKV、Encoder-Decoder）
2. 阅读原始论文《Attention Is All You Need》
3. 使用 PyTorch 或 TensorFlow 实现简单版本
4. 使用 Hugging Face 库在实际项目中应用预训练模型

---

## 结论

Transformer 架构的出现，标志着深度学习进入了一个新时代。它抛弃了复杂的循环结构，用简单而优雅的注意力机制，实现了对序列数据的强大建模能力。

回顾核心要点：
- **Self-Attention** 让每个词都能关注整个序列，计算公式为：`Attention(Q,K,V) = softmax(QK^T/√d_k)V`
- **QKV** 是注意力的三剑客，分别表示查询、键、值，通过线性变换得到
- **Encoder-Decoder** 架构实现了编码-解码的范式，Encoder 提取特征，Decoder 生成输出
- **多头机制** 让模型从多个子空间并行地理解信息
- **残差连接与层归一化** 帮助训练深层网络

对于初学者来说，掌握这些核心概念是理解现代 NLP 模型的关键。不要被复杂的公式吓到，从直觉出发，结合代码实现，你会发现 Transformer 的原理其实很自然。

最后，我想提出一个问题供你思考：既然 Transformer 在序列建模上如此强大，它能否应用于非序列数据，比如图像、图结构数据？事实上，Vision Transformer 已经在计算机视觉领域取得了巨大成功。这启发我们：**优秀的架构往往具有普适性，理解本质比记住形式更重要**。

如果你觉得这篇文章对你有帮助，欢迎点赞、收藏和分享。如果你有任何问题或想深入讨论某个细节，欢迎在评论区交流！
