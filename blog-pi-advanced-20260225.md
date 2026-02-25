# Pi Agent 进阶：从 Skill 到完整业务系统的架构之路

## 引言

在前两篇文章中，我们分别介绍了：

- **第一篇**《OpenClaw 火爆背后：揭秘 Pi Agent 框架的极简哲学》：了解了 Pi 的设计哲学和核心理念
- **第二篇**《Pi Agent 实战：手把手教你构建第一个 AI Agent》：学会了如何创建一个简单的文档写作 Agent

但是，当你真正想把 Pi 用到生产环境，构建一个**可扩展、可维护、多用户协作**的业务系统时，会遇到很多新问题：

- 如何设计高质量的 Skill？
- 如何管理多个 Agent 之间的协作？
- 如何构建前端 + 后端 + Pi 的完整架构？
- 如何处理权限、任务队列、错误重试？

这篇文章将带你深入探索这些问题的答案。我们将：

1. **深度解析 Pi 的 Skill 机制**：理解其工作原理，掌握编写高质量 Skill 的技巧
2. **设计基于 Pi 的业务系统架构**：从零设计一个企业级的 Agent 系统
3. **完整实战案例**：构建一个智能内容管理系统，从前端到后端到部署，全流程实战

准备好了吗？让我们开始。

---

## 一、Pi 的 Skill 机制深度解析

### 1.1 Skill 的本质：从文本到执行

在 Pi 中，Skill 的本质是什么？

**答案**：Skill 是**描述 Agent 行为的文本**，Agent 通过 LLM 理解这个文本，然后生成代码并执行。

让我们把这个过程拆解开来：

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Agent 读取 Skill 文件                              │
│                                                              │
│  content = read('/skills/article-writer.md')                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: 将 Skill 内容发送给 LLM                            │
│                                                              │
│  prompt = f"""                                               │
│  你是一个 AI Agent。请理解以下 Skill 并执行：               │
│                                                              │
│  {content}                                                   │
│                                                              │
│  用户请求：{user_request}                                    │
│  """                                                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: LLM 返回执行计划                                    │
│                                                              │
│  "理解了。这个 Skill 的作用是写文章。                        │
│   执行步骤：                                                 │
│   1. 调用 DeepSeek API 生成大纲                             │
│   2. 根据大纲撰写正文                                       │
│   3. 格式化并保存文件"                                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Agent 根据 LLM 的理解生成代码                      │
│                                                              │
│  code = f"""                                                 │
│  import requests                                             │
│  response = requests.post(                                   │
│      'https://api.deepseek.com/v1/chat/completions',        │
│      json={...}                                              │
│  )                                                           │
│  """                                                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Agent 用 Bash 执行生成的代码                       │
│                                                              │
│  bash("python3 /tmp/generated_code.py")                     │
└─────────────────────────────────────────────────────────────┘
```

**关键洞察**：

- Skill 是**声明式的**（描述"做什么"），而不是**命令式的**（描述"怎么做"）
- LLM 是**编译器**，将 Skill 文本"编译"成可执行代码
- Agent 是**运行时**，负责执行代码并管理生命周期

### 1.2 如何编写高质量 Skill

一个高质量的 Skill 应该具备以下特征：

#### ✅ 结构清晰

**差的 Skill**：
```markdown
写文章。生成大纲，然后写内容，最后保存。
```

**好的 Skill**：
```markdown
# Article Writer Skill

## 功能
自动生成技术文章的完整 Agent。

## 输入
- `topic`：文章主题（字符串）
- `style`：文章风格（可选，默认"专业"）
- `length`：目标字数（可选，默认 1500）

## 输出
- 完整的 Markdown 格式文章

## 流程
1. 生成大纲
2. 撰写正文
3. 生成引言和结论
4. 格式化输出
5. 保存文件

详细步骤见下文。
```

#### ✅ Prompt 优化

在 Skill 中使用好的 Prompt 技巧：

**技巧 1：提供示例**

```markdown
## Prompt 模板

请为主题 "{topic}" 生成大纲。

示例：
输入：AI Agent 的未来
输出：
{
  "title": "AI Agent：从工具到伙伴",
  "sections": [
    {"title": "技术演进", "content": "从规则引擎到大模型"},
    {"title": "应用场景", "content": "从个人助理到企业自动化"}
  ]
}
```

**技巧 2：明确约束**

```markdown
## 约束条件
- 大纲必须包含 3-5 个段落
- 每个段落标题不超过 15 字
- 段落之间逻辑连贯
- 必须以 JSON 格式返回
```

**技巧 3：错误处理**

```markdown
## 错误处理
如果 API 调用失败：
1. 记录错误日志到 /logs/article-writer.log
2. 检查错误类型（网络错误、API 限流、格式错误）
3. 网络错误：重试 3 次，间隔 2 秒
4. API 限流：等待 60 秒后重试
5. 格式错误：使用默认大纲模板
```

#### ✅ 参数化设计

让 Skill 支持参数，提高复用性：

```markdown
# Email Writer Skill

## 功能
发送专业邮件。

## 参数
- `to`：收件人邮箱（必需）
- `subject`：邮件主题（必需）
- `tone`：语气（可选，默认"专业"，可选"友好"、"正式"）
- `template`：模板（可选，默认"standard"，可选"invitation"、"followup"）

## 参数使用示例
```
tone=professional, template=standard → 生成标准商务邮件
tone=friendly, template=followup → 生成友好的跟进邮件
```
```

#### ✅ 版本控制

在 Skill 中包含版本信息：

```markdown
# Article Writer Skill

## 版本
- Version: 2.1.0
- Last Updated: 2026-02-25
- Changelog:
  - v2.1.0: 添加搜索功能
  - v2.0.0: 重构为参数化设计
  - v1.0.0: 初始版本

## 兼容性
- 依赖 DeepSeek API v1.0+
- 需要 Node.js 18+
```

### 1.3 Skill 的生命周期管理

#### 动态加载

```javascript
import { watch } from 'chokidar';

const skillWatcher = watch(['skills/', 'agents/']);

skillWatcher.on('change', async (path) => {
  console.log(`🔄 Skill 变化: ${path}`);

  // 重新加载 Skill
  await agent.reloadSkill(path);

  // 通知所有连接的客户端
  broadcastToClients({
    type: 'skill_reloaded',
    path: path
  });
});
```

#### 版本管理

```javascript
const skillVersions = new Map();

async function loadSkill(path, version = 'latest') {
  const skill = await readSkillFile(path);

  // 记录版本
  if (!skillVersions.has(path)) {
    skillVersions.set(path, new Map());
  }

  const versionMap = skillVersions.get(path);

  // 如果是新版本，保存旧版本
  if (versionMap.has('current')) {
    versionMap.set(versionMap.get('current'), skill);
  }

  versionMap.set('current', version);
  versionMap.set(version, skill);

  return skill;
}

// 回滚到之前的版本
async function rollbackSkill(path, version) {
  const versionMap = skillVersions.get(path);
  if (versionMap && versionMap.has(version)) {
    const oldSkill = versionMap.get(version);
    await applySkill(path, oldSkill);
  }
}
```

#### 依赖管理

```markdown
# Article Writer Skill

## 依赖
- `skills/search.md`：用于搜索相关资料
- `skills/review.md`：用于校对文章
- `utils/markdown.js`：用于 Markdown 格式化

## 依赖版本
- search.md v1.2+
- review.md v2.0+
- markdown.js v3.1.0
```

### 1.4 高级技巧

#### Skill 组合

```javascript
// 复合 Skill：组合多个 Skill
const compositeSkill = `
# Content Production Pipeline

## 功能
完整的内容生产流程。

## 步骤
1. 使用 skills/researcher.md 搜索资料
2. 使用 skills/writer.md 撰写内容
3. 使用 skills/reviewer.md 校对
4. 使用 skills/publisher.md 发布
`;
```

#### 条件分支

```markdown
# Adaptive Writer Skill

## 功能
根据内容类型选择不同的写作策略。

## 条件判断
如果 `type == "technical"`：
  - 使用 skills/technical-writer.md
  - 语气：专业、严谨
  - 结构：问题 → 分析 → 解决方案

如果 `type == "marketing"`：
  - 使用 skills/marketing-writer.md
  - 语气：热情、有说服力
  - 结构：痛点 → 方案 → 证明 → 行动

如果 `type == "news"`：
  - 使用 skills/news-writer.md
  - 语气：客观、简洁
  - 结构：导语 → 细节 → 背景 → 影响
```

---

## 二、基于 Pi 的业务系统架构设计

现在我们理解了 Skill 机制，接下来看看如何设计一个完整的业务系统。

### 2.1 典型架构模式

一个基于 Pi 的业务系统通常包含以下层次：

```
┌─────────────────────────────────────────────────────────────┐
│                        前端层                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Web App    │  │  Mobile App  │  │  Admin Panel │      │
│  │   (React)    │  │   (React)    │  │   (React)    │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │               │
│         └─────────────────┴─────────────────┘               │
│                           │                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │ HTTP/WebSocket
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        后端层                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   API 层     │  │  任务队列     │  │  权限管理     │      │
│  │  (Express)   │  │  (BullMQ)    │  │  (Casbin)    │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │               │
│         └─────────────────┴─────────────────┘               │
│                           │                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      Pi Agent 层                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Session 管理 │  │  Skill 调用   │  │ LLM 统一接口  │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │               │
│         └─────────────────┴─────────────────┘               │
│                           │                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      外部服务层                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ DeepSeek API │  │   PostgreSQL │  │    Redis     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 核心设计决策

#### 前后端分离 vs 集成

| 方案 | 优势 | 劣势 | 适用场景 |
|------|------|------|---------|
| **前后端分离** | 开发效率高，易于扩展 | 部署复杂 | 大型系统、多端支持 |
| **集成部署** | 部署简单，性能好 | 开发耦合 | 小型系统、单一应用 |

**推荐**：前后端分离，使用 Docker Compose 编排部署。

#### Session 管理策略

**方案 A：有状态 Session**
- 每个 Session 对应一个 Agent 实例
- 优点：状态保持简单
- 缺点：资源占用高，难以扩展

**方案 B：无状态 Session**
- Session 状态序列化存储
- 优点：易扩展，资源利用率高
- 缺点：状态管理复杂

**推荐**：方案 B，将 Session 存储在 Redis 中。

#### 多租户隔离

```typescript
// 租户上下文
interface TenantContext {
  tenantId: string;
  userId: string;
  permissions: string[];
}

// 中间件：注入租户上下文
app.use((req, res, next) => {
  const token = req.headers.authorization;
  const context = decodeToken(token);

  req.tenant = context;
  next();
});

// Agent 执行时隔离
await agent.execute(skill, {
  input: params,
  tenant: req.tenant
});
```

---

## 三、完整实战案例：智能内容管理系统

现在让我们把所有知识整合起来，构建一个完整的智能内容管理系统。

### 3.1 系统需求

**功能需求**：

1. **内容生产**：
   - 支持多种内容类型（文章、报告、营销文案）
   - 自动生成大纲、撰写、校对
   - 支持自定义模板和风格

2. **任务管理**：
   - 创建、查看、取消任务
   - 实时进度展示
   - 任务历史记录

3. **内容审核**：
   - 人工审核流程
   - 审核意见反馈
   - 版本管理

4. **权限管理**：
   - 角色权限（管理员、编辑、作者）
   - 内容访问控制
   - 操作日志

**非功能需求**：

- 响应时间：任务创建 < 100ms，进度更新 < 500ms
- 并发：支持 100 个用户同时创建任务
- 可用性：99.9%
- 可扩展性：支持横向扩展

### 3.2 项目结构

```
content-management-system/
├── frontend/                 # 前端项目
│   ├── src/
│   │   ├── components/       # React 组件
│   │   ├── pages/            # 页面
│   │   ├── services/         # API 调用
│   │   └── types/            # 类型定义
│   ├── package.json
│   └── vite.config.ts
├── backend/                  # 后端项目
│   ├── src/
│   │   ├── routes/           # API 路由
│   │   ├── services/         # 业务逻辑
│   │   ├── agents/           # Pi Agent 集成
│   │   ├── skills/           # Skill 文件
│   │   ├── queue/            # 任务队列
│   │   └── db/               # 数据库
│   └── package.json
├── docker/                   # Docker 配置
│   ├── Dockerfile.frontend
│   ├── Dockerfile.backend
│   └── docker-compose.yml
└── README.md
```

### 3.3 数据库设计

```sql
-- 用户表
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(100),
  role VARCHAR(20) DEFAULT 'user',
  created_at TIMESTAMP DEFAULT NOW()
);

-- 任务表
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  type VARCHAR(50) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  params JSONB,
  result JSONB,
  error TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- 内容表
CREATE TABLE contents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id),
  title VARCHAR(500),
  content TEXT,
  status VARCHAR(20) DEFAULT 'draft',
  published_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 审核表
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  content_id UUID REFERENCES contents(id),
  reviewer_id UUID REFERENCES users(id),
  status VARCHAR(20),
  comments TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_tasks_user ON tasks(user_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_contents_task ON contents(task_id);
```

### 3.4 后端实现

#### API 路由

```typescript
// routes/tasks.ts
import express from 'express';
import { taskQueue } from '../queue';
import { db } from '../db';

const router = express.Router();

// 创建任务
router.post('/', async (req, res) => {
  const { type, params, userId } = req.body;

  // 创建任务记录
  const task = await db.tasks.create({
    type,
    params,
    userId,
    status: 'pending'
  });

  // 添加到队列
  await taskQueue.add('process-task', {
    taskId: task.id,
    type,
    params
  });

  res.json(task);
});

// 获取任务状态
router.get('/:id', async (req, res) => {
  const task = await db.tasks.findById(req.params.id);
  res.json(task);
});

// 获取用户任务列表
router.get('/', async (req, res) => {
  const { userId } = req.query;
  const tasks = await db.tasks.findByUser(userId as string);
  res.json(tasks);
});

export default router;
```

#### 任务处理器

```typescript
// services/taskProcessor.ts
import { agent } from '../agents';
import { loadSkill } from '../skills';

export async function processTask(
  type: string,
  params: Record<string, any>,
  callbacks: {
    onProgress?: (progress: number, step: string) => void;
  }
) {
  try {
    // 加载 Skill
    const skill = await loadSkill(type);
    callbacks.onProgress?.(10, '加载 Skill');

    // 创建 Session
    const session = agent.createSession();
    callbacks.onProgress?.(20, '创建 Session');

    // 准备参数
    const input = {
      ...params,
      deepseek_api_key: process.env.DEEPSEEK_API_KEY
    };

    // 执行
    const result = await session.execute(skill.content, {
      input,
      onProgress: (step, progress) => {
        const totalProgress = 20 + Math.floor(progress * 0.7);
        callbacks.onProgress?.(totalProgress, step);
      }
    });

    callbacks.onProgress?.(100, '完成');

    return result;
  } catch (error) {
    throw new Error(`Task processing failed: ${error.message}`);
  }
}
```

### 3.5 前端实现

#### 任务列表组件

```tsx
// components/TaskList.tsx
import { useEffect, useState } from 'react';
import { api } from '../services/api';

interface Task {
  id: string;
  type: string;
  status: string;
  createdAt: string;
}

export function TaskList() {
  const [tasks, setTasks] = useState<Task[]>([]);

  useEffect(() => {
    loadTasks();
  }, []);

  const loadTasks = async () => {
    const data = await api.tasks.list();
    setTasks(data);
  };

  return (
    <div className="task-list">
      <h2>任务列表</h2>
      <table>
        <thead>
          <tr>
            <th>类型</th>
            <th>状态</th>
            <th>创建时间</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          {tasks.map(task => (
            <tr key={task.id}>
              <td>{task.type}</td>
              <td>{task.status}</td>
              <td>{new Date(task.createdAt).toLocaleString()}</td>
              <td>
                <button>查看</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

### 3.6 部署配置

#### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: cms
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7
    volumes:
      - redis-data:/data

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://admin:password@postgres:5432/cms
      REDIS_HOST: redis
      REDIS_PORT: 6379
      DEEPSEEK_API_KEY: ${DEEPSEEK_API_KEY}
    depends_on:
      - postgres
      - redis
    ports:
      - "3000:3000"

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    environment:
      VITE_API_URL: http://backend:3000
    depends_on:
      - backend
    ports:
      - "5173:5173"

volumes:
  postgres-data:
  redis-data:
```

---

## 四、多 Agent 协同模式

在复杂系统中，单个 Agent 往往不够用。我们需要多个 Agent 协同工作。

### 4.1 主从模式（Master-Worker）

**适用场景**：任务可分解为独立的子任务

```typescript
// master-worker.ts
class MasterAgent {
  private workers: WorkerAgent[] = [];

  async execute(task: Task) {
    // 分解任务
    const subtasks = await this.decompose(task);

    // 分配给 Worker
    const results = await Promise.all(
      subtasks.map(subtask => this.assignWorker(subtask))
    );

    // 聚合结果
    return this.aggregate(results);
  }

  private async assignWorker(subtask: Subtask) {
    const worker = this.getAvailableWorker();
    return worker.execute(subtask);
  }
}
```

### 4.2 流水线模式（Pipeline）

**适用场景**：任务有明确的先后顺序

```typescript
// pipeline.ts
class PipelineAgent {
  private stages: Agent[] = [];

  constructor(stages: Agent[]) {
    this.stages = stages;
  }

  async execute(input: any) {
    let output = input;

    for (const stage of this.stages) {
      output = await stage.execute(output);
    }

    return output;
  }
}

// 使用示例
const pipeline = new PipelineAgent([
  new ResearcherAgent(),
  new WriterAgent(),
  new ReviewerAgent(),
  new PublisherAgent()
]);

const result = await pipeline.execute({ topic: 'AI Agent' });
```

### 4.3 议会模式（Multi-Agent Voting）

**适用场景**：需要多个角度评估

```typescript
// parliament.ts
class ParliamentAgent {
  private members: Agent[] = [];

  async execute(task: Task) {
    // 每个 Agent 生成方案
    const proposals = await Promise.all(
      this.members.map(member => member.execute(task))
    );

    // 互相评估
    const scores = await this.evaluate(proposals);

    // 选择最佳方案
    const best = proposals.reduce((best, current) =>
      scores[current.id] > scores[best.id] ? current : best
    );

    return best;
  }

  private async evaluate(proposals: Proposal[]) {
    // 让每个 Agent 评估其他人的方案
    const scores = {};

    for (const member of this.members) {
      const evaluation = await member.evaluate(proposals);
      Object.assign(scores, evaluation);
    }

    return scores;
  }
}
```

### 4.4 实战案例：内容生产流水线

```typescript
// content-pipeline.ts
import { PipelineAgent } from './pipeline';
import { ResearcherAgent } from './agents/researcher';
import { WriterAgent } from './agents/writer';
import { ReviewerAgent } from './agents/reviewer';
import { PublisherAgent } from './agents/publisher';

export class ContentProductionPipeline extends PipelineAgent {
  constructor() {
    super([
      new ResearcherAgent(),  // 研究员：搜索资料
      new WriterAgent(),      // 写作：撰写内容
      new ReviewerAgent(),    // 审核：校对修改
      new PublisherAgent()    // 发布：推送到平台
    ]);
  }

  async execute(input: { topic: string; platform: string }) {
    console.log(`🚀 开始内容生产: ${input.topic}`);

    // 研究员搜索资料
    const research = await this.stages[0].execute(input);
    console.log('📚 研究完成');

    // 写作根据研究撰写内容
    const draft = await this.stages[1].execute({
      ...input,
      research
    });
    console.log('✍️  初稿完成');

    // 审核校对
    const reviewed = await this.stages[2].execute({
      ...input,
      draft
    });
    console.log('👀 审核完成');

    // 发布到平台
    const published = await this.stages[3].execute({
      ...input,
      content: reviewed
    });
    console.log('🚀 发布完成');

    return published;
  }
}

// 使用
const pipeline = new ContentProductionPipeline();
const result = await pipeline.execute({
  topic: 'AI Agent 的未来',
  platform: 'weixin'
});
```

---

## 五、性能优化与最佳实践

### 5.1 减少 Token 消耗

**Prompt 压缩**

```markdown
# 差的 Prompt（冗长）
```
请你帮我写一篇关于人工智能的文章。这篇文章应该包含引言、正文和结论。引言部分应该介绍背景，正文部分应该分析技术，结论部分应该总结观点。文章大约 1500 字，风格要专业。
```

# 好的 Prompt（简洁）
```
写一篇 1500 字的专业技术文章，主题：人工智能。
结构：引言(背景) + 正文(技术分析) + 结论(总结)。
```
```

**缓存策略**

```typescript
const cache = new Map();

async function callLLM(prompt: string) {
  const cacheKey = hash(prompt);

  if (cache.has(cacheKey)) {
    return cache.get(cacheKey);
  }

  const result = await deepseekAPI(prompt);
  cache.set(cacheKey, result);

  // 设置过期时间
  setTimeout(() => cache.delete(cacheKey), 1000 * 60 * 60);

  return result;
}
```

### 5.2 提升响应速度

**并行请求**

```typescript
// 串行（慢）
const outline = await generateOutline(topic);
const title = await generateTitle(topic);
const intro = await generateIntroduction(topic);

// 并行（快）
const [outline, title, intro] = await Promise.all([
  generateOutline(topic),
  generateTitle(topic),
  generateIntroduction(topic)
]);
```

**流式输出**

```typescript
async function streamGenerate(prompt: string) {
  const response = await fetch('https://api.deepseek.com/v1/chat/completions', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      model: 'deepseek-chat',
      messages: [{ role: 'user', content: prompt }],
      stream: true  // 启用流式输出
    })
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = JSON.parse(line.slice(6));
        const content = data.choices[0].delta.content;

        // 实时输出
        process.stdout.write(content);
      }
    }
  }
}
```

### 5.3 稳定性保障

**重试机制**

```typescript
async function retry<T>(
  fn: () => Promise<T>,
  options: { maxRetries?: number; delay?: number } = {}
): Promise<T> {
  const { maxRetries = 3, delay = 2000 } = options;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      console.log(`重试 ${i + 1}/${maxRetries}...`);
      await sleep(delay * (i + 1));  // 指数退避
    }
  }

  throw new Error('Max retries exceeded');
}

// 使用
const result = await retry(() => callDeepSeek(prompt), {
  maxRetries: 3,
  delay: 2000
});
```

**降级策略**

```typescript
async function robustAgent(input: string) {
  try {
    // 尝试使用 DeepSeek
    return await callDeepSeek(input);
  } catch (error) {
    console.warn('DeepSeek 失败，降级到本地模型');

    // 降级到本地模型
    return await callLocalModel(input);
  }
}
```

**监控告警**

```typescript
import { Prometheus } from '@prometheus/client';

const requestDuration = new Prometheus.Histogram({
  name: 'agent_request_duration_seconds',
  help: 'Agent request duration'
});

const requestErrors = new Prometheus.Counter({
  name: 'agent_request_errors_total',
  help: 'Total agent request errors'
});

async function monitoredAgent(input: string) {
  const end = requestDuration.startTimer();

  try {
    const result = await agent.execute(input);
    end();
    return result;
  } catch (error) {
    requestErrors.inc();
    end();
    throw error;
  }
}
```

---

## 六、总结与展望

通过这篇文章，我们深入探讨了：

### 核心要点

1. **Skill 机制深度解析**
   - Skill 的本质是"声明式文本 + LLM 编译器"
   - 编写高质量 Skill 的技巧（结构、Prompt、参数化、版本控制）
   - Skill 生命周期管理（动态加载、版本管理、依赖管理）

2. **业务系统架构设计**
   - 典型的四层架构（前端、后端、Pi Agent、外部服务）
   - 核心设计决策（前后端分离、Session 管理、多租户隔离）

3. **完整实战案例**
   - 智能内容管理系统的全流程实现
   - 数据库设计、后端 API、前端组件、部署配置

4. **多 Agent 协同**
   - 主从模式、流水线模式、议会模式
   - 内容生产流水线的实际应用

5. **性能优化**
   - 减少 Token 消耗、提升响应速度、稳定性保障

### Pi 的适用场景

**非常适合**：
- 内容生产（文章、报告、文案）
- 代码生成（项目脚手架、CRUD、测试）
- 数据处理（分析、转换、可视化）
- 自动化运维（部署、监控、故障处理）

**不太适合**：
- 需要复杂逻辑运算的场景
- 实时性要求极高的场景
- 需要精确数值计算的场景

### 未来发展方向

1. **更好的 Prompt 管理**：Prompt 版本控制、A/B 测试、自动优化
2. **更强的多 Agent 协同**：更复杂的协作模式、自动分工
3. **更完善的工具生态**：预装常用工具、插件市场
4. **更低的延迟**：模型蒸馏、量化、边缘部署

---

## 参考资源

- **Pi 官方仓库**：https://github.com/badlogic/pi-mono
- **DeepSeek API 文档**：https://platform.deepseek.com/docs
- **系列文章**：
  - 第一篇：《OpenClaw 火爆背后：揭秘 Pi Agent 框架的极简哲学》
  - 第二篇：《Pi Agent 实战：手把手教你构建第一个 AI Agent》
- **推荐阅读**：
  - 《Prompt Engineering Guide》
  - 《设计模式：可复用面向对象软件的基础》

---

**作者简介**：辉哥是一名技术类公众号博主，专注于 AI、Agent、开源技术等领域。欢迎关注我的公众号获取更多技术文章。

> "架构不是设计出来的，而是演化出来的。" —— 这句话在 AI Agent 系统中同样适用。
