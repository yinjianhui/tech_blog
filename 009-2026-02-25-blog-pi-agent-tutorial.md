# Pi Agent 实战：手把手教你构建第一个 AI Agent

## 引言

在上一篇文章《OpenClaw 火爆背后：揭秘 Pi Agent 框架的极简哲学》中，我们介绍了 Pi Agent 的设计哲学和核心理念。你可能已经被它的极简主义所吸引——只有 4 个工具（Read、Write、Edit、Bash），却能实现无限的可能。

但是，**光说不练假把式**。在这篇文章中，我将带你从零开始，亲自动手构建一个基于 Pi 的 AI Agent。

我们要实现的 Agent 功能是：**文档自动写作 Agent**。你只需要给它一个主题，它就能自动搜索资料、生成大纲、撰写正文、格式化输出，最终生成一篇完整的技术文章。

---

## 一、环境准备

### 1.1 安装 Node.js

Pi 是用 TypeScript 编写的，所以需要先安装 Node.js。

**macOS/Linux**：
```bash
# 使用 nvm（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22
```

**Windows**：
从 [nodejs.org](https://nodejs.org/) 下载并安装 LTS 版本。

验证安装：
```bash
node -v  # 应该显示 v22.x.x
npm -v
```

### 1.2 安装 Pi CLI

Pi 提供了命令行工具，方便快速创建和运行 Agent。

```bash
# 全局安装 Pi Coding Agent
npm install -g @mariozechner/pi-coding-agent

# 验证安装
pi --version
```

### 1.3 配置 DeepSeek API

Pi 支持多种 LLM 提供商，本教程使用 **DeepSeek**（性价比高，中文支持好）。

**获取 API Key**：
1. 访问 [DeepSeek 开放平台](https://platform.deepseek.com/)
2. 注册账号
3. 在"API Keys"页面创建新的 API Key

**配置环境变量**：

**macOS/Linux**：
```bash
# 编辑 ~/.bashrc 或 ~/.zshrc
export DEEPSEEK_API_KEY="your-api-key-here"

# 重新加载配置
source ~/.bashrc
```

**Windows**：
```powershell
# 在系统环境变量中添加
# 变量名：DEEPSEEK_API_KEY
# 变量值：your-api-key-here
```

验证配置：
```bash
echo $DEEPSEEK_API_KEY  # 应该显示你的 API Key
```

### 1.4 创建项目目录

```bash
mkdir pi-agent-tutorial
cd pi-agent-tutorial
npm init -y  # 初始化 npm 项目
```

---

## 二、创建第一个 Agent

### 2.1 理解 Pi 的 4 个工具

在开始之前，我们先回顾一下 Pi 的 4 个核心工具：

| 工具 | 功能 | 示例 |
|------|------|------|
| **Read** | 读取文件内容 | `read('/tmp/file.txt')` |
| **Write** | 写入文件 | `write('/tmp/output.txt', 'Hello World')` |
| **Edit** | 编辑文件（替换内容） | `edit('/tmp/file.txt', oldText, newText)` |
| **Bash** | 执行 shell 命令 | `bash('ls -la')` |

这 4 个工具看似简单，但组合起来可以实现任何功能。

### 2.2 第一个简单示例

让我们创建一个最简单的 Agent：读取一个文件并回答问题。

**创建文件 `question-answer.md`**：
```markdown
# Question Answer Skill

## 功能
读取文件并回答用户的问题。

## 使用方法
1. 告诉 Agent 文件路径
2. 告诉 Agent 你的问题
3. Agent 会读取文件并用 LLM 回答
```

**创建测试文件 `test.txt`**：
```
Pi Agent 是一个极简的 AI 框架，只有 4 个工具：Read、Write、Edit、Bash。
它由 Mario Zechner 创建，他是 libGDX 游戏框架的作者。
```

**运行 Agent**：
```bash
pi "读取 test.txt 文件，然后回答：Pi 有哪 4 个工具？"
```

**预期输出**：
```
Pi 有 4 个工具：
1. Read - 读取文件
2. Write - 写入文件
3. Edit - 编辑文件
4. Bash - 执行 shell 命令
```

恭喜！你已经成功运行了第一个 Pi Agent 🎉

---

## 三、实战：文档自动写作 Agent

现在让我们进入正题，构建一个完整的文档自动写作 Agent。

### 3.1 需求分析

我们的 Agent 需要实现以下功能：

1. **输入**：文章主题（例如："AI Agent 的未来"）
2. **输出**：完整的 Markdown 格式文章
3. **流程**：
   - 生成文章大纲
   - 根据大纲撰写正文
   - 格式化为标准 Markdown
   - 保存到文件

### 3.2 创建 Skill 文件

首先，我们创建一个 Skill 文件，告诉 Agent 如何工作。

**创建 `skills/article-writer.md`**：
```markdown
# Article Writer Skill

## 功能
自动生成技术文章的完整 Agent。

## 输入
- `topic`：文章主题（字符串）

## 输出
- 完整的 Markdown 格式文章，包含：
  - 标题（吸引人）
  - 引言（约 100 字）
  - 正文（1000-1500 字，分为 3-5 段）
  - 结论（约 100 字）

## 流程

### 步骤 1：生成大纲
使用 DeepSeek API 生成文章大纲。

Prompt 模板：
```
请为主题 "{topic}" 生成一份文章大纲。
要求：
1. 大纲包含 3-5 个主要段落
2. 每个段落有明确的主题
3. 逻辑清晰，层层递进
```

### 步骤 2：撰写正文
根据大纲逐段撰写正文。

Prompt 模板：
```
根据以下大纲撰写文章正文：

大纲：
{outline}

要求：
1. 每段 200-300 字
2. 逻辑连贯，过渡自然
3. 使用通俗易懂的语言
4. 加入实际案例或数据
```

### 步骤 3：生成引言和结论
Prompt 模板：
```
根据文章正文，生成：
1. 引言（100 字）：概述主题，引出下文
2. 结论（100 字）：总结观点，提出展望
```

### 步骤 4：格式化输出
将所有部分组合成标准 Markdown 格式：
```markdown
# {标题}

## 引言
{引言内容}

## 正文
### {段落1 标题}
{段落1 内容}

### {段落2 标题}
{段落2 内容}

...

## 结论
{结论内容}
```

### 步骤 5：保存文件
将文章保存到 `output/{timestamp}-{topic-slug}.md`

## API 配置
- Provider: DeepSeek
- Model: deepseek-chat
- API Key: 从环境变量 DEEPSEEK_API_KEY 读取

## 错误处理
如果任何步骤失败，Agent 应该：
1. 记录错误日志
2. 回滚已生成的部分内容
3. 向用户报告错误信息
```

### 3.3 实现 Agent 代码

现在我们创建 Agent 的实现代码。

**创建 `agent.js`**：
```javascript
import fs from 'fs';
import path from 'path';
import { fileURLToPath } from 'url';
import { spawn } from 'child_process';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// 配置
const CONFIG = {
  apiKey: process.env.DEEPSEEK_API_KEY,
  model: 'deepseek-chat',
  apiBaseUrl: 'https://api.deepseek.com/v1',
  outputDir: path.join(__dirname, 'output')
};

// 确保 output 目录存在
if (!fs.existsSync(CONFIG.outputDir)) {
  fs.mkdirSync(CONFIG.outputDir, { recursive: true });
}

/**
 * 调用 DeepSeek API
 */
async function callDeepSeek(prompt, options = {}) {
  const response = await fetch(`${CONFIG.apiBaseUrl}/chat/completions`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${CONFIG.apiKey}`
    },
    body: JSON.stringify({
      model: CONFIG.model,
      messages: [
        { role: 'system', content: '你是一个专业的内容创作者，擅长撰写技术文章。' },
        { role: 'user', content: prompt }
      ],
      temperature: 0.7,
      max_tokens: options.maxTokens || 2000
    })
  });

  if (!response.ok) {
    throw new Error(`DeepSeek API 错误: ${response.status} ${response.statusText}`);
  }

  const data = await response.json();
  return data.choices[0].message.content;
}

/**
 * 生成文章大纲
 */
async function generateOutline(topic) {
  console.log(`📝 正在为主题 "${topic}" 生成大纲...`);

  const prompt = `请为主题 "${topic}" 生成一份文章大纲。

要求：
1. 大纲包含 3-5 个主要段落
2. 每个段落有明确的主题
3. 逻辑清晰，层层递进

请以 JSON 格式返回：
{
  "title": "文章标题",
  "sections": [
    {"title": "段落1标题", "content": "段落1主题描述"},
    {"title": "段落2标题", "content": "段落2主题描述"}
  ]
}`;

  const result = await callDeepSeek(prompt, { maxTokens: 1000 });

  // 尝试提取 JSON
  const jsonMatch = result.match(/\{[\s\S]*\}/);
  if (jsonMatch) {
    return JSON.parse(jsonMatch[0]);
  }

  throw new Error('无法解析大纲 JSON');
}

/**
 * 撰写段落内容
 */
async function writeSection(sectionTitle, sectionDescription, previousContent = '') {
  console.log(`✍️  正在撰写段落: ${sectionTitle}...`);

  const prompt = `请撰写文章段落。

段落标题：${sectionTitle}
段落主题：${sectionDescription}

${previousContent ? `前文内容：\n${previousContent}\n\n` : ''}

要求：
1. 200-300 字
2. 逻辑连贯
3. 使用通俗易懂的语言
4. 加入实际案例或数据`;

  return await callDeepSeek(prompt, { maxTokens: 800 });
}

/**
 * 生成引言
 */
async function generateIntroduction(title, outline) {
  console.log('📖 正在生成引言...');

  const outlineText = outline.sections.map(s => `- ${s.title}`).join('\n');

  const prompt = `根据文章信息生成引言。

文章标题：${title}

文章大纲：
${outlineText}

要求：
1. 约 100 字
2. 概述主题，引出下文
3. 吸引读者继续阅读`;

  return await callDeepSeek(prompt, { maxTokens: 300 });
}

/**
 * 生成结论
 */
async function generateConclusion(title, bodyContent) {
  console.log('💡 正在生成结论...');

  const prompt = `根据文章内容生成结论。

文章标题：${title}

文章正文摘要：
${bodyContent.slice(0, 500)}...

要求：
1. 约 100 字
2. 总结观点
3. 提出展望或启发`;

  return await callDeepSeek(prompt, { maxTokens: 300 });
}

/**
 * 生成标题
 */
async function generateTitle(topic) {
  console.log('🎯 正在生成标题...');

  const prompt = `为主题 "${topic}" 生成一个吸引人的文章标题。

要求：
1. 简洁有力（不超过 20 字）
2. 能引起读者兴趣
3. 准确表达文章主题

只返回标题，不要其他内容。`;

  return await callDeepSeek(prompt, { maxTokens: 100 });
}

/**
 * 保存文章
 */
function saveArticle(article, topic) {
  const timestamp = new Date().toISOString().replace(/[:.]/g, '-').slice(0, -5);
  const slug = topic.toLowerCase().replace(/\s+/g, '-').replace(/[^\w-]/g, '');
  const filename = `${timestamp}-${slug}.md`;
  const filepath = path.join(CONFIG.outputDir, filename);

  fs.writeFileSync(filepath, article, 'utf-8');
  console.log(`✅ 文章已保存到: ${filepath}`);

  return filepath;
}

/**
 * 主函数：生成完整文章
 */
async function generateArticle(topic) {
  try {
    console.log(`\n🚀 开始生成文章: ${topic}\n`);

    // 1. 生成大纲
    const outline = await generateOutline(topic);

    // 2. 生成标题
    const title = await generateTitle(topic);

    // 3. 撰写正文段落
    const sections = [];
    let previousContent = '';

    for (const section of outline.sections) {
      const content = await writeSection(section.title, section.content, previousContent);
      sections.push({
        title: section.title,
        content: content
      });
      previousContent += `\n\n## ${section.title}\n\n${content}`;
    }

    // 4. 生成引言
    const introduction = await generateIntroduction(title, outline);

    // 5. 生成结论
    const bodyText = sections.map(s => s.content).join('\n\n');
    const conclusion = await generateConclusion(title, bodyText);

    // 6. 组装文章
    const article = `# ${title}

## 引言
${introduction}

## 正文
${sections.map(s => `### ${s.title}\n\n${s.content}`).join('\n\n')}

## 结论
${conclusion}
`;

    // 7. 保存文章
    const filepath = saveArticle(article, topic);

    console.log(`\n🎉 文章生成完成！\n`);
    console.log(`标题: ${title}`);
    console.log(`字数: ${article.length} 字`);
    console.log(`文件: ${filepath}\n`);

    return { title, article, filepath };
  } catch (error) {
    console.error('❌ 生成文章时出错:', error.message);
    throw error;
  }
}

// 导出供 Pi 使用
export default async function(topic) {
  if (!topic) {
    throw new Error('请提供文章主题');
  }

  return await generateArticle(topic);
}

// 如果直接运行此脚本
if (import.meta.url === `file://${process.argv[1]}`) {
  const topic = process.argv[2] || 'AI Agent 的未来';

  generateArticle(topic)
    .then(() => console.log('✅ 完成'))
    .catch(err => console.error('❌ 失败:', err));
}
```

### 3.4 运行测试

现在让我们测试一下这个 Agent。

**运行 Agent**：
```bash
node agent.js "Pi Agent 的极简哲学"
```

**预期输出**：
```
🚀 开始生成文章: Pi Agent 的极简哲学

🎯 正在生成标题...
📝 正在为主题 "Pi Agent 的极简哲学" 生成大纲...
✍️  正在撰写段落: 什么是极简主义...
✍️  正在撰写段落: 4个工具的威力...
✍️  正在撰写段落: 自我进化的Agent...
📖 正在生成引言...
💡 正在生成结论...
✅ 文章已保存到: /path/to/output/2026-02-25-Pi-Agent.md

🎉 文章生成完成！

标题: Pi Agent：用极简重新定义 AI 框架
字数: 1234 字
文件: /path/to/output/2026-02-25-Pi-Agent.md

✅ 完成
```

**查看生成的文章**：
```bash
cat output/2026-02-25-Pi-Agent.md
```

---

## 四、进阶：让 Agent 更智能

基础版本已经能工作了，现在让我们添加一些高级功能。

### 4.1 添加搜索功能

在生成文章之前，先搜索相关资料，让内容更有深度。

**创建 `utils/search.js`**：
```javascript
/**
 * 使用 Google Custom Search API 搜索资料
 */
async function searchResources(query, apiKey, cx) {
  const url = `https://www.googleapis.com/customsearch/v1?key=${apiKey}&cx=${cx}&q=${encodeURIComponent(query)}`;

  const response = await fetch(url);
  const data = await response.json();

  return data.items?.slice(0, 5).map(item => ({
    title: item.title,
    snippet: item.snippet,
    link: item.link
  })) || [];
}

export { searchResources };
```

**修改 Agent 代码，在生成大纲前先搜索**：
```javascript
import { searchResources } from './utils/search.js';

async function generateArticle(topic) {
  // ...

  // 搜索相关资料
  console.log('🔍 正在搜索相关资料...');
  const resources = await searchResources(
    topic,
    process.env.GOOGLE_API_KEY,
    process.env.GOOGLE_CX
  );

  // 将搜索结果传递给 LLM
  const outline = await generateOutline(topic, resources);

  // ...
}
```

### 4.2 添加自我校对

让 Agent 在生成文章后，自己检查并改进。

**创建 `utils/review.js`**：
```javascript
/**
 * 文章校对
 */
async function reviewArticle(article) {
  const prompt = `请检查以下文章的问题：

${article}

请检查：
1. 错别字和语法错误
2. 逻辑不连贯的地方
3. 可以改进的表达

返回修改建议。`;

  const suggestions = await callDeepSeek(prompt);

  // 根据建议修改文章
  const revisedPrompt = `根据以下建议修改文章：

建议：
${suggestions}

原文：
${article}

返回修改后的完整文章。`;

  return await callDeepSeek(revisedPrompt, { maxTokens: 4000 });
}

export { reviewArticle };
```

### 4.3 多 Agent 协同

我们可以创建多个专门的 Agent，各司其职：

**创建 `agents/researcher.js`**：
```javascript
// 研究员 Agent：负责搜索和整理资料
export async function researcher(topic) {
  console.log('🔬 研究员 Agent 开始工作...');

  // 搜索资料
  // 整理信息
  // 返回研究报告
}
```

**创建 `agents/writer.js`**：
```javascript
// 写作 Agent：负责撰写文章
export async function writer(research) {
  console.log('✍️  写作 Agent 开始工作...');

  // 根据研究报告写作
  // 返回文章草稿
}
```

**创建 `agents/editor.js`**：
```javascript
// 编辑 Agent：负责校对和修改
export async function editor(draft) {
  console.log('📝 编辑 Agent 开始工作...');

  // 校对文章
  // 返回最终版本
}
```

**协调多个 Agent**：
```javascript
import { researcher } from './agents/researcher.js';
import { writer } from './agents/writer.js';
import { editor } from './agents/editor.js';

async function multiAgentWorkflow(topic) {
  // 1. 研究员搜索资料
  const research = await researcher(topic);

  // 2. 写作撰写文章
  const draft = await writer(research);

  // 3. 编辑校对修改
  const final = await editor(draft);

  return final;
}
```

---

## 五、调试与优化

### 5.1 查看 Agent 的思考过程

Pi 支持输出 Agent 的详细思考日志。

**启用调试模式**：
```bash
pi --debug "写一篇关于 AI 的文章"
```

**在代码中添加日志**：
```javascript
console.log('📊 Agent 状态:', JSON.stringify(state, null, 2));
console.log('🧠 Agent 思考:', thoughtProcess);
```

### 5.2 使用 Session 树状结构

Pi 支持 Session 的分支和回退。

**创建分支**：
```javascript
const branch1 = await agent.createBranch();
const branch2 = await agent.createBranch();

// 在 branch1 上尝试一种方案
await branch1.run('方案 A');

// 在 branch2 上尝试另一种方案
await branch2.run('方案 B');

// 比较结果，选择更好的
```

**回退到之前的状态**：
```javascript
await agent.resetToCheckpoint('outline-generated');
```

### 5.3 热重载

开发时修改代码后，不需要重启 Agent。

**启用热重载**：
```javascript
import { watch } from 'chokidar';

watch(['skills/', 'agents/])
  .on('change', (path) => {
    console.log(`🔄 文件变化: ${path}`);
    agent.reloadSkill(path);
  });
```

### 5.4 常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| API 调用失败 | API Key 错误 | 检查环境变量 `DEEPSEEK_API_KEY` |
| 生成内容质量差 | Prompt 不够清晰 | 优化 Prompt，增加示例 |
| 内存占用过高 | 没有清理旧的 Session | 定期清理不需要的 Session |
| 速度太慢 | API 调用串行 | 使用并行请求 |

---

## 六、总结与展望

恭喜你！通过这篇文章，你已经：

✅ 了解了 Pi Agent 的 4 个核心工具
✅ 创建了第一个简单的 Agent
✅ 构建了一个完整的文档自动写作 Agent
✅ 学会了如何添加高级功能
✅ 掌握了调试和优化技巧

### Pi 的更多可能性

Pi 的极简设计让它可以应用于各种场景：

- **代码生成**：自动生成项目代码
- **数据分析**：处理 CSV、Excel 文件
- **自动化测试**：编写和运行测试用例
- **DevOps**：部署和运维自动化

### 下一步学习

1. **深入研究 Pi 的源码**：https://github.com/badlogic/pi-mono
2. **学习 Prompt Engineering**：写好 Prompt 是 Agent 成功的关键
3. **加入社区**：Pi Discord 社区有很多实战经验分享
4. **构建自己的 Agent 系统**：基于 Pi 开发符合你业务需求的 Agent

---

## 参考资源

- **Pi 官方仓库**：https://github.com/badlogic/pi-mono
- **DeepSeek API 文档**：https://platform.deepseek.com/docs
- **上一篇文章**：《OpenClaw 火爆背后：揭秘 Pi Agent 框架的极简哲学》
- **Pi Discord 社区**：https://discord.gg/3cU7Bz4UPx

---

**作者简介**：辉哥是一名技术类公众号博主，专注于 AI、Agent、开源技术等领域。欢迎关注我的公众号获取更多技术文章。

> "实践是检验真理的唯一标准。" —— 这句话在 AI Agent 领域同样适用。
