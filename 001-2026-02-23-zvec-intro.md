# 001-2026-02-23 Zvec 入门完全指南：从零开始掌握阿里巴巴开源向量数据库

> 本文基于与 AI 助手的实际对话整理而成，适合新手快速入门向量数据库技术。

## 📚 目录

- [什么是 Zvec？](#什么是-zvec)
- [为什么要用向量数据库？](#为什么要用向量数据库)
- [快速上手](#快速上手)
- [核心概念解析](#核心概念解析)
  - [什么是向量？](#什么是向量)
  - [什么是 384 维向量？](#什么是-384-维向量)
  - [Token vs 句子：向量的归属](#token-vs-句子向量的归属)
  - [Transformer 的层级结构](#transformer-的层级结构)
- [实战应用](#实战应用)
  - [场景一：文档语义搜索](#场景一文档语义搜索)
  - [场景二：图片相似度搜索](#场景二图片相似度搜索)
- [持久化与存储](#持久化与存储)
- [常见问题解答](#常见问题解答)
- [学习资源](#学习资源)

---

## 什么是 Zvec？

**Zvec** 是阿里巴巴开源的一款**轻量级、超高速的进程内向量数据库**。它基于阿里巴巴经过实战验证的 Proxima 向量搜索引擎构建，可以直接嵌入到应用程序中，提供生产级别的低延迟、可扩展的相似度搜索能力。

### 核心特性

| 特性 | 描述 |
|------|------|
| ⚡ **闪电般快速** | 可在毫秒级搜索数十亿向量 |
| 🎯 **开箱即用** | 安装即可使用，无需配置服务器 |
| 🔀 **混合搜索** | 支持稠密向量和稀疏向量，可结合结构化过滤 |
| 📦 **进程内运行** | 无需独立部署，随应用启动 |
| 🌐 **跨平台** | 支持 Linux (x86_64/ARM64) 和 macOS (ARM64) |

---

## 为什么要用向量数据库？

### 传统搜索的局限

传统关键词搜索（如 `LIKE '%关键词%'`）存在以下问题：

```python
# 传统 SQL 搜索
SELECT * FROM documents WHERE content LIKE '%手机%'
# 只能找到包含"手机"这两个字的文档
```

**问题：**
- ❌ 无法理解语义（"手机"和"智能手机"是同义词但搜不到）
- ❌ 无法处理多语言（英文 "smartphone" 搜不到）
- ❌ 无法跨模态（无法用图片搜图片）

### 向量搜索的优势

向量搜索通过将数据转换成高维向量，利用数学计算相似度：

```python
# 向量搜索示例
query = "什么是智能手机？"
# 自动找到包含"手机"、"移动设备"等相关概念的文档
```

**优势：**
- ✅ 理解语义（同义词、近义词都能搜到）
- ✅ 跨语言能力（中文搜索能找到英文文档）
- ✅ 多模态支持（图片搜图片、音频搜音频）
- ✅ 速度极快（数学计算，毫秒级响应）

---

## 快速上手

### 环境要求

- **Python 版本**：3.10 - 3.12
- **操作系统**：Linux (x86_64/ARM64) 或 macOS (ARM64)

### 安装 Zvec

```bash
pip install zvec
```

### 一分钟快速体验

```python
import zvec

# 1. 定义集合架构
schema = zvec.CollectionSchema(
    name="example",
    vectors=zvec.VectorSchema("embedding", zvec.DataType.VECTOR_FP32, 4),
)

# 2. 创建集合并打开（数据会持久化到 ./zvec_example 目录）
collection = zvec.create_and_open(path="./zvec_example", schema=schema)

# 3. 插入向量数据
collection.insert([
    zvec.Doc(id="doc_1", vectors={"embedding": [0.1, 0.2, 0.3, 0.4]}),
    zvec.Doc(id="doc_2", vectors={"embedding": [0.2, 0.3, 0.4, 0.1]}),
])

# 4. 搜索相似向量
results = collection.query(
    zvec.VectorQuery("embedding", vector=[0.4, 0.3, 0.3, 0.1]),
    topk=10
)

print(results)
# 输出: [{'id': 'doc_1', 'score': 0.95}, {'id': 'doc_2', 'score': 0.82}]
```

---

## 核心概念解析

### 什么是向量？

**向量**就是一组有序的数字列表，在数学中用于表示大小和方向。在 AI 中，向量被用来表示数据（文本、图片、音频等）的**特征**。

#### 低维向量的可视化

```python
# 1 维向量：直线上的一个点
[0.5]

# 2 维向量：平面上的点 (x, y)
[0.5, 0.3]  # x=0.5, y=0.3
```

在纸上画出来：

```
   y
   ↑
 0.3   • (0.5, 0.3)
   |
   +─────→ x
    0.5
```

```python
# 3 维向量：空间中的点 (x, y, z)
[0.5, 0.3, 0.8]  # x=0.5, y=0.3, z=0.8
```

这就是我们现实世界中的三维空间，可以表示长宽高。

### 什么是 384 维向量？

**384 维向量**就是包含 384 个数字的数组：

```python
[0.5, 0.3, 0.8, 0.2, 0.7, ..., 0.1]  # 共 384 个数字
```

#### 为什么需要这么多维度？

每**一维都代表一个极其细小的语义特征**。虽然人类无法解释每一维的具体含义，但组合起来能精确表达文本的语义。

**类比：用数字描述一个人**

| 维度数 | 描述能力 | 示例 |
|--------|----------|------|
| 2 维 | 极其粗糙 | `[身高: 175, 体重: 70]` |
| 10 维 | 基础信息 | `[身高, 体重, 年龄, 性别, 眼睛颜色, 头发颜色, 血型, 职业, 学历, 居住地]` |
| 384 维 | 细致入微 | `[积极情感, 技术相关性, 未来导向, 正式语气, ...]` （384 个特征） |

**384 维的关键点：**

1. **由 AI 模型自动学习** —— 不是人工定义的
2. **每一维捕获不同特征** —— 可能表示情感、主题、语气等
3. **组合起来表示完整语义** —— 单个维度的含义不重要，组合才有意义

#### 384 维是怎么来的？

维度由**模型架构决定**，不同的 AI 模型输出不同维度的向量：

| 模型 | 维度 | 用途 |
|------|------|------|
| `all-MiniLM-L6-v2` | **384** | 小巧快速，通用场景 |
| `sentence-t5-base` | **768** | 更精确，中等大小 |
| `LaBSE` | **768** | 多语言翻译 |
| `openai/clip-vit-base` | **512** | 图像+文本 |
| `openai/text-embedding-3-large` | **3072** | OpenAI 付费模型，超强 |

#### 模型的内部结构

以 `all-MiniLM-L6-v2` 为例，输出 384 维向量的过程：

```
输入文本
    ↓
嵌入层 (Embedding Layer)  ← 每个词变成 384 维向量
    ↓
6 层 Transformer (L1 ~ L6)  ← 层层提取和抽象特征
    ↓
池化层 (Pooling Layer)  ← 整句合并成一个 384 维向量
    ↓
输出: [0.5, 0.3, 0.8, ...]  ← 384 维！
```

### Token vs 句子：向量的归属

这是一个关键问题：**向量是在 Token 级别还是句子级别？**

#### 答案：**两者都有！**

#### 完整流程

```
输入句子: "我喜欢编程"
    ↓
【分词】Tokenization
    ↓
["我", "喜欢", "编程"]  # 3 个 tokens
    ↓
【嵌入层】Embedding Layer
    ↓
每个 token 变成 384 维向量:
- "我"   → [0.1, 0.2, 0.3, ..., 0.5]  (384 维)
- "喜欢" → [0.4, 0.5, 0.6, ..., 0.8]  (384 维)
- "编程" → [0.7, 0.8, 0.9, ..., 0.2]  (384 维)
    ↓
形状: (3, 384)  # 3 个 token，每个 384 维
    ↓
【Transformer 层】Layer 1 ~ 6
    ↓
每个 token 的向量不断更新:
- "我"   → [0.2, 0.3, 0.4, ..., 0.6]  (384 维，已更新)
- "喜欢" → [0.5, 0.6, 0.7, ..., 0.9]  (384 维，已更新)
- "编程" → [0.8, 0.9, 0.1, ..., 0.3]  (384 维，已更新)
    ↓
形状仍然是: (3, 384)  # 仍然是 3 个 token
    ↓
【池化层】Pooling Layer
    ↓
合并成一个句子向量:
- 句子 → [0.4, 0.6, 0.7, ..., 0.5]  (384 维)
    ↓
形状: (384,)  # 1 个句子向量
```

#### 总结

| 阶段 | 维度归属 | 形状 | 说明 |
|------|----------|------|------|
| 嵌入层 | Token 级别 | `(N, 384)` | N=token 数量 |
| Transformer 1-6 层 | Token 级别 | `(N, 384)` | 每层更新 token 向量 |
| 池化层后 | 句子级别 | `(384,)` | 合并成 1 个句子向量 |

#### 为什么 Token 级别很重要？

**例子：理解上下文**

```
句子: "苹果是一种水果，但苹果手机很贵"
```

- Token "苹果" 第一次出现 → 初始向量 `[0.3, 0.5, ...]`
- Token "水果" → `[0.7, 0.2, ...]`
- Token "手机" → `[0.1, 0.9, ...]`
- Token "苹果" 第二次出现 → 初始向量 `[0.3, 0.5, ...]`

**经过 Transformer 层后（通过自注意力机制）：**

- 第一个 "苹果" 看到了后面的 "水果"，向量更新为 `[0.8, 0.3, ...]` → 表示"水果"
- 第二个 "苹果" 看到了后面的 "手机"，向量更新为 `[0.2, 0.7, ...]` → 表示"科技产品"

**结果：同一个词，根据上下文有了不同的表示！**

### Transformer 的层级结构

#### 层级架构图

```
┌─────────────────────────────────────┐
│         Pooling Layer (池化层)       │ ← 合并 token 成句子
├─────────────────────────────────────┤
│         Transformer Layer 6         │ ← 第 6 层（最抽象）
├─────────────────────────────────────┤
│         Transformer Layer 5         │ ← 第 5 层
├─────────────────────────────────────┤
│         Transformer Layer 4         │ ← 第 4 层
├─────────────────────────────────────┤
│         Transformer Layer 3         │ ← 第 3 层
├─────────────────────────────────────┤
│         Transformer Layer 2         │ ← 第 2 层
├─────────────────────────────────────┤
│         Transformer Layer 1         │ ← 第 1 层（最基础）
├─────────────────────────────────────┤
│      Embedding Layer (嵌入层)        │ ← 初始化 token 向量
└─────────────────────────────────────┘
```

#### 每一层的核心组件

每个 Transformer 层包含：

```
┌─────────────────────────────┐
│  Self-Attention (自注意力)    │  ← 让 token 关注其他 token
├─────────────────────────────┤
│  Feed Forward Network (FFN)  │  ← 处理和转换特征
├─────────────────────────────┤
│  Residual Connection (残差)  │  ← 保留原始信息
├─────────────────────────────┤
│  Layer Normalization (归一化)│  ← 稳定训练
└─────────────────────────────┘
```

#### 层的作用：层层抽象

**类比：理解一张图片**

| 层级 | 类比 | 处理内容 |
|------|------|----------|
| Layer 1 | 看到颜色和线条 | "红色"、"圆形" |
| Layer 2 | 识别基本形状 | "苹果"、"车轮" |
| Layer 3 | 识别物体组合 | "苹果在桌子上" |
| Layer 4 | 理解场景 | "在餐厅" |
| Layer 5 | 理解动作 | "正在吃苹果" |
| Layer 6 | 理解抽象意图 | "享受美食" |

**文本同理：**

| 层级 | 处理内容 | 示例 |
|------|----------|------|
| Layer 1 | 基础语法和词性 | 名词、动词、形容词 |
| Layer 2 | 短语理解 | "红色的苹果" |
| Layer 3 | 句法结构 | 主谓宾关系 |
| Layer 4 | 本地语义 | "这个苹果很好吃" |
| Layer 5 | 全局语义 | 这句话在讨论水果 |
| Layer 6 | 高级抽象 | 积极情感、推荐语气 |

#### 自注意力机制（Self-Attention）可视化

句子：`"那只猫追着球跑"`

```
注意力权重（简化的注意力图）：

          我   只   那   猫   追   着   球   跑
       ┌────┬────┬────┬────┬────┬────┬────┬────┐
   我   │0.9 │0.1 │0.1 │0.1 │0.1 │0.1 │0.1 │0.1 │
       ├────┼────┼────┼────┼────┼────┼────┼────┤
   只   │0.2 │0.8 │0.6 │0.1 │0.1 │0.1 │0.1 │0.1 │
       ├────┼────┼────┼────┼────┼────┼────┼────┤
   那   │0.1 │0.7 │0.9 │0.3 │0.1 │0.1 │0.1 │0.1 │  ← "那"主要关注"猫"
       ├────┼────┼────┼────┼────┼────┼────┼────┤
   猫   │0.1 │0.3 │0.6 │0.9 │0.5 │0.3 │0.4 │0.2 │  ← "猫"被"追"修饰
       ├────┼────┼────┼────┼────┼────┼────┼────┤
   追   │0.1 │0.1 │0.2 │0.6 │0.9 │0.5 │0.7 │0.4 │  ← "追"关注"猫"和"球"
       ├────┼────┼────┼────┼────┼────┼────┼────┤
   着   │0.1 │0.1 │0.1 │0.3 │0.6 │0.8 │0.5 │0.3 │
       ├────┼────┼────┼────┼────┼────┼────┼────┤
   球   │0.1 │0.1 │0.2 │0.4 │0.7 │0.4 │0.9 │0.5 │  ← "球"被"追"和"跑"修饰
       ├────┼────┼────┼────┼────┼────┼────┼────┤
   跑   │0.1 │0.1 │0.1 │0.3 │0.6 │0.3 │0.7 │0.9 │
       └────┴────┴────┴────┴────┴────┴────┴────┘
       ↑
    高亮 = 强关注（权重高）
```

#### 上下文理解的实际效果

**"苹果" 在不同句子中的向量变化：**

```
句子 1: "苹果是一种水果"
         "苹果" 的向量 → [0.8, 0.2, ...]  ← 表示"水果"

句子 2: "苹果手机很好用"
         "苹果" 的向量 → [0.2, 0.8, ...]  ← 表示"品牌"
```

**为什么？** 因为自注意力机制让 "苹果" 关注了不同的上下文词！

---

## 实战应用

### 场景一：文档语义搜索

构建一个智能文档搜索系统，支持自然语言查询。

```python
import zvec
from sentence_transformers import SentenceTransformer

# 加载文本转向量模型
model = SentenceTransformer('all-MiniLM-L6-v2')

# 定义集合（384 维向量）
schema = zvec.CollectionSchema(
    name="documents",
    vectors=zvec.VectorSchema("content_embedding", zvec.DataType.VECTOR_FP32, 384),
)

# 创建集合并打开（数据持久化到 ./doc_db）
collection = zvec.create_and_open(path="./doc_db", schema=schema)

# 准备文档数据
documents = [
    {
        "id": "doc_001",
        "title": "网络安全规范 v1.0",
        "content": """
        本规范规定了网络安全的基本要求，包括：
        1. 密码复杂度要求：至少8位，包含大小写字母、数字和特殊字符
        2. 定期更换密码：每90天更换一次
        3. 禁止共享账号
        4. 重要数据必须加密存储
        """
    },
    {
        "id": "doc_002",
        "title": "数据隐私保护规范",
        "content": """
        数据隐私保护规范包括：
        1. 个人敏感信息采集必须获得用户明确授权
        2. 数据存储不得超过必要期限
        3. 数据传输必须使用加密通道
        4. 发生数据泄露需在24小时内上报
        """
    },
    {
        "id": "doc_003",
        "title": "系统运维手册",
        "content": """
        系统运维手册：
        1. 每日检查服务器状态
        2. 每周备份数据库
        3. 每月进行安全审计
        4. 监控系统资源使用情况
        """
    },
]

# 上传文档
print("📤 开始上传文档...")
for doc in documents:
    # 将文本转换成向量
    embedding = model.encode(doc["content"]).tolist()

    # 插入到 Zvec
    collection.insert([
        zvec.Doc(
            id=doc["id"],
            fields={"title": doc["title"]},
            vectors={"content_embedding": embedding}
        )
    ])
    print(f"✅ 已上传: {doc['title']}")

# 搜索示例
print("\n🔍 开始搜索...\n")

queries = [
    "密码应该多久换一次？",
    "用户数据如何保护？",
    "怎么备份系统？"
]

for query in queries:
    # 将查询文本转换成向量
    query_vector = model.encode(query).tolist()

    # 在 Zvec 中搜索
    results = collection.query(
        zvec.VectorQuery("content_embedding", vector=query_vector),
        topk=2
    )

    print(f"查询: {query}")
    for r in results:
        print(f"  - 文档: {r.get('title', 'N/A')} (相似度: {r['score']:.3f})")
    print()
```

**输出示例：**

```
查询: 密码应该多久换一次？
  - 文档: 网络安全规范 v1.0 (相似度: 0.876)
  - 文档: 系统运维手册 (相似度: 0.342)

查询: 用户数据如何保护？
  - 文档: 数据隐私保护规范 (相似度: 0.891)
  - 文档: 网络安全规范 v1.0 (相似度: 0.523)

查询: 怎么备份系统？
  - 文档: 系统运维手册 (相似度: 0.915)
  - 文档: 网络安全规范 v1.0 (相似度: 0.378)
```

### 场景二：图片相似度搜索

使用预训练的 CNN 模型提取图片特征，实现以图搜图。

```python
import zvec
from torchvision.models import resnet18
import torch
from PIL import Image
import torchvision.transforms as transforms

# 加载预训练图片模型
model = resnet18(pretrained=True)
model.eval()

# 图片预处理
transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
])

def get_image_embedding(image_path):
    """提取图片的向量表示"""
    img = Image.open(image_path).convert('RGB')
    img_t = transform(img)
    batch_t = torch.unsqueeze(img_t, 0)

    with torch.no_grad():
        embedding = model(batch_t)

    return embedding.flatten().tolist()

# 定义集合（512 维向量）
schema = zvec.CollectionSchema(
    name="images",
    vectors=zvec.VectorSchema("image_embedding", zvec.DataType.VECTOR_FP32, 512),
)

collection = zvec.create_and_open(path="./image_db", schema=schema)

# 上传图片向量
image_paths = ["cat1.jpg", "cat2.jpg", "dog1.jpg"]
print("📤 开始上传图片...")

for i, path in enumerate(image_paths):
    embedding = get_image_embedding(path)
    collection.insert([
        zvec.Doc(
            id=path,
            vectors={"image_embedding": embedding}
        )
    ])
    print(f"✅ 已上传: {path}")

# 搜索相似图片
query_image = "query_cat.jpg"
query_embedding = get_image_embedding(query_image)

print(f"\n🔍 搜索与 {query_image} 相似的图片...")

results = collection.query(
    zvec.VectorQuery("image_embedding", vector=query_embedding),
    topk=3
)

for r in results:
    print(f"  - 相似图片: {r['id']} (相似度: {r['score']:.3f})")
```

---

## 持久化与存储

### Zvec 的持久化机制

**Zvec 自动持久化到磁盘**，无需手动管理。

```python
# 创建时指定路径，数据会自动保存到磁盘
collection = zvec.create_and_open(path="./my_vector_db", schema=schema)

# 即使程序重启，数据仍然存在：
# 1. 第一次运行：创建新数据库
# 2. 之后的运行：自动打开已存在的数据库
```

### 持久化文件结构

运行后，会在指定目录生成持久化文件：

```
./my_vector_db/
├── data/           # 向量数据
├── meta/           # 元数据
└── ...             # 其他内部文件
```

**不需要手动管理这些文件**，Zvec 会自动处理。

### 文档管理系统的完整实现

```python
import zvec
from sentence_transformers import SentenceTransformer
from pathlib import Path
import json

class DocumentManager:
    """文档管理系统：结合 Zvec 向量搜索和元数据存储"""

    def __init__(self, db_path):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.schema = zvec.CollectionSchema(
            name="documents",
            vectors=zvec.VectorSchema("content_embedding", zvec.DataType.VECTOR_FP32, 384),
        )
        self.collection = zvec.create_and_open(path=db_path, schema=self.schema)

        # 额外元数据存储（Zvec 主要存向量，额外元数据存 JSON）
        self.meta_path = Path(db_path) / "metadata.json"
        self.metadata = self._load_metadata()

    def _load_metadata(self):
        """加载元数据"""
        if self.meta_path.exists():
            with open(self.meta_path, 'r', encoding='utf-8') as f:
                return json.load(f)
        return {}

    def _save_metadata(self):
        """保存元数据"""
        with open(self.meta_path, 'w', encoding='utf-8') as f:
            json.dump(self.metadata, f, ensure_ascii=False, indent=2)

    def upload(self, doc_id, title, content, extra_meta=None):
        """
        上传文档

        Args:
            doc_id: 文档唯一标识符
            title: 文档标题
            content: 文档内容
            extra_meta: 额外的元数据字典
        """
        # 将文本转换成向量
        embedding = self.model.encode(content).tolist()

        # 存储向量到 Zvec
        self.collection.insert([
            zvec.Doc(
                id=doc_id,
                fields={"title": title},
                vectors={"content_embedding": embedding}
            )
        ])

        # 保存完整元数据（包含原始内容）
        self.metadata[doc_id] = {
            "title": title,
            "content": content,
            **(extra_meta or {})
        }
        self._save_metadata()

        return f"✅ 已上传: {title}"

    def search(self, query, topk=5):
        """
        搜索文档

        Args:
            query: 查询文本
            topk: 返回前 K 个结果

        Returns:
            包含元数据的搜索结果列表
        """
        # 将查询转换成向量
        query_vector = self.model.encode(query).tolist()

        # 在 Zvec 中搜索
        results = self.collection.query(
            zvec.VectorQuery("content_embedding", vector=query_vector),
            topk=topk
        )

        # 补充元数据
        enriched_results = []
        for r in results:
            doc_id = r['id']
            if doc_id in self.metadata:
                r['meta'] = self.metadata[doc_id]
            enriched_results.append(r)

        return enriched_results

    def get_document(self, doc_id):
        """获取完整文档"""
        return self.metadata.get(doc_id, None)

# 使用示例
manager = DocumentManager("./my_doc_db")

# 上传文档
manager.upload(
    doc_id="001",
    title="网络安全规范",
    content="密码必须包含大小写字母、数字和特殊字符，长度至少8位...",
    extra_meta={"author": "安全部", "date": "2024-01-01"}
)

# 搜索
results = manager.search("密码有什么要求？", topk=3)

# 显示结果
for r in results:
    print(f"文档: {r.get('meta', {}).get('title', 'N/A')}")
    print(f"相似度: {r['score']:.3f}")
    print(f"内容预览: {r.get('meta', {}).get('content', 'N/A')[:100]}...")
    print()
```

### 关键概念：文件存储 vs 向量数据库

| 特性 | 文件存储（MongoDB、文件系统） | 向量数据库（Zvec） |
|------|------------------------------|---------------------|
| **存储内容** | 原始文件（PDF、Word、图片等） | **向量**（数字数组） |
| **用途** | 保存文档内容、元数据 | 相似度搜索 |
| **关系** | 存储原始文档 | 存储文档的**向量表示** |

**Zvec 不是用来存储原始文件的**，而是存储文件的**向量表示**（embeddings）。原始内容应该单独存储（文件、数据库、JSON 等）。

---

## 常见问题解答

### Q1: 能直接上传 PDF 文件吗？

**A:** 不能。Zvec 只存储向量，不存储原始文件。

正确流程：
```
PDF 文件
  ↓
提取文本内容
  ↓
文本转向量
  ↓
存储向量到 Zvec
```

原始 PDF 可以存储在文件系统或对象存储中，Zvec 只负责向量搜索。

### Q2: 384 维越大越好吗？

**A:** 不是。维度需要在精度和性能之间平衡：

| 维度 | 优点 | 缺点 |
|------|------|------|
| 低（128-256） | 快速、内存占用小 | 精度可能不足 |
| 中（384-768） | 平衡 | 性能耗费适中 |
| 高（1024+） | 精度高 | 慢、内存占用大 |

选择原则：
- 嵌入式设备：低维度（128-256）
- 通用场景：中等维度（384-768）
- 高精度需求：高维度（1024+）

### Q3: Zvec 和其他向量数据库的区别？

| 数据库 | 部署方式 | 优点 | 适用场景 |
|--------|----------|------|----------|
| **Zvec** | 进程内库 | 轻量、快速、易集成 | 单机应用、边缘设备 |
| **Milvus** | 独立服务器 | 功能强大、可扩展 | 大规模分布式系统 |
| **Qdrant** | 独立服务器 | 易用、支持过滤 | 生产环境 |
| **Pinecone** | 云服务 | 托管、免运维 | 快速原型、中小规模 |

### Q4: 如何选择向量模型？

根据场景选择：

| 场景 | 推荐模型 |
|------|----------|
| 通用文本搜索 | `all-MiniLM-L6-v2` (384维) |
| 多语言 | `sentence-t5-base` (768维) |
| 图像+文本 | `openai/clip-vit-base` (512维) |
| 最高精度 | `openai/text-embedding-3-large` (3072维) |
| 本地部署（中文） | `shibing624/text2vec-base-chinese` (768维) |

### Q5: Zvec 支持哪些过滤条件？

支持结构化过滤：

```python
results = collection.query(
    zvec.VectorQuery("embedding", vector=query_vector),
    topk=10,
    filters={"category": "tech"}  # 只搜索分类为 tech 的文档
)
```

---

## 学习资源

### 官方资源

- **Zvec 官网**: https://zvec.org/
- **快速开始**: https://zvec.org/en/docs/quickstart/
- **完整文档**: https://zvec.org/en/docs/
- **性能基准**: https://zvec.org/en/docs/benchmarks/
- **GitHub 仓库**: https://github.com/alibaba/zvec

### 社区

- **Discord**: https://discord.gg/rKddFBBu9z
- **X (Twitter)**: https://x.com/zvec_ai

### 相关技术

- **SentenceTransformers**: https://www.sbert.net/
- **Hugging Face**: https://huggingface.co/
- **向量数据库入门**: https://www.pinecone.io/learn/vector-database/

---

## 总结

本文从零开始，详细讲解了 Zvec 向量数据库的核心概念和使用方法：

✅ **基础概念**：理解了什么是向量、什么是 384 维
✅ **模型架构**：掌握了 Transformer 的层级结构和自注意力机制
✅ **Token vs 句子**：理解了向量在不同阶段的作用
✅ **实战应用**：学会了文档搜索和图片搜索的实现
✅ **持久化机制**：掌握了 Zvec 的数据存储方式

**下一步建议：**

1. 在本地运行本文的代码示例
2. 尝试用自己的数据构建向量搜索系统
3. 探索不同的向量模型和维度选择
4. 学习更高级的过滤和混合搜索功能

---

> **写作时间**：2026年2月23日
>
> **作者**：基于与 OpenClaw AI 助手的对话整理
>
> **许可证**：本文采用 CC BY-NC-SA 4.0 许可证
