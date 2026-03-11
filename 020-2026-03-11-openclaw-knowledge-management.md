# 用 OpenClaw 构建第二大脑：产品经理的个人知识管理实践

> **TL;DR** 作为一名产品经理，我每天面临海量信息碎片：需求文档、技术讨论、行业动态、学习笔记...本文分享如何用 OpenClaw 构建个人知识管理系统，实现信息的**自动采集、智能关联、长期记忆**，让 AI 真正成为你的"第二大脑"。

## 🎯 为什么需要个人知识管理系统？

### 产品经理的信息密度挑战

| 信息类型 | 来源 | 传统处理方式 | 问题 |
|---------|------|------------|------|
| 需求灵感 | 灵光一闪、聊天记录 | 散落在各处 | 容易遗忘 |
| 技术笔记 | 文档、代码注释 | 笔记软件/Git | 检索困难 |
| 行业动态 | Twitter、公众号、RSS | 收藏夹 | 从未回顾 |
| 会议记录 | 飞书、语雀、邮件 | 文件夹 | 知识孤岛 |

**核心痛点**：
- 💔 **碎片化**：信息散落各处，形成孤岛
- 🔍 **检索难**：关键词搜索不精准，关联性弱
- 🧠 **记忆衰减**：人类记忆有遗忘曲线，知识无法沉淀
- ⏰ **时间成本**：手动整理、归档耗时耗力

### AI 时代的知识管理范式

传统知识管理工具（Notion、Obsidian、Roam Research）解决了**存储和检索**，但无法解决**理解和关联**。

AI Agent 的优势：
- 🤖 **自动分类**：识别主题、标签、优先级
- 🔗 **语义关联**：跨越时间窗口找到相关记忆
- 💡 **主动提醒**：根据上下文推送相关知识
- 📝 **知识提炼**：从原始信息中提取洞察

---

## 🏗️ 架构设计

### 系统全景图

```
┌─────────────────────────────────────────────────────────────┐
│                      信息采集层                              │
├─────────────┬─────────────┬─────────────┬─────────────────┤
│  飞书记事   │  GitHub README  │  网页收藏  │  对话记录      │
└──────┬──────┴──────┬──────┴──────┬──────┴──────┬──────────┘
       │             │             │             │
       ▼             ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────┐
│                   OpenClaw Agent                            │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐  │
│  │  memory_search (语义检索)  │  sessions_history (对话) │  │
│  ├─────────────┴─────────────┴─────────────┴─────────────┤  │
│  │  MEMORY.md (长期记忆) + memory/YYYY-MM-DD.md (日记)    │  │
│  └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│                   知识输出层                                 │
├─────────────┬─────────────┬─────────────┬─────────────────┤
│  智能回答   │  定期回顾   │  博客文章   │  产品决策支持   │
└─────────────┴─────────────┴─────────────┴─────────────────┘
```

### 核心组件

| 组件 | 作用 | 配置要点 |
|-----|------|---------|
| **MEMORY.md** | 长期知识库 | 结构化、分类清晰 |
| **memory/YYYY-MM-DD.md** | 每日记事 | 时间序列、原始记录 |
| **memory_search** | 语义检索 | RAG 技术、相关性评分 |
| **sessions_history** | 对话历史 | 跨会话知识关联 |
| **Feishu Doc** | 协作笔记 | 双向同步、版本管理 |

### 数据流向

```
用户输入
  → OpenClaw Agent
  → 实时对话 + memory_search
  → 提取关键信息
  → 写入 MEMORY.md / memory/YYYY-MM-DD.md
  → GitHub 自动备份
  → 飞书记本同步（可选）
```

---

## 📊 实际效果：我的知识管理现状

### 已自动化的部分

✅ **雅思每日新闻**：每日 07:30 自动推送到飞书群
- GitHub 仓库自动更新（https://github.com/yinjianhui/english-learning）
- 已累计 69 篇文章（截至 2026-03-11）
- 脚本：`/root/.openclaw/workspace/scripts/ielts-daily-task.js`

✅ **技术博客双仓库同步**：
- GitHub 主仓库（https://github.com/yinjianhui/tech_blog）
- Gitee 镜像（https://gitee.com/huisir999/tech_blog）
- 已发布 15+ 篇技术文章
- 主题：AI、产品设计、工程实践

✅ **研究笔记结构化**：
- 光纤传感技术研究（4 篇深度分析）
- 存放在 `memory/` 目录，支持语义检索
- 跨会话知识关联，可追溯历史研究

### 数据规模

| 类型 | 数量 | 存储位置 |
|-----|------|---------|
| 每日记事 | 30+ 天 | `memory/YYYY-MM-DD.md` |
| 长期记忆条目 | 50+ | `MEMORY.md` |
| 技术文章 | 15+ | GitHub/Gitee |
| 自动化脚本 | 3 个 | `scripts/` |

---

## 🆚 知识管理工具对比

| 维度 | Notion | Obsidian | OpenClaw |
|-----|--------|----------|----------|
| **存储** | ✅ 云端 | ✅ 本地 | ✅ 本地 + 可选云端 |
| **检索** | 🔍 关键词 | 🔍 全文 + 插件 | 🧠 语义检索（RAG） |
| **关联** | 🔗 数据库 | 🔗 双向链接 | 🔗 跨时间语义关联 |
| **自动化** | ⚙️ API + Zapier | ⚙️ 插件 | 🤖 原生 AI Agent |
| **AI 能力** | 💬 AI Chat（附加） | 🔌 插件（第三方） | 🧠 内置记忆 + 推理 |
| **移动端** | 📱 官方 App | 📱 第三方 | 📱 飞书/Telegram 集成 |
| **成本** | 💰 免费/付费 | 💰 一次性买断 | 🆓 开源（自托管） |

**结论**：OpenClaw 在**语义理解**和**自动化**上有独特优势，特别适合技术用户。

---

## ⚠️ 踩坑记录

### 问题 1：飞书群消息发送失败
**现象**：雅思新闻推送返回 400 错误  
**原因**：消息内容超过长度限制（约 4000 字符）  
**解决**：分段发送或精简内容，移除冗余格式  
**教训**：自动化脚本要先测试再上定时任务

### 问题 2：MEMORY.md 内容泄露风险
**现象**：在群聊中意外加载了敏感配置（API Keys）  
**原因**：未设置主会话检测，所有会话都加载 MEMORY.md  
**解决**：在 `AGENTS.md` 中明确"MEMORY.md 只在主会话加载"  
**教训**：安全边界要清晰，敏感信息独立存储

### 问题 3：检索结果不准确
**现象**：`memory_search` 返回无关内容  
**原因**：查询关键词太泛（如"研究"、"项目"）  
**解决**：用更具体的关键词组合（"光纤传感 研究"）  
**教训**：语义检索需要优化查询策略，添加上下文

### 问题 4：Cron 任务时区混乱
**现象**：雅思新闻推送时间不对（晚了 8 小时）  
**原因**：服务器时区是 UTC，Cron 配置用的是 GMT+8  
**解决**：在 crontab 中明确时区或使用 UTC 时间计算  
**教训**：自动化任务要验证时区配置

---

## 💻 实战实现

### 1. Memory 长期记忆配置

在 workspace 根目录创建 `MEMORY.md`：

```markdown
# MEMORY.md - 长期记忆

## 用户信息
- **姓名**: huisir
- **职业**: 产品经理
- **技能**: 需求分析、原型设计、需求文档编写、项目管理

## 重要项目

### 技术博客
- **GitHub**: https://github.com/yinjianhui/tech_blog
- **Gitee**: https://gitee.com/huisir999/tech_blog
- **主题**: AI、产品设计、工程实践
- **更新频率**: 每周 2-3 篇

### 雅思备考计划
- **目标分数**: 7.0
- **当前进度**: 听力 7.5, 阅读 6.5
- **每日任务**: VOA 慢速英语 + TechCrunch 科技新闻
- **自动化**: 每日 07:30 推送到飞书群

## 技术配置

### GitHub Token
- 路径: `/root/.openclaw/workspace/.config/github-token`
- 权限: repo (读写私有仓库)

### OpenClaw 核心配置
```yaml
channels:
  feishu:
    requireMention: false  # 免@模式
    capabilities:
      inlineButtons: "allowlist"  # 互动卡片

agents:
  default_model: glmcode/glm-4.7
  fallback:
    - deepseek/deepseek-chat
```

## 研究方向
- **文献阅读**: 需要高效阅读学术论文
- **技术领域**: AI、产品设计、分布式系统
- **工具**: 学术研究团队插件 (已安装)
```

**设计原则**：
- ✅ **结构化**：用二级标题分类（用户信息、项目、配置、研究）
- ✅ **可检索**：关键词前置（"GitHub"、"雅思"、"研究"）
- ✅ **时间线**：重大事件标注日期
- ✅ **链接化**：重要资源附 URL

### 2. 每日记事自动化

创建 `scripts/daily-memory.sh`：

```bash
#!/bin/bash
# 每日记事自动创建脚本

MEMORY_DIR="/root/.openclaw/workspace/memory"
TODAY=$(date +%Y-%m-%d)

# 创建每日文件（如果不存在）
if [ ! -f "$MEMORY_DIR/$TODAY.md" ]; then
  cat > "$MEMORY_DIR/$TODAY.md" << EOF
# Daily Notes - $TODAY

## 今日焦点
<!-- 最重要的 3 件事 -->

## 学习笔记
<!-- 新学到的概念、技术、观点 -->

## 灵感记录
<!-- 产品想法、优化点、待研究 -->

## 对话精华
<!-- 值得保留的讨论 -->

## 明日计划
<!-- TODO -->

---
EOF
  echo "✅ 已创建每日记事: $TODAY.md"
else
  echo "ℹ️  今日记事已存在: $TODAY.md"
fi
```

添加到 crontab（每日 00:00 自动创建）：

```bash
0 0 * * * /root/.openclaw/workspace/scripts/daily-memory.sh
```

### 3. 智能检索集成

在 `SOUL.md` 或 `AGENTS.md` 中配置检索规则：

```markdown
## Memory Recall

**触发条件**：
- 用户询问"过去的"、"之前的"、"还记得"
- 涉及人名、项目名、日期的查询
- 需要上下文的复杂问题

**检索流程**：
1. 调用 `memory_search` 搜索 MEMORY.md + memory/*.md
2. 使用 `memory_get` 读取相关片段
3. 结合上下文生成回答
4. 标注来源 `Source: <path>#<line>`

**示例**：
> User: "我之前研究的那个光纤传感项目"
> Claw: [检索 memory/2026-03-02.md]
>      "你在 2026-03-02 研究了光纤传感技术..."
>      Source: memory/2026-03-02.md#L10-50
```

### 4. 飞书记本双向同步

使用 OpenClaw 的 Feishu 集成：

```javascript
// scripts/sync-to-feishu.js
const { execSync } = require('child_process');

// 同步 MEMORY.md 到飞书
async function syncMemoryToFeishu() {
  const memoryContent = execSync('cat /root/.openclaw/workspace/MEMORY.md')
    .toString();

  // 调用 feishu_doc 工具
  // (需要配置 doc_token 和权限)
  await feishuDoc.write({
    doc_token: process.env.FEISHU_MEMORY_DOC_TOKEN,
    content: memoryContent
  });

  console.log('✅ 已同步到飞书');
}

syncMemoryToFeishu();
```

**优势**：
- 📱 移动端访问（飞书 App）
- 👥 团队共享（可协作编辑）
- 🔍 全文搜索（飞书原生支持）

---

## 🚀 高级玩法

### 1. 定期回顾机制

**周日回顾**（每周日晚）：
- 调用 `memory_search` 搜索本周关键词
- 提炼洞察，更新 MEMORY.md
- 删除过期信息，整理分类

**月度复盘**（每月末）：
- 汇总本月学习笔记
- 更新"重要项目"进度
- 调整下月重点方向

### 2. 跨会话知识关联

OpenClaw 的 `sessions_history` 可以检索历史对话：

```markdown
## 场景：产品决策支持

**用户**: "我们之前讨论过 AI 智能体的成本优化方案"

**Agent 流程**:
1. memory_search("AI 智能体 成本 优化")
2. sessions_history(sessionId, limit=50)  // 搜索相关会话
3. 综合记忆和对话历史
4. 生成决策建议
```

### 3. AI 辅助知识提炼

让 OpenClaw 自动从对话中提取知识：

```javascript
// 在对话结束时触发
if (shouldCaptureKnowledge(lastMessage)) {
  const insight = await llm.extractInsight({
    messages: sessionMessages,
    context: "产品经理的工作经验"
  });

  // 追加到 memory/YYYY-MM-DD.md
  await appendToMemoryFile(insight);
}
```

---

## 📋 最佳实践与踩坑

### 写作规范

| 元素 | 规范 | 示例 |
|-----|------|------|
| **标题** | 日期优先 | `2026-03-11 - OpenClaw 知识管理实践` |
| **标签** | 用 emoji | `💡 灵感` `📚 学习` `🔧 技术` |
| **引用** | 标注来源 | `Source: MEMORY.md#L10-20` |
| **链接** | 相对路径 | `[相关文档](./memory/2026-03-10.md)` |

### 命名约定

```
memory/
├── 2026-03-11.md          # 每日记事
├── 2026-03-10.md
├── ielts-task-requirements.md  # 专项任务
└── research/              # 分类目录
    ├── ai-agents.md
    └── product-design.md
```

### 权限管理

⚠️ **安全注意**：
- **MEMORY.md** 只在主会话加载（不在群聊中）
- **API Keys** 存在 `.config/` 目录（不提交 Git）
- **敏感信息** 用占位符（`<TOKEN>`、`<PASSWORD>`）

---

## 🔮 未来优化方向

### 1. 知识图谱

用 Neo4j 或 GraphDB 构建实体关系：

```
(huisir)-[:学习]->(OpenClaw)
(OpenClaw)-[:属于]->(AI Agent)
(AI Agent)-[:应用于]->(知识管理)
```

### 2. 协作分享

- 飞书知识库共享（团队可见）
- GitHub 公开仓库（技术博客）
- Notion 双向链接（公开知识库）

### 3. 多模态支持

- 📸 图片 OCR（截图笔记）
- 🎙️ 语音转文字（会议记录）
- 📄 PDF 提取（论文阅读）

---

## 🎯 总结

用 OpenClaw 构建知识管理系统的核心优势：

| 传统工具 | OpenClaw 方案 |
|---------|--------------|
| 手动归档 | 自动采集 |
| 关键词搜索 | 语义检索 |
| 静态存储 | 动态关联 |
| 单一工具 | 集成生态 |
| 个人使用 | AI 辅助 |

**下一步行动**：
1. ✅ 创建 `MEMORY.md` 结构框架
2. ✅ 配置每日记事自动化
3. ✅ 测试 `memory_search` 检索效果
4. ✅ 对接飞书记本（可选）
5. ✅ 建立每周回顾习惯

---

## 📎 附录：完整配置文件

### A. Workspace 目录结构

```
/root/.openclaw/workspace/
├── MEMORY.md              # 长期记忆（主会话加载）
├── memory/               # 每日记事
│   ├── 2026-03-11.md
│   ├── 2026-03-10.md
│   ├── ielts-task-requirements.md
│   └── research/
│       ├── fiber-sensing.md
│       └── ai-agents.md
├── scripts/              # 自动化脚本
│   ├── ielts-daily-task.js
│   └── daily-memory.sh
├── .config/              # 敏感配置（不提交 Git）
│   ├── github-token
│   ├── gitee-token
│   └── feishu-credentials.json
└── drafts/               # 文章草稿
    └── 001-openclaw-knowledge-management.md
```

### B. OpenClaw 核心配置

```yaml
# ~/.openclaw/config.yaml（示例）
channels:
  feishu:
    requireMention: false  # 免@模式（已在雅思群配置）
    capabilities:
      inlineButtons: "allowlist"  # 互动卡片白名单

agents:
  default_model: glmcode/glm-4.7
  fallback:
    - deepseek/deepseek-chat  # 备用模型
  memory:
    enabled: true
    search:
      provider: local  # 使用本地向量检索
      maxResults: 5    # 最多返回 5 条结果
```

### C. 每日记事自动化脚本

```bash
#!/bin/bash
# scripts/daily-memory.sh

MEMORY_DIR="/root/.openclaw/workspace/memory"
TODAY=$(date +%Y-%m-%d)

# 创建目录（如果不存在）
mkdir -p "$MEMORY_DIR"

# 创建每日文件（如果不存在）
if [ ! -f "$MEMORY_DIR/$TODAY.md" ]; then
  cat > "$MEMORY_DIR/$TODAY.md" << EOF
# Daily Notes - $TODAY

## 📌 今日焦点
<!-- 最重要的 3 件事 -->

## 📚 学习笔记
<!-- 新学到的概念、技术、观点 -->

## 💡 灵感记录
<!-- 产品想法、优化点、待研究 -->

## 💬 对话精华
<!-- 值得保留的讨论 -->

## ✅ 明日计划
<!-- TODO -->

---
EOF
  echo "✅ 已创建每日记事: $TODAY.md"
else
  echo "ℹ️  今日记事已存在: $TODAY.md"
fi
```

添加到 crontab（每日 00:00 自动创建）：

```bash
# 编辑 crontab
crontab -e

# 添加以下行（每天 00:00 执行）
0 0 * * * /root/.openclaw/workspace/scripts/daily-memory.sh >> /var/log/daily-memory.log 2>&1
```

### D. GitHub Actions 自动同步到 Gitee

```yaml
# .github/workflows/sync-to-gitee.yml
name: Sync to Gitee
on:
  push:
    branches: [main]
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Sync to Gitee
        uses: wearerequired/git-mirror-action@master
        env:
          SSH_PRIVATE_KEY: ${{ secrets.GITEE_SSH_KEY }}
        with:
          source-repo: "git@github.com:yinjianhui/tech_blog.git"
          destination-repo: "git@gitee.com:huisir999/tech_blog.git"
```

### E. Memory 检索规则配置

在 `AGENTS.md` 中添加：

```markdown
## Memory Recall 规则

### 触发条件
- 用户询问"过去的"、"之前的"、"还记得"
- 涉及人名、项目名、日期的查询
- 需要上下文的复杂问题

### 检索流程
1. 调用 `memory_search` 搜索 MEMORY.md + memory/*.md
2. 使用 `memory_get` 读取相关片段
3. 结合上下文生成回答
4. 标注来源 `Source: <path>#<line>`

### 安全规则
- **MEMORY.md 只在主会话加载**（不在群聊中）
- API Keys 存在 `.config/` 目录（不提交 Git）
- 敏感信息用占位符（`<TOKEN>`、`<PASSWORD>`）
```

### F. Feishu Webhook 配置（可选）

```javascript
// scripts/feishu-webhook.js（示例）
const https = require('https');

function sendToFeishu(webhookUrl, content) {
  const data = JSON.stringify({
    msg_type: 'text',
    content: {
      text: content
    }
  });

  return new Promise((resolve, reject) => {
    const req = https.request(webhookUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      }
    }, (res) => {
      let body = '';
      res.on('data', (chunk) => body += chunk);
      res.on('end', () => resolve(body));
    });

    req.on('error', reject);
    req.write(data);
    req.end();
  });
}

// 使用示例
sendToFeishu(process.env.FEISHU_WEBHOOK_URL, '✅ 知识库已更新');
```

---

## 🔗 相关资源

### 官方文档
- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [OpenClaw GitHub 仓库](https://github.com/openclaw/openclaw)
- [Feishu 开放平台](https://open.feishu.cn/)

### 我的文章
- [Zvec 入门指南](https://github.com/yinjianhui/tech_blog/blob/main/001-2026-02-23-zvec-guide.md)
- [App 到智能体 API 转型](https://github.com/yinjianhui/tech_blog/blob/main/002-2026-02-23-app-to-agent-api.md)
- [Transformer 架构详解](https://github.com/yinjianhui/tech_blog/blob/main/003-2026-02-24-transformer-architecture.md)

### 推荐阅读
- [Building a Second Brain](https://buildingasecondbrain.com/) - Tiago Forte
- [How to Take Smart Notes](https://www.goodreads.com/book/show/34523069-how-to-take-smart-notes) - Sönke Ahrens
- [Obsidian 官方文档](https://help.obsidian.md/)

---
