# 从零制作一款 AI 应用：完整动手教程

**版本：v1.3 | 日期：2026-05-02**
**适合对象：想亲手做出第一款 AI 应用的新手开发者 / 产品人**

> 本文档由 **UCL**（社交名流 UCL）原创整理，转载请注明出处。
> 联系作者：1030504893@qq.com

---

## 目录

**第一部分：开始之前**

1. [教程定位：你会做出什么？](#一教程定位你会做出什么)
2. [模型选型指南：先选好工具](#二模型选型指南先选好工具)

**第二部分：产品定义 & 项目搭建**

3. [准备工作：环境 & 工具](#三准备工作环境--工具)
4. [第一阶段：想清楚再动手（产品定义）](#四第一阶段想清楚再动手产品定义)
5. [第二阶段：搭脚手架（项目初始化）](#五第二阶段搭脚手架项目初始化)

**第三部分：AI 核心功能开发**

6. [System Prompt 设计指南](#六system-prompt-设计指南)
7. [第三阶段：接通 AI 大脑（API 集成）](#七第三阶段接通-ai-大脑api-集成)
8. [第四阶段：做出界面（前端 UI）](#八第四阶段做出界面前端-ui)
9. [第五阶段：让它更聪明（Prompt 工程）](#九第五阶段让它更聪明prompt-工程)
10. [多轮对话与上下文管理](#十多轮对话与上下文管理)
11. [API 成本控制与 Token 管理](#十一api-成本控制与-token-管理)

**第四部分：质量 & 扩展**

12. [第六阶段：处理真实问题（错误 & 边界）](#十二第六阶段处理真实问题错误--边界)
13. [用户认证与多用户隔离](#十三用户认证与多用户隔离)
14. [数据持久化：历史记录存储](#十四数据持久化历史记录存储)
15. [AI 应用测试策略](#十五ai-应用测试策略)

**第五部分：上线 & 迭代**

16. [第七阶段：部署上线](#十六第七阶段部署上线)
17. [第八阶段：收集反馈 & 迭代](#十七第八阶段收集反馈--迭代)

**第六部分：参考 & 替代方案**

18. [完整项目结构参考](#十八完整项目结构参考)
19. [常见卡点 & 解决方案](#十九常见卡点--解决方案)
20. [Python 技术栈实现方案](#二十python-技术栈实现方案)
21. [番外篇：用百度秒哒零代码制作应用](#二十一番外篇用百度秒哒零代码制作应用) ← **不会写代码？从这里开始**

---

## 一、教程定位：你会做出什么？

本教程将带你从零构建一款 **AI 会议纪要助手**，它能够：

- 接收用户粘贴的会议文字记录
- 自动生成结构化摘要（决策 + 行动项 + 待讨论事项）
- 支持用户在线编辑、一键复制

**为什么选这个场景？**
- 功能边界清晰，适合第一个项目
- 覆盖 AI 应用核心技术点（Prompt 设计 / 流式输出 / 错误处理）
- 可以真实解决你自己的问题

**技术栈选择（可替换）：**

```
前端：HTML + CSS + JavaScript（无框架，最小依赖）
后端：Node.js + Express（或 Python + FastAPI）
AI：Claude API（可替换为其他 LLM API）
部署：Vercel（免费，一键部署）
```

> 如果你熟悉 React / Vue，跟着同样的逻辑用你熟悉的框架做即可。
> 核心是 AI 集成逻辑，不是前端框架。

---

## 二、模型选型指南：先选好工具

> Claude API 只是众多 LLM 之一。当你要做 AI 应用时，选哪个模型往往是第一个问题。
> 本章给出实用的选型框架，不做无意义的"谁更强"争论。

### 21.1 主流 LLM API 概览

| 模型 | 提供商 | 中文能力 | 速度 | 价格 | 适用场景 |
|-----|-------|---------|-----|-----|---------|
| claude-opus-4-6 | Anthropic | 优秀 | 慢 | $$$ | 复杂推理、代码审查、长文分析 |
| claude-sonnet-4-6 | Anthropic | 优秀 | 中 | $$ | 大多数生产场景的最佳平衡点 |
| claude-haiku-4-5 | Anthropic | 良好 | 极快 | $ | 格式化输出、高频简单任务 |
| GPT-4o | OpenAI | 优秀 | 中 | $$ | 多模态（图文混合）场景 |
| GPT-4o-mini | OpenAI | 良好 | 快 | $ | 简单任务，成本敏感场景 |
| 文心4.5 | 百度 | 最强 | 中 | $$ | 中文专属场景、国内合规需求 |
| DeepSeek-V3 | DeepSeek | 优秀 | 快 | 极低 $ | 成本极敏感场景 |
| Qwen-Max | 阿里 | 优秀 | 中 | $ | 国内场景、阿里云生态 |

### 21.2 选型决策树

```
Q1：是否必须在中国大陆合规运营？
  是 → 文心 / Qwen / DeepSeek（国内服务）
  否 → 继续 Q2

Q2：任务类型？
  格式化/分类/提取 → claude-haiku / GPT-4o-mini（快且便宜）
  推理/分析/创作  → claude-sonnet-4-6（首选）
  需要图片理解    → GPT-4o / Claude claude-sonnet-4-6（支持多模态）
  极度成本敏感    → DeepSeek-V3 / Qwen（价格最低）

Q3：是否有企业安全/隐私要求？
  是 → 考虑私有部署（Ollama + 开源模型）或签署企业协议
  否 → 用 API 即可
```

### 21.3 同一个应用使用多个模型

生产场景中，可以根据任务复杂度动态选模型：

```javascript
// model-selector.js
function selectModelForTask(task) {
  // 简单格式化任务用便宜模型
  if (task.type === 'format' && task.inputLength < 2000) {
    return 'claude-haiku-4-5-20251001';
  }
  // 中等复杂度用默认模型
  if (task.type === 'analysis') {
    return 'claude-sonnet-4-6';
  }
  // 高精度需求用最强模型
  if (task.type === 'critical' || task.requireHighAccuracy) {
    return 'claude-opus-4-6';
  }
  return 'claude-sonnet-4-6';  // 默认
}
```

### 21.4 本地部署方案（隐私敏感场景）

如果不能把数据发送到第三方 API（金融、医疗、内部系统），可以用 Ollama 在本地运行开源模型：

```bash
# 安装 Ollama（macOS）
brew install ollama

# 下载并运行 Qwen2.5 7B 模型（支持中文，约 4GB）
ollama pull qwen2.5:7b
ollama run qwen2.5:7b
```

```javascript
// Ollama 兼容 OpenAI API 格式，切换成本极低
const { OpenAI } = require('openai');

const client = new OpenAI({
  baseURL: 'http://localhost:11434/v1',  // Ollama 本地地址
  apiKey: 'ollama',  // 占位符，本地不需要真实 key
});

const response = await client.chat.completions.create({
  model: 'qwen2.5:7b',
  messages: [{ role: 'user', content: '帮我整理这段会议记录...' }],
  stream: true,
});
```

> 本地模型能力弱于云端顶尖模型，但对于格式化任务（纪要、摘要），7B/14B 参数的模型已经够用。

---

## 三、准备工作：环境 & 工具

### 2.1 必装工具

```bash
# 检查 Node.js 是否已安装（需要 v18+）
node --version

# 如果没有，去 https://nodejs.org 下载 LTS 版本

# 初始化包管理器
npm --version
```

### 2.2 获取 API Key

以 Claude API 为例：
1. 前往 [console.anthropic.com](https://console.anthropic.com)
2. 注册账号 → 创建 API Key
3. **重要**：Key 只会显示一次，立即保存到安全的地方

> API Key 类似于账号密码，**绝对不要提交到 Git 仓库**！

### 2.3 创建项目目录

```bash
mkdir ai-meeting-helper
cd ai-meeting-helper
npm init -y
```

### 2.4 安装依赖

```bash
# 后端 Web 框架
npm install express

# Anthropic SDK（Claude API 官方库）
npm install @anthropic-ai/sdk

# 环境变量管理
npm install dotenv

# 开发热重载（开发体验提升）
npm install --save-dev nodemon
```

### 2.5 配置环境变量（关键安全步骤）

创建 `.env` 文件（**不要提交到 Git**）：

```bash
# .env
ANTHROPIC_API_KEY=your_api_key_here
PORT=3000
```

创建 `.gitignore`（防止 Key 泄漏）：

```bash
# .gitignore
.env
node_modules/
```

---

## 四、第一阶段：想清楚再动手（产品定义）

> 很多人做 AI 应用最大的坑是"边做边想"，导致做到一半推倒重来。
> 花 30 分钟想清楚以下问题，能省你 3 天返工时间。

### 3.1 填写产品一页纸

在正式写代码前，回答这 5 个问题：

```
【用户是谁】
目标用户：参加多方协作会议、需要整理纪要的职场人（PM / 产品 / 运营）

【核心场景】
当 [会议结束后] 时，
用户想要 [不花30分钟手动整理纪要]，
所以他 [粘贴原始记录，让 AI 替他整理]。

【最小功能边界（MVP）】
✅ 要做：文本输入 + AI 生成摘要 + 复制输出
❌ 不做：登录系统 / 历史记录 / 多人协作（留给下一版）

【成功标准（如何知道做成了）】
- 自己用了一次，觉得确实比手写快
- 给 5 个同事试用，3 个人说"下次还想用"

【最大风险点】
- AI 可能漏掉重要行动项 → 需要用户能手动编辑
- 输入文本过长时 API 超时 → 需要做长度限制和等待提示
```

### 3.2 画出核心用户流程

```
用户打开页面
    ↓
[文本框] 粘贴会议原始记录
    ↓
点击「生成纪要」按钮
    ↓
[加载状态] 流式显示生成过程
    ↓
[结果区域] 结构化纪要展示
    - 会议结论（3条以内）
    - 行动项（人 + 事 + 时间）
    - 待讨论事项（如有）
    ↓
用户可以直接在结果区域编辑
    ↓
点击「复制」，粘贴到钉钉/飞书
```

这个流程图就是你的开发清单，每个节点都要实现。

---

## 五、第二阶段：搭脚手架（项目初始化）

### 4.1 最终项目结构

```
ai-meeting-helper/
├── server.js          # 后端主文件
├── public/
│   ├── index.html     # 前端页面
│   ├── style.css      # 样式
│   └── app.js         # 前端逻辑
├── .env               # 环境变量（不提交 Git）
├── .gitignore
└── package.json
```

### 4.2 创建目录结构

```bash
mkdir public
touch server.js public/index.html public/style.css public/app.js
```

### 4.3 配置 package.json 启动脚本

编辑 `package.json`，在 `scripts` 中添加：

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

---

## 六、System Prompt 设计指南

> System Prompt 是 AI 应用里最容易被忽视、但影响最深远的部分。
> 它决定了 AI 的"角色"和"行为边界"，相当于给员工写岗位说明书。

### 16.1 System Prompt vs User Prompt

| 对比项 | System Prompt | User Prompt |
|------|--------------|------------|
| 位置 | `system` 参数，独立于对话历史 | `messages` 数组中的 `user` 消息 |
| 作用 | 定义 AI 的角色、规则、输出格式 | 携带用户的具体输入内容 |
| 频率 | 每次请求都发送，内容固定 | 每轮对话变化 |
| 优先级 | 高于 User Prompt | 正常优先级 |
| 适合放什么 | 角色定义、格式约束、禁止事项 | 具体数据、用户问题 |

```javascript
// 正确用法：System Prompt 放在 system 参数
const stream = await client.messages.stream({
  model: 'claude-opus-4-6',
  max_tokens: 2048,
  system: '你是会议纪要整理专家。输出必须是 Markdown 格式。不要输出任何额外说明。',
  messages: [
    { role: 'user', content: `请整理以下会议记录：\n${transcript}` }
  ]
});
```

### 16.2 System Prompt 的五个核心模块

一个完整的 System Prompt 应该包含：

```
【模块 1：角色定义】
你是 [角色名]，[核心能力描述]，[服务对象]。

【模块 2：任务说明】
你的主要任务是：[具体职责]。

【模块 3：输出格式】
输出必须严格遵循以下格式：
[格式示例]

【模块 4：行为约束】
- 必须做：[正向规则]
- 禁止做：[负向规则]

【模块 5：边界处理】
如果遇到 [异常情况]，应该 [处理方式]。
```

### 16.3 会议纪要助手的完整 System Prompt 示例

```javascript
const SYSTEM_PROMPT = `
你是"会议精灵"，一位专业的会议纪要整理助手，服务于忙碌的职场人。

【核心任务】
从用户提供的原始会议记录中，精准提炼关键信息，生成结构化、可直接使用的会议纪要。

【输出格式（必须严格遵守）】
## 📋 会议结论
（最多3条，只写最终决策，不写讨论过程。如无明确结论，写"本次会议无明确结论"）
- [结论内容]

## ✅ 行动项
（每条必须含：责任人 + 具体事项 + 截止时间。无责任人写"待确认"，无截止时间写"待定"）
- @[责任人] | [事项描述] | [截止时间]

## ❓ 待讨论事项
（本次会议未决策、需要后续跟进的问题。如无，写"无"）
- [待讨论问题]

【行为约束】
✅ 必须做：
- 忠实于原文，不添加原文中没有的信息
- 行动项中的责任人必须是原文明确提到的人名
- 保留原文中的专有名词、项目名、数字

🚫 禁止做：
- 不要写"根据您提供的记录..."等废话开场
- 不要在格式之外添加任何总结性文字
- 不要猜测或推断原文没有明说的责任人

【边界处理】
- 如果原文少于30字或明显不是会议记录：回复"提供的内容过短或格式不符，请提供完整的会议记录"
- 如果原文是英文：用英文输出纪要
- 如果用户要求修改某一项：只修改该项，其余保持不变
`;
```

### 16.4 常见 System Prompt 设计错误

**错误一：角色定义过于宽泛**
```
❌ 错误："你是一个有用的 AI 助手"
✅ 正确："你是会议纪要整理专家，只处理会议记录，拒绝回答其他类型的问题"
```

**错误二：格式要求模糊**
```
❌ 错误："输出要简洁明了"
✅ 正确："每条行动项不超过50字，结论不超过3条，全部用 Markdown 格式"
```

**错误三：没有明确边界处理**
```
❌ 错误：（不写边界处理，让 AI 自由发挥）
✅ 正确："如果输入不是会议记录，直接回复'请提供会议记录内容'，不要尝试回答"
```

**错误四：在 System Prompt 里放用户数据**
```
❌ 错误：把具体的会议记录内容写死在 System Prompt 里
✅ 正确：System Prompt 只放规则，用户数据通过 messages 传入
```

---

## 七、第三阶段：接通 AI 大脑（API 集成）

这是整个应用的核心，先把 AI 部分跑通，其他都是外壳。

### 5.1 后端服务器（server.js）

```javascript
// server.js
const express = require('express');
const Anthropic = require('@anthropic-ai/sdk');
require('dotenv').config();

const app = express();
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// 解析 JSON 请求体
app.use(express.json());
// 提供静态文件（前端页面）
app.use(express.static('public'));

// ============================================
// 核心 API 路由：处理会议纪要生成请求
// ============================================
app.post('/api/summarize', async (req, res) => {
  const { transcript } = req.body;

  // 输入验证
  if (!transcript || transcript.trim().length === 0) {
    return res.status(400).json({ error: '请提供会议记录内容' });
  }

  if (transcript.length > 50000) {
    return res.status(400).json({ error: '文本过长，请控制在 50000 字以内' });
  }

  try {
    // 设置流式输出的响应头
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    // 调用 Claude API（流式模式）
    const stream = await client.messages.stream({
      model: 'claude-opus-4-6',
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: buildPrompt(transcript)
        }
      ]
    });

    // 逐字符推送给前端
    for await (const chunk of stream) {
      if (chunk.type === 'content_block_delta' && chunk.delta.type === 'text_delta') {
        // SSE 格式：data: {...}\n\n
        res.write(`data: ${JSON.stringify({ text: chunk.delta.text })}\n\n`);
      }
    }

    // 发送结束信号
    res.write(`data: ${JSON.stringify({ done: true })}\n\n`);
    res.end();

  } catch (error) {
    console.error('API 调用失败:', error.message);

    // 如果流还没开始，返回 JSON 错误
    if (!res.headersSent) {
      res.status(500).json({ error: '生成失败，请稍后重试' });
    }
  }
});

// ============================================
// Prompt 构建函数（核心！）
// ============================================
function buildPrompt(transcript) {
  return `你是一位专业的会议纪要整理助手，擅长从混乱的会议记录中提炼关键信息。

请根据以下会议记录，生成一份结构化会议纪要。

【输出格式要求】
严格按照以下 Markdown 格式输出，不要添加任何额外内容：

## 📋 会议结论
（最多3条，每条不超过50字，只写最终决策，不写过程讨论）
- 结论1
- 结论2

## ✅ 行动项
（格式：@责任人 | 事项描述 | 截止时间，如无截止时间写"待定"）
- @张三 | 完成XX方案 | 本周五
- @李四 | 联系XX供应商 | 下周一

## ❓ 待讨论事项
（本次会议未能决策、需要下次讨论的问题，如无则写"无"）
- 待讨论问题1

【重要约束】
- 如果原文中没有提到截止日期，写"待定"，不要编造
- 如果没有明确的责任人，写"待确认"
- 行动项只写需要人去执行的事，不写已完成的事
- 如果信息不足以生成某个部分，直接说明"原文未提及"

【会议记录原文】
${transcript}`;
}

// 启动服务器
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`✅ 服务已启动：http://localhost:${PORT}`);
});
```

### 5.2 先用命令行测试 API 是否通

在写前端之前，先验证后端是否工作：

```bash
# 启动服务器
npm run dev

# 新开一个终端，用 curl 测试
curl -X POST http://localhost:3000/api/summarize \
  -H "Content-Type: application/json" \
  -d '{"transcript": "张三：我们决定下周五上线新功能。李四：我负责测试，本周四完成。"}'
```

看到 SSE 数据流输出 → ✅ 后端正常，继续做前端
看到错误信息 → 先排查后端，常见问题见第十二章

---

## 八、第四阶段：做出界面（前端 UI）

### 6.1 HTML 结构（public/index.html）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI 会议纪要助手</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <header>
      <h1>🎯 AI 会议纪要助手</h1>
      <p class="subtitle">粘贴会议记录，AI 帮你整理成结构化纪要</p>
    </header>

    <main>
      <!-- 输入区域 -->
      <section class="input-section">
        <div class="label-row">
          <label for="transcript">会议原始记录</label>
          <span class="char-count" id="charCount">0 / 50000</span>
        </div>
        <textarea
          id="transcript"
          placeholder="在这里粘贴会议记录、聊天记录或语音转文字内容..."
          rows="12"
        ></textarea>
        <div class="button-row">
          <button id="clearBtn" class="btn-secondary">清空</button>
          <button id="generateBtn" class="btn-primary">
            <span id="btnText">✨ 生成纪要</span>
          </button>
        </div>
      </section>

      <!-- 输出区域（初始隐藏）-->
      <section class="output-section" id="outputSection" style="display:none">
        <div class="label-row">
          <label>AI 生成的会议纪要</label>
          <button id="copyBtn" class="btn-copy">📋 复制全文</button>
        </div>
        <!-- 流式输出内容展示在这里 -->
        <div id="output" class="output-content" contenteditable="true"></div>
        <p class="edit-tip">💡 内容可直接编辑，修改后再复制</p>
      </section>

      <!-- 错误提示 -->
      <div id="errorMsg" class="error-msg" style="display:none"></div>
    </main>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

### 6.2 样式（public/style.css）

```css
/* ===== 基础重置 ===== */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: #f5f7fa;
  color: #333;
  line-height: 1.6;
}

/* ===== 布局 ===== */
.container {
  max-width: 860px;
  margin: 0 auto;
  padding: 40px 20px;
}

header { text-align: center; margin-bottom: 40px; }
header h1 { font-size: 28px; color: #1a1a2e; }
.subtitle { color: #666; margin-top: 8px; font-size: 15px; }

/* ===== 区块卡片 ===== */
.input-section, .output-section {
  background: white;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.06);
  margin-bottom: 24px;
}

/* ===== 标签行 ===== */
.label-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}
label { font-weight: 600; color: #444; font-size: 14px; }
.char-count { font-size: 13px; color: #999; }

/* ===== 文本框 ===== */
textarea {
  width: 100%;
  border: 1.5px solid #e0e0e0;
  border-radius: 8px;
  padding: 14px;
  font-size: 14px;
  line-height: 1.7;
  resize: vertical;
  transition: border-color 0.2s;
  font-family: inherit;
}
textarea:focus { outline: none; border-color: #5b8dee; }

/* ===== 按钮行 ===== */
.button-row {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 14px;
}

.btn-primary {
  background: linear-gradient(135deg, #5b8dee, #3a7bd5);
  color: white;
  border: none;
  padding: 10px 28px;
  border-radius: 8px;
  font-size: 15px;
  cursor: pointer;
  transition: opacity 0.2s, transform 0.1s;
  font-weight: 500;
}
.btn-primary:hover { opacity: 0.9; }
.btn-primary:active { transform: scale(0.97); }
.btn-primary:disabled { opacity: 0.5; cursor: not-allowed; }

.btn-secondary {
  background: #f0f0f0;
  color: #666;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  font-size: 15px;
  cursor: pointer;
  transition: background 0.2s;
}
.btn-secondary:hover { background: #e4e4e4; }

.btn-copy {
  background: none;
  border: 1.5px solid #5b8dee;
  color: #5b8dee;
  padding: 6px 14px;
  border-radius: 6px;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-copy:hover { background: #5b8dee; color: white; }

/* ===== 输出内容 ===== */
.output-content {
  border: 1.5px solid #e8e8e8;
  border-radius: 8px;
  padding: 20px;
  min-height: 120px;
  font-size: 14px;
  line-height: 1.8;
  white-space: pre-wrap;
  word-break: break-word;
}
.output-content:focus { outline: none; border-color: #5b8dee; }

/* Markdown 简单渲染 */
.output-content h2 { font-size: 16px; margin: 16px 0 8px; color: #333; }
.output-content ul { padding-left: 20px; }
.output-content li { margin: 4px 0; }

.edit-tip { font-size: 12px; color: #aaa; margin-top: 10px; }

/* ===== 错误提示 ===== */
.error-msg {
  background: #fff0f0;
  border: 1px solid #ffcccc;
  color: #c0392b;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 14px;
}

/* ===== 加载动画 ===== */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
.loading { animation: pulse 1.2s infinite; }
```

### 6.3 前端逻辑（public/app.js）

```javascript
// public/app.js

// ===== DOM 元素引用 =====
const transcriptEl = document.getElementById('transcript');
const generateBtn  = document.getElementById('generateBtn');
const btnText      = document.getElementById('btnText');
const clearBtn     = document.getElementById('clearBtn');
const outputSection = document.getElementById('outputSection');
const outputEl     = document.getElementById('output');
const copyBtn      = document.getElementById('copyBtn');
const errorMsg     = document.getElementById('errorMsg');
const charCount    = document.getElementById('charCount');

// ===== 字符计数 =====
transcriptEl.addEventListener('input', () => {
  const count = transcriptEl.value.length;
  charCount.textContent = `${count.toLocaleString()} / 50,000`;
  charCount.style.color = count > 45000 ? '#e74c3c' : '#999';
});

// ===== 清空按钮 =====
clearBtn.addEventListener('click', () => {
  transcriptEl.value = '';
  charCount.textContent = '0 / 50,000';
  outputSection.style.display = 'none';
  hideError();
});

// ===== 核心：生成纪要 =====
generateBtn.addEventListener('click', async () => {
  const transcript = transcriptEl.value.trim();
  if (!transcript) {
    showError('请先粘贴会议记录内容');
    return;
  }

  // UI 状态：开始加载
  setLoading(true);
  hideError();
  outputSection.style.display = 'block';
  outputEl.textContent = '';

  try {
    // 调用后端 API，获取 SSE 流
    const response = await fetch('/api/summarize', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ transcript })
    });

    if (!response.ok) {
      const err = await response.json();
      throw new Error(err.error || '请求失败');
    }

    // 读取 SSE 流，逐步渲染文字
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = '';

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      buffer += decoder.decode(value, { stream: true });

      // 按行处理 SSE 数据
      const lines = buffer.split('\n');
      buffer = lines.pop(); // 保留可能不完整的最后一行

      for (const line of lines) {
        if (!line.startsWith('data: ')) continue;
        const jsonStr = line.slice(6); // 去掉 "data: "
        if (!jsonStr.trim()) continue;

        try {
          const data = JSON.parse(jsonStr);
          if (data.done) break;
          if (data.text) {
            outputEl.textContent += data.text;
            // 自动滚动到底部
            outputEl.scrollTop = outputEl.scrollHeight;
          }
        } catch { /* 忽略解析错误 */ }
      }
    }

  } catch (error) {
    showError(error.message || '生成失败，请检查网络后重试');
    outputSection.style.display = 'none';
  } finally {
    setLoading(false);
  }
});

// ===== 复制按钮 =====
copyBtn.addEventListener('click', async () => {
  const text = outputEl.textContent;
  try {
    await navigator.clipboard.writeText(text);
    copyBtn.textContent = '✅ 已复制！';
    setTimeout(() => { copyBtn.textContent = '📋 复制全文'; }, 2000);
  } catch {
    // 降级方案：选中文字提示用户手动复制
    const range = document.createRange();
    range.selectNodeContents(outputEl);
    window.getSelection().removeAllRanges();
    window.getSelection().addRange(range);
    showError('请使用 Ctrl+C 手动复制');
  }
});

// ===== 工具函数 =====
function setLoading(isLoading) {
  generateBtn.disabled = isLoading;
  btnText.textContent = isLoading ? '⏳ 生成中...' : '✨ 生成纪要';
  if (isLoading) btnText.classList.add('loading');
  else btnText.classList.remove('loading');
}

function showError(msg) {
  errorMsg.textContent = msg;
  errorMsg.style.display = 'block';
}

function hideError() {
  errorMsg.style.display = 'none';
}
```

---

## 九、第五阶段：让它更聪明（Prompt 工程）

Prompt 是 AI 应用的"产品核心"，花时间打磨值得。

### 7.1 Prompt 迭代方法论

**Step 1：先跑起来，不求完美**
用最简单的 Prompt，让功能能用就行。

**Step 2：收集 Bad Case**
真实使用中找到 AI 输出不对的案例，记录下来：
```
Bad Case #001
输入：[粘贴原文]
期望输出：[你希望 AI 说什么]
实际输出：[AI 实际说了什么]
问题描述：行动项漏掉了"王五要做的事"
```

**Step 3：针对 Bad Case 改 Prompt**
每次改动只改一个地方，对比前后输出：
```
改动记录：
v1：原始 Prompt
v2：在输出约束中增加"不要遗漏含有动词短语的句子"
v3：在示例中增加了一个多人场景的例子
```

**Step 4：回归测试**
改了新 Prompt 后，用之前所有 Bad Case 重新测试，确认没有引入新问题。

### 7.2 进阶 Prompt 技巧

**技巧一：给 AI 看例子（Few-shot）**

比描述规则更有效，直接给几个输入-输出例子：

```javascript
function buildPrompt(transcript) {
  return `你是会议纪要整理专家。

【示例 1】
输入：
"下午和老板讨论了产品方向，最终决定先做B端。张三负责写方案，下周一给我看。"

输出：
## 📋 会议结论
- 产品方向调整为优先做B端市场

## ✅ 行动项
- @张三 | 撰写B端产品方案 | 下周一

## ❓ 待讨论事项
- 无

【示例 2】
输入：
"我们聊了一下推广的事，感觉还没想清楚，下次再说。预算的问题也悬在那儿。"

输出：
## 📋 会议结论
- 无明确结论

## ✅ 行动项
- 无明确行动项

## ❓ 待讨论事项
- 推广方案待进一步讨论
- 预算方案待确认

---

现在请处理以下会议记录：

${transcript}`;
}
```

**技巧二：让 AI 先分析再输出（Chain of Thought）**

对于复杂场景，让 AI "先想后说"：

```javascript
// 在 Prompt 结尾加上这段，提升推理质量
`
请先在内心分析：
1. 这段对话中有哪些最终决策？（排除讨论过程）
2. 谁承诺了做什么事情？
3. 哪些事情还悬而未决？

然后按照格式输出最终纪要。不要输出你的分析过程，只输出最终纪要。`
```

**技巧三：设置明确的失败处理**

```javascript
// 在约束中加入：
`如果会议记录太短（少于50字）或内容不是会议记录，
请直接回复："内容较少，无法生成有效纪要，请提供更详细的会议记录。"
不要强行生成空洞的纪要。`
```

---

## 十、多轮对话与上下文管理

> 前面的会议纪要助手是"单次问答"模式：用户输入 → AI 输出，结束。
> 但更多 AI 产品需要**多轮对话**——用户可以追问、修改、补充。
> 本章讲清楚多轮对话的实现原理和关键陷阱。

### 15.1 多轮对话的核心原理

LLM 本身是**无状态的**——每次 API 调用都是全新的，模型不记得上次说了什么。
实现"记忆"的唯一方法是：**每次请求都把完整的历史对话塞进去**。

```
第 1 轮请求发送：
  messages: [
    { role: "user", content: "帮我整理这段会议记录：..." }
  ]

第 2 轮请求发送（必须包含第 1 轮的完整历史）：
  messages: [
    { role: "user",      content: "帮我整理这段会议记录：..." },
    { role: "assistant", content: "## 📋 会议结论\n- 决定下周上线..." },
    { role: "user",      content: "行动项里再加一条：王五负责通知客户" }
  ]
```

### 15.2 前端：维护对话历史

```javascript
// public/app.js — 多轮对话版

// 对话历史存储在内存中
let conversationHistory = [];

async function sendMessage(userInput) {
  // 将用户消息加入历史
  conversationHistory.push({
    role: 'user',
    content: userInput
  });

  try {
    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ messages: conversationHistory })
    });

    // 读取流式输出
    let assistantReply = '';
    const reader = response.body.getReader();
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      const lines = decoder.decode(value).split('\n');
      for (const line of lines) {
        if (!line.startsWith('data: ')) continue;
        const data = JSON.parse(line.slice(6));
        if (data.text) {
          assistantReply += data.text;
          // 实时渲染到界面...
        }
      }
    }

    // 将 AI 回复加入历史，供下次请求使用
    conversationHistory.push({
      role: 'assistant',
      content: assistantReply
    });

  } catch (error) {
    // 出错时移除刚添加的用户消息，避免历史污染
    conversationHistory.pop();
    throw error;
  }
}

// 清空对话（开始新会话）
function resetConversation() {
  conversationHistory = [];
}
```

### 15.3 后端：透传历史消息

```javascript
// server.js — 多轮对话路由
app.post('/api/chat', async (req, res) => {
  const { messages } = req.body;

  if (!messages || !Array.isArray(messages) || messages.length === 0) {
    return res.status(400).json({ error: '消息列表不能为空' });
  }

  // 简单校验消息格式
  for (const msg of messages) {
    if (!['user', 'assistant'].includes(msg.role) || !msg.content) {
      return res.status(400).json({ error: '消息格式错误' });
    }
  }

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');

  try {
    const stream = await client.messages.stream({
      model: 'claude-opus-4-6',
      max_tokens: 2048,
      system: '你是一位专业的会议纪要整理助手。用户可能会追问或要求修改，请保持上下文连贯地回复。',
      messages: messages  // 直接透传完整历史
    });

    for await (const chunk of stream) {
      if (chunk.type === 'content_block_delta' && chunk.delta.type === 'text_delta') {
        res.write(`data: ${JSON.stringify({ text: chunk.delta.text })}\n\n`);
      }
    }

    res.write(`data: ${JSON.stringify({ done: true })}\n\n`);
    res.end();

  } catch (error) {
    if (!res.headersSent) {
      res.status(500).json({ error: '生成失败' });
    }
  }
});
```

### 15.4 上下文长度管理（关键陷阱）

随着对话越来越长，塞入的历史也越来越多，会遇到两个问题：

**问题 1：超出模型 Context Window 上限**

```javascript
// 粗略估算 token 数：中文约 1 token/字，英文约 0.75 token/词
function estimateTokens(messages) {
  return messages.reduce((sum, msg) => sum + msg.content.length, 0);
}

// 超出阈值时，移除最早的几轮对话（保留系统提示和最近 N 轮）
function trimHistory(messages, maxTokens = 6000) {
  while (estimateTokens(messages) > maxTokens && messages.length > 2) {
    // 每次移除最早的一组 user+assistant（2条）
    messages.splice(0, 2);
  }
  return messages;
}
```

**问题 2：成本随轮数线性增长**

| 对话轮数 | 每次请求的 token 数（示意） |
|--------|----------------------|
| 第 1 轮 | ~500 tokens |
| 第 5 轮 | ~2500 tokens（5倍） |
| 第 10 轮 | ~5000 tokens（10倍） |

**最佳实践：**
- 设置合理的最大轮数上限（如 10 轮），超出后提示用户开启新会话
- 对于长文档场景，只保留"摘要 + 最近 3 轮"，而不是全量历史
- 每轮对话结束后，在后端把历史写入数据库（详见第十九章），前端只发送 `sessionId`

### 15.5 会话 ID 方案（推荐用于生产）

```javascript
// 更健壮的方案：前端只保存 sessionId，历史由后端管理
// 前端
const sessionId = localStorage.getItem('sessionId') || generateUUID();
localStorage.setItem('sessionId', sessionId);

// 每次请求只发送 sessionId + 新消息
await fetch('/api/chat', {
  method: 'POST',
  body: JSON.stringify({ sessionId, userMessage: input })
});

// 后端从数据库查出历史，追加新消息，调用 API，再存回去
```

---

## 十一、API 成本控制与 Token 管理

> 个人测试可以忽略成本，但一旦有真实用户，成本就是绕不开的问题。
> 本章帮你建立成本意识，并提供几个立竿见影的控制手段。

### 17.1 Token 是什么，怎么估算

Token 是 LLM 计费的基本单位，近似理解：
- 英文：**1 个词 ≈ 1.3 个 token**
- 中文：**1 个汉字 ≈ 1～2 个 token**（具体取决于模型的分词器）

```javascript
// 粗略估算（不含特殊 token 开销）
function roughTokenEstimate(text) {
  // 中文字符按 1.5 倍计，英文按字符数 / 4 估算
  const chineseChars = (text.match(/[\u4e00-\u9fff]/g) || []).length;
  const otherChars = text.length - chineseChars;
  return Math.ceil(chineseChars * 1.5 + otherChars / 4);
}
```

**准确统计**：从 API 响应的 `usage` 字段获取：

```javascript
// 流式结束后，可以从 finalMessage 获取实际 token 用量
const stream = client.messages.stream({ ... });
const finalMessage = await stream.finalMessage();

console.log('实际 token 消耗：', {
  input:  finalMessage.usage.input_tokens,
  output: finalMessage.usage.output_tokens,
  total:  finalMessage.usage.input_tokens + finalMessage.usage.output_tokens
});
```

### 17.2 一次请求的成本计算

以 Claude claude-opus-4-6 为例（价格以官网实时为准）：

```
成本 = (输入 token 数 × 输入单价) + (输出 token 数 × 输出单价)

示例：用户粘贴 2000 字会议记录 → AI 输出 500 字纪要
  输入 token：2000字 × 1.5 + System Prompt 约500 token ≈ 3500 token
  输出 token：500字 × 1.5 ≈ 750 token
  单次成本：约 $0.01 ~ $0.03（视具体模型和价格）

月活 1000 用户，每人每天使用 1 次：
  月成本 ≈ 1000 × 30 × $0.02 = $600/月
```

### 17.3 五个降低成本的实用手段

**① 选合适的模型（最有效）**

不是每个场景都需要最强的模型：

| 场景 | 推荐模型 | 原因 |
|------|---------|-----|
| 格式化输出（纪要、摘要） | claude-haiku-4-5 | 速度快，成本低 10 倍，效果差异小 |
| 复杂推理（代码审查、方案分析） | claude-sonnet-4-6 | 能力和成本的平衡点 |
| 最高精度需求 | claude-opus-4-6 | 用在真正需要的地方 |

```javascript
// 按任务复杂度选模型
function selectModel(taskType) {
  const modelMap = {
    'format':   'claude-haiku-4-5-20251001',  // 格式化任务
    'analysis': 'claude-sonnet-4-6',          // 分析任务
    'creative': 'claude-opus-4-6',            // 创作/复杂推理
  };
  return modelMap[taskType] || 'claude-sonnet-4-6';
}
```

**② 限制输出长度**

```javascript
// max_tokens 按需设置，不要无脑设 4096
// 纪要场景：500 token 足够
const stream = await client.messages.stream({
  model: 'claude-haiku-4-5-20251001',
  max_tokens: 600,  // 而不是默认的 4096
  messages: [...]
});
```

**③ 精简 System Prompt**

每次请求都会发送 System Prompt，要定期审查是否有冗余内容：

```
优化前（800 token）：
"你是一个非常专业的、经验丰富的、擅长处理各种会议记录的AI助手，
你的名字是会议精灵，你服务于忙碌的职场人，你需要根据用户提供的
会议原始记录，精准提炼其中的关键信息，生成结构化的会议纪要..."

优化后（200 token）：
"会议纪要助手。从原始记录中提炼：结论（≤3条）+ 行动项（@人|事|时间）
+ 待讨论事项。忠实原文，不添加猜测内容。"
```

**④ 前端做预校验，减少无效请求**

```javascript
// 在发请求前做基础检查，过滤掉明显无效的输入
function preValidate(text) {
  if (text.length < 30) {
    showError('内容太短，请至少输入 30 字的会议记录');
    return false;
  }
  if (text.length > 50000) {
    showError('内容超出限制，请精简后重试');
    return false;
  }
  return true;
}
```

**⑤ 缓存相同输入的结果**

```javascript
// 对于完全相同的输入，直接返回缓存（适合演示/测试场景）
const cache = new Map();

app.post('/api/summarize', async (req, res) => {
  const cacheKey = require('crypto')
    .createHash('md5')
    .update(req.body.transcript)
    .digest('hex');

  if (cache.has(cacheKey)) {
    // 直接返回缓存的结果（非流式）
    return res.json({ result: cache.get(cacheKey), cached: true });
  }
  // ...正常调用 API，结束后存入缓存
});
```

### 17.4 成本监控

在生产环境中，建议记录每次请求的 token 用量：

```javascript
// 每次 API 调用结束后记录
async function logUsage(userId, usage) {
  await db.insert('api_usage', {
    user_id:       userId,
    input_tokens:  usage.input_tokens,
    output_tokens: usage.output_tokens,
    created_at:    new Date()
  });
}

// 每日汇总，超过阈值发警报
// SELECT DATE(created_at), SUM(input_tokens + output_tokens)
// FROM api_usage GROUP BY DATE(created_at)
```

---

## 十二、第六阶段：处理真实问题（错误 & 边界）

### 8.1 必须处理的 5 类边界情况

**① 空输入**
```javascript
if (!transcript || transcript.trim().length === 0) {
  return res.status(400).json({ error: '请提供会议记录内容' });
}
```

**② 超长文本**
```javascript
// Claude API 有 token 限制，预处理截断
if (transcript.length > 50000) {
  return res.status(400).json({ error: '文本过长，请控制在 50000 字以内' });
}
```

**③ API 超时**
```javascript
// 设置超时控制（30 秒）
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 30000);

try {
  const stream = await client.messages.stream({
    // ...
  }, { signal: controller.signal });
  // ...
} finally {
  clearTimeout(timeout);
}
```

**④ API Key 无效**
```javascript
// 在 server.js 启动时检查
if (!process.env.ANTHROPIC_API_KEY) {
  console.error('❌ 缺少 ANTHROPIC_API_KEY 环境变量');
  process.exit(1);
}
```

**⑤ 前端断网重试**
```javascript
// 在 app.js 中，在 catch 中给用户清晰的错误提示
catch (error) {
  if (error.name === 'AbortError') {
    showError('请求超时，请检查网络连接后重试');
  } else if (!navigator.onLine) {
    showError('网络已断开，请连接网络后重试');
  } else {
    showError('生成失败：' + (error.message || '未知错误'));
  }
}
```

### 8.2 用户体验的细节打磨

| 场景 | 错误做法 | 正确做法 |
|-----|---------|---------|
| AI 生成中 | 页面无任何反馈 | 流式逐字输出 + 加载动画 |
| 生成失败 | "Error: 500" | "生成失败，请稍后重试" |
| 内容为空 | 不提示，按钮无反应 | 聚焦文本框 + 红色提示 |
| 首次使用 | 空白页面 | 内置示例文本（点击可填入） |

---

## 十三、用户认证与多用户隔离

> 个人工具只有你自己用，不需要登录。但一旦分享给他人，就必须考虑：
> 谁在用？每人的数据怎么隔离？怎么防止滥用？

### 18.1 三种认证方案的选择

```
场景 A：内部工具（团队 10 人以内）
  → 简单密码保护，无需真正的用户系统

场景 B：公开产品（需要区分用户、保存历史）
  → JWT 或 Session 认证 + 用户表

场景 C：快速上线（不想自己做认证）
  → 接入第三方认证（GitHub OAuth / 微信登录 / Clerk）
```

### 18.2 方案 A：简单密码保护

最快实现，适合内部工具：

```javascript
// server.js — 添加密码中间件
const ACCESS_PASSWORD = process.env.ACCESS_PASSWORD;

function requirePassword(req, res, next) {
  const password = req.headers['x-access-password'];
  if (password !== ACCESS_PASSWORD) {
    return res.status(401).json({ error: '访问密码错误' });
  }
  next();
}

// 只保护 API 路由，不保护静态文件
app.post('/api/summarize', requirePassword, async (req, res) => {
  // ...
});
```

```javascript
// public/app.js — 发请求时带上密码
const password = localStorage.getItem('accessPassword') || prompt('请输入访问密码：');
localStorage.setItem('accessPassword', password);

const response = await fetch('/api/summarize', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Access-Password': password
  },
  body: JSON.stringify({ transcript })
});
```

### 18.3 方案 B：JWT 用户认证

适合有注册/登录需求的正式产品：

```bash
npm install jsonwebtoken bcryptjs
```

```javascript
// auth.js — 认证逻辑
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

const JWT_SECRET = process.env.JWT_SECRET; // 必须是高强度随机字符串

// 注册
app.post('/api/register', async (req, res) => {
  const { email, password } = req.body;
  const hashed = await bcrypt.hash(password, 10);
  // 存入数据库...
  const token = jwt.sign({ userId, email }, JWT_SECRET, { expiresIn: '7d' });
  res.json({ token });
});

// 登录
app.post('/api/login', async (req, res) => {
  const user = await db.findUserByEmail(req.body.email);
  const valid = await bcrypt.compare(req.body.password, user.passwordHash);
  if (!valid) return res.status(401).json({ error: '密码错误' });
  const token = jwt.sign({ userId: user.id }, JWT_SECRET, { expiresIn: '7d' });
  res.json({ token });
});

// 认证中间件
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1]; // "Bearer <token>"
  if (!token) return res.status(401).json({ error: '请先登录' });
  try {
    req.user = jwt.verify(token, JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Token 已过期，请重新登录' });
  }
}

// 受保护的路由
app.post('/api/summarize', authenticate, async (req, res) => {
  const { userId } = req.user; // 可以用 userId 做数据隔离
  // ...
});
```

### 18.4 防滥用：速率限制

防止单个用户消耗大量 API 额度：

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require('express-rate-limit');

// 每个 IP 每分钟最多 10 次请求
const apiLimiter = rateLimit({
  windowMs: 60 * 1000,  // 1 分钟
  max: 10,
  message: { error: '请求过于频繁，请 1 分钟后再试' },
  standardHeaders: true,
  legacyHeaders: false,
});

app.post('/api/summarize', apiLimiter, async (req, res) => {
  // ...
});
```

---

## 十四、数据持久化：历史记录存储

> 本教程附录提到了"历史记录"方向，本章给出完整实现。
> 从最简单的 LocalStorage，到 SQLite，逐步升级。

### 19.1 方案一：LocalStorage（零后端，最快）

适合个人工具，数据只存在用户浏览器里：

```javascript
// public/app.js — 历史记录管理

const HISTORY_KEY = 'meeting-helper-history';
const MAX_HISTORY = 20;

// 保存一条记录
function saveToHistory(transcript, summary) {
  const history = getHistory();
  history.unshift({  // 插到最前面
    id: Date.now(),
    title: transcript.slice(0, 30) + '...',
    transcript,
    summary,
    createdAt: new Date().toISOString()
  });
  // 最多保留 20 条
  localStorage.setItem(HISTORY_KEY, JSON.stringify(history.slice(0, MAX_HISTORY)));
}

// 读取历史
function getHistory() {
  try {
    return JSON.parse(localStorage.getItem(HISTORY_KEY) || '[]');
  } catch {
    return [];
  }
}

// 渲染历史列表
function renderHistory() {
  const history = getHistory();
  const container = document.getElementById('historyList');
  container.innerHTML = history.map(item => `
    <div class="history-item" onclick="loadHistoryItem(${item.id})">
      <div class="history-title">${item.title}</div>
      <div class="history-date">${new Date(item.createdAt).toLocaleDateString()}</div>
    </div>
  `).join('');
}

// 在生成完成后保存
// saveToHistory(transcript, outputEl.textContent);
```

### 19.2 方案二：SQLite 本地数据库（适合单机部署）

```bash
npm install better-sqlite3
```

```javascript
// db.js — 数据库初始化
const Database = require('better-sqlite3');
const db = new Database('data.db');

// 建表
db.exec(`
  CREATE TABLE IF NOT EXISTS summaries (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id     TEXT,
    title       TEXT,
    transcript  TEXT NOT NULL,
    summary     TEXT NOT NULL,
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP
  )
`);

// 封装常用操作
module.exports = {
  saveSummary(userId, transcript, summary) {
    const title = transcript.slice(0, 30);
    return db.prepare(
      'INSERT INTO summaries (user_id, title, transcript, summary) VALUES (?, ?, ?, ?)'
    ).run(userId, title, transcript, summary);
  },

  getHistory(userId, limit = 20) {
    return db.prepare(
      'SELECT id, title, created_at FROM summaries WHERE user_id = ? ORDER BY created_at DESC LIMIT ?'
    ).all(userId, limit);
  },

  getSummaryById(id, userId) {
    return db.prepare(
      'SELECT * FROM summaries WHERE id = ? AND user_id = ?'
    ).get(id, userId);
  },

  deleteSummary(id, userId) {
    return db.prepare(
      'DELETE FROM summaries WHERE id = ? AND user_id = ?'
    ).run(id, userId);
  }
};
```

```javascript
// server.js — 使用数据库
const db = require('./db');

// 生成纪要后保存（在 SSE 流结束后）
app.post('/api/summarize', authenticate, async (req, res) => {
  // ...流式输出...

  let fullSummary = '';
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta') {
      fullSummary += chunk.delta.text;
      res.write(`data: ${JSON.stringify({ text: chunk.delta.text })}\n\n`);
    }
  }

  // 流结束后，保存到数据库
  db.saveSummary(req.user.userId, req.body.transcript, fullSummary);

  res.write(`data: ${JSON.stringify({ done: true })}\n\n`);
  res.end();
});

// 查询历史记录列表
app.get('/api/history', authenticate, (req, res) => {
  const history = db.getHistory(req.user.userId);
  res.json(history);
});

// 查询单条历史详情
app.get('/api/history/:id', authenticate, (req, res) => {
  const item = db.getSummaryById(req.params.id, req.user.userId);
  if (!item) return res.status(404).json({ error: '记录不存在' });
  res.json(item);
});
```

---

## 十五、AI 应用测试策略

> 传统软件：输入确定 → 输出确定，测试直接断言结果。
> AI 应用：输入确定 → 输出**不确定**，传统测试方法失效。
> 本章讲 AI 应用特有的测试思路。

### 20.1 AI 应用测试的三个层次

```
层次 1：工程层测试（传统单元测试）
  测什么：API 路由、数据库操作、输入校验、错误处理
  怎么测：Mock 掉 AI 调用，测试工程代码的正确性
  工具：Jest / Vitest / Pytest

层次 2：Prompt 回归测试（Golden Set 测试）
  测什么：特定输入下，AI 输出是否满足预期标准
  怎么测：维护一组"黄金测试用例"，每次改 Prompt 后运行
  工具：自定义脚本 + 人工评分 / LLM-as-Judge

层次 3：端到端测试
  测什么：用户完整使用流程（页面操作 → API → AI → 结果展示）
  怎么测：Playwright / Cypress 模拟用户操作
  工具：Playwright
```

### 20.2 层次一：工程层单元测试

```javascript
// tests/server.test.js
const request = require('supertest');
const app = require('../server');

// Mock Anthropic SDK，避免真实 API 调用
jest.mock('@anthropic-ai/sdk', () => ({
  default: jest.fn().mockImplementation(() => ({
    messages: {
      stream: jest.fn().mockReturnValue({
        [Symbol.asyncIterator]: async function* () {
          yield { type: 'content_block_delta', delta: { type: 'text_delta', text: '## 📋 会议结论\n- 决定下周上线' } };
          yield { type: 'message_stop' };
        }
      })
    }
  }))
}));

describe('POST /api/summarize', () => {
  test('空输入返回 400', async () => {
    const res = await request(app)
      .post('/api/summarize')
      .send({ transcript: '' });
    expect(res.status).toBe(400);
    expect(res.body.error).toContain('请提供');
  });

  test('超长输入返回 400', async () => {
    const res = await request(app)
      .post('/api/summarize')
      .send({ transcript: 'a'.repeat(50001) });
    expect(res.status).toBe(400);
  });

  test('正常输入返回 SSE 流', async () => {
    const res = await request(app)
      .post('/api/summarize')
      .send({ transcript: '张三：下周五上线新功能。李四：我负责测试。' });
    expect(res.status).toBe(200);
    expect(res.headers['content-type']).toContain('text/event-stream');
  });
});
```

### 20.3 层次二：Prompt 回归测试（Golden Set）

维护一份测试用例 YAML 文件：

```yaml
# tests/golden_cases.yaml

- id: TC001
  name: 标准会议记录
  input: |
    张三：我们决定下周五上线B端功能。
    李四：我负责测试，周四完成。
    王五：UI 稿这周三给到研发。
  expectations:
    - contains: "@李四"
    - contains: "周四"
    - contains: "@王五"
    - not_contains: "张三" # 张三没有行动项
    - format: markdown_headers  # 包含 ## 标题

- id: TC002
  name: 无明确结论的会议
  input: |
    今天讨论了推广方案，大家意见不一，还没定下来。预算也没说清楚。
  expectations:
    - section_empty: "行动项"  # 行动项应为空
    - contains: "待讨论"
```

```javascript
// tests/run_golden.js — 运行黄金测试
const Anthropic = require('@anthropic-ai/sdk');
const yaml = require('js-yaml');
const fs = require('fs');

const client = new Anthropic();
const cases = yaml.load(fs.readFileSync('tests/golden_cases.yaml', 'utf8'));

async function runCase(tc) {
  const result = await client.messages.create({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 600,
    system: SYSTEM_PROMPT,
    messages: [{ role: 'user', content: tc.input }]
  });

  const output = result.content[0].text;
  const failures = [];

  for (const exp of tc.expectations) {
    if (exp.contains && !output.includes(exp.contains)) {
      failures.push(`期望包含 "${exp.contains}"，但未找到`);
    }
    if (exp.not_contains && output.includes(exp.not_contains)) {
      failures.push(`期望不包含 "${exp.not_contains}"，但找到了`);
    }
  }

  return { id: tc.id, name: tc.name, passed: failures.length === 0, failures, output };
}

// 运行所有用例
(async () => {
  console.log('🧪 运行 Prompt 回归测试...\n');
  let passed = 0, failed = 0;

  for (const tc of cases) {
    const result = await runCase(tc);
    if (result.passed) {
      console.log(`✅ ${result.id}: ${result.name}`);
      passed++;
    } else {
      console.log(`❌ ${result.id}: ${result.name}`);
      result.failures.forEach(f => console.log(`   - ${f}`));
      failed++;
    }
  }

  console.log(`\n结果：${passed} 通过 / ${failed} 失败`);
  if (failed > 0) process.exit(1);
})();
```

### 20.4 Prompt 测试的实用原则

| 原则 | 说明 |
|-----|-----|
| 每次只改一个变量 | 修改 Prompt 时只改一处，才能知道是哪里影响了输出 |
| 先写测试用例再改 Prompt | 明确"什么叫正确"，再动手优化 |
| 保留历史 Bad Case | 每次改 Prompt 后，用所有历史 Bad Case 验证没有回退 |
| 对比测试 | 同一个 Case，用旧 Prompt 和新 Prompt 各跑一次，直观对比 |
| 不要 100% 追求通过率 | AI 有随机性，80% 通过率的 Prompt 比 100% 的复杂 Prompt 更实用 |

---

## 十六、第七阶段：部署上线

### 9.1 本地验证

```bash
# 确认本地完全跑通
npm start

# 用真实的会议记录完整测试一遍
# 测试清单：
# □ 正常输入 → 正常输出
# □ 空输入 → 错误提示
# □ 超长文本 → 错误提示
# □ 复制按钮工作
# □ 清空按钮工作
```

### 9.2 部署到 Vercel（推荐，免费）

**方案：将 Express 服务部署为 Serverless Function**

1. 安装 Vercel CLI
```bash
npm install -g vercel
```

2. 创建 `vercel.json` 配置文件
```json
{
  "version": 2,
  "builds": [
    { "src": "server.js", "use": "@vercel/node" }
  ],
  "routes": [
    { "src": "/(.*)", "dest": "server.js" }
  ]
}
```

3. 部署
```bash
vercel

# 按提示操作：
# - 链接或创建项目
# - 配置环境变量（会提示你输入 ANTHROPIC_API_KEY）
```

4. 在 Vercel 控制台配置环境变量
   - 进入项目设置 → Environment Variables
   - 添加 `ANTHROPIC_API_KEY`，值为你的 API Key

5. 重新部署生效
```bash
vercel --prod
```

### 9.3 部署安全自查

```
上线前必查：
□ .env 文件不在 Git 仓库中
□ ANTHROPIC_API_KEY 已配置为环境变量（不是硬编码在代码里）
□ 没有在代码中 console.log(apiKey) 这类泄漏操作
□ 输入长度已限制（防止恶意超长请求）
□ 错误信息不暴露内部实现细节
```

---

## 十七、第八阶段：收集反馈 & 迭代

上线不是终点，而是真正学习的开始。

### 10.1 最小可行的反馈系统

在输出区域下方加一个简单评分：

```html
<!-- 在 output-section 内添加 -->
<div class="feedback-row" id="feedbackRow" style="display:none">
  <span>这份纪要对你有帮助吗？</span>
  <button class="feedback-btn" data-value="1">👍 有帮助</button>
  <button class="feedback-btn" data-value="0">👎 不准确</button>
</div>
```

```javascript
// 生成完成后显示评分
// 点击 👎 时，弹出文本框收集具体问题
document.querySelectorAll('.feedback-btn').forEach(btn => {
  btn.addEventListener('click', async (e) => {
    const value = e.target.dataset.value;
    await fetch('/api/feedback', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        helpful: value === '1',
        transcript: transcriptEl.value,
        output: outputEl.textContent
      })
    });
    document.getElementById('feedbackRow').innerHTML = '感谢反馈！';
  });
});
```

### 10.2 迭代优先级框架

收到反馈后，按以下优先级决定下一步：

```
P0（本周必修）：
- 功能无法使用（Bug）
- 数据错误（AI 生成了错误事实）

P1（下次版本）：
- 超过 30% 用户反馈某类结果不准确
- 核心流程有明显体验问题

P2（看情况）：
- 少数人反馈的细节优化
- 加速/UI 美化

P3（暂不做）：
- 只有 1-2 人提到的特殊需求
- 与核心用户场景无关的功能请求
```

### 10.3 版本节奏建议

```
Week 1-2：种子用户内测（10-20人）
  → 核心流程 Bug 修复
  → Prompt 基础优化

Week 3-4：小范围扩散（50-100人）
  → 基于反馈数据调整
  → 加入最高优先级的新功能

Month 2：持续迭代
  → 每2周一个小版本
  → 每次都附上 Changelog 告知用户更新内容
```

---

## 十八、完整项目结构参考

```
ai-meeting-helper/
├── server.js                 # Express 后端 + AI 路由
├── public/
│   ├── index.html            # 单页面前端
│   ├── style.css             # 样式
│   └── app.js                # 前端逻辑（fetch + SSE 处理）
├── vercel.json               # 部署配置
├── .env                      # 本地环境变量（不提交 Git）
├── .gitignore
└── package.json
```

---

## 十九、常见卡点 & 解决方案

### 问题 1：`Error: ANTHROPIC_API_KEY is not set`
```bash
# 检查 .env 文件是否存在
cat .env

# 确认 server.js 顶部有 require('dotenv').config()
# 确认变量名拼写正确：ANTHROPIC_API_KEY
```

### 问题 2：前端拿不到流式输出（一直加载，最后才全部出现）
```javascript
// 原因：浏览器默认会缓冲 SSE 响应
// 解决：确保服务器设置了正确的响应头
res.setHeader('X-Accel-Buffering', 'no');  // Nginx 穿透缓冲
res.flushHeaders();                         // 立即发送响应头
```

### 问题 3：部署后 API Key 不生效
```
检查项：
□ Vercel 控制台中环境变量是否已保存
□ 环境变量是否选择了正确的环境（Production / Preview / Development）
□ 修改环境变量后是否重新部署了
```

### 问题 4：输入中文时字符统计不准
```javascript
// length 统计的是 UTF-16 编码单元，中文字符也是 1
// 如需字节数统计（用于后端限制）：
const byteLength = new Blob([text]).size;
```

### 问题 5：手机端粘贴后文字框很小
```css
/* 在 style.css 中加入 */
@media (max-width: 600px) {
  textarea { rows: 8; font-size: 16px; }
  /* font-size: 16px 防止 iOS Safari 自动缩放 */
}
```

### 问题 6：AI 输出的 Markdown 没有渲染（显示原始符号）
```javascript
// 方案 A（简单）：用 marked.js 渲染 Markdown
// 在 HTML 中引入：
// <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>

// 在 app.js 中，收到完整输出后渲染：
outputEl.innerHTML = marked.parse(outputEl.textContent);

// 方案 B（更安全）：用 DOMPurify 防止 XSS
outputEl.innerHTML = DOMPurify.sanitize(marked.parse(text));
```

---

## 二十、Python 技术栈实现方案

> 前面章节用 Node.js + Express 实现了完整的 AI 会议纪要助手。
> 如果你更熟悉 Python，本章提供等价的 Python + FastAPI 实现，逻辑完全一致，可以直接替换。

### 14.1 环境准备

```bash
# 创建虚拟环境（推荐）
python -m venv venv
source venv/bin/activate      # Linux / macOS
venv\Scripts\activate         # Windows

# 安装依赖
pip install fastapi uvicorn anthropic python-dotenv
```

`requirements.txt`：

```
fastapi==0.110.0
uvicorn==0.27.0
anthropic==0.21.0
python-dotenv==1.0.1
```

### 14.2 后端服务（main.py）

```python
# main.py
import os
import json
from fastapi import FastAPI, HTTPException
from fastapi.responses import StreamingResponse
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel
import anthropic
from dotenv import load_dotenv

load_dotenv()

app = FastAPI()
client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

# 挂载前端静态文件（与 Node.js 版的 express.static('public') 等价）
app.mount("/static", StaticFiles(directory="public"), name="static")


class TranscriptRequest(BaseModel):
    transcript: str


def build_prompt(transcript: str) -> str:
    return f"""你是一位专业的会议纪要整理助手，擅长从混乱的会议记录中提炼关键信息。

请根据以下会议记录，生成一份结构化会议纪要。

【输出格式要求】
严格按照以下 Markdown 格式输出：

## 📋 会议结论
（最多3条，只写最终决策）
- 结论1

## ✅ 行动项
（格式：@责任人 | 事项描述 | 截止时间）
- @张三 | 完成XX方案 | 本周五

## ❓ 待讨论事项
- 待讨论问题1

【重要约束】
- 没有截止日期写"待定"，没有责任人写"待确认"
- 如果信息不足，直接说明"原文未提及"

【会议记录原文】
{transcript}"""


@app.post("/api/summarize")
async def summarize(req: TranscriptRequest):
    if not req.transcript or not req.transcript.strip():
        raise HTTPException(status_code=400, detail="请提供会议记录内容")

    if len(req.transcript) > 50000:
        raise HTTPException(status_code=400, detail="文本过长，请控制在 50000 字以内")

    def generate():
        # 使用 with 语句确保流正确关闭
        with client.messages.stream(
            model="claude-opus-4-6",
            max_tokens=2048,
            messages=[{"role": "user", "content": build_prompt(req.transcript)}],
        ) as stream:
            for text in stream.text_stream:
                # SSE 格式推送
                yield f"data: {json.dumps({'text': text}, ensure_ascii=False)}\n\n"

        yield f"data: {json.dumps({'done': True})}\n\n"

    return StreamingResponse(
        generate(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
            "X-Accel-Buffering": "no",
        },
    )


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=3000, reload=True)
```

### 14.3 启动与测试

```bash
# 开发模式（热重载）
uvicorn main:app --reload --port 3000

# 测试 API
curl -X POST http://localhost:3000/api/summarize \
  -H "Content-Type: application/json" \
  -d '{"transcript": "张三：下周五上线新功能。李四：我负责测试。"}'
```

### 14.4 Python 版部署到 Vercel

创建 `vercel.json`：

```json
{
  "builds": [{ "src": "main.py", "use": "@vercel/python" }],
  "routes": [{ "src": "/(.*)", "dest": "main.py" }]
}
```

> **Node.js vs Python 怎么选？**
> 没有绝对优劣——用你更熟的那个。
> 如果你的团队主力是数据/算法方向，Python 生态（numpy、pandas、sklearn）更方便后续集成；
> 如果你的团队主力是前端，Node.js 全栈更统一。

---

## 二十一、番外篇：用百度秒哒零代码制作应用

> 前面十二章讲的是"写代码"的方式做 AI 应用。但如果你是产品人、运营、设计师，
> 或者只是想快速验证一个想法——**百度秒哒**提供了一条完全不需要写代码的路径。
> 本章覆盖从简单 H5 小工具到完整站点级应用的全场景，以及如何写好 Prompt 和需求文档。

---

### 13.1 秒哒是什么？

**百度秒哒（MeDo）** 是百度于 2024 年 11 月发布、2025 年 3 月正式全量上线的**无代码 AI 全栈应用搭建平台**，底层由文心大模型驱动，通过多智能体协作实现"说话即编程"。

2025 年 11 月升级至 2.0，核心能力从"生成原型"进化为**"全栈应用一键生成"**——不再只是演示用的静态界面，而是具备完整前端、后端、数据库、AI 逻辑、云端部署的**生产可用应用**。

**官方入口：** `https://www.miaoda.cn`

**关键数据（截至 2025 年底）：**

| 数据项 | 数值 |
|---|---|
| 累计生成商业应用 | 50 万+ 个 |
| 用户中非程序员比例 | **81%** |
| 服务用户数 | 1000 万+ |
| 已接入支付的应用 | 2 万+ 个 |
| 累计真实交易笔数 | 8 万+ 笔 |
| 累计创造价值 | 超 50 亿元 |

> 这组数据说明秒哒已经不是玩具，而是真正在商业场景落地的工具。

---

### 13.2 秒哒能做什么规模的应用？

秒哒支持四个规模层次，从最简单的 H5 小工具到完整的站点级产品：

```
Level 1：单页 H5 工具（5～15 分钟）
──────────────────────────────────
特征：单页面、无后端、无数据存储
典型产品：BMI 计算器、文字处理工具、颜色选择器、
          表情生成器、倒计时工具、随机点名器
输出形式：H5 链接，可分享到微信朋友圈

Level 2：带数据的轻应用（20～60 分钟）
──────────────────────────────────
特征：有后端存储、支持数据提交和查询
典型产品：活动报名页、问卷收集、投票系统、
          AI 内容生成工具、知识问答助手
输出形式：H5 链接 + 管理后台

Level 3：多页面功能站点（1～4 小时）
──────────────────────────────────
特征：多页面、有用户系统（可选）、完整业务流程
典型产品：产品官网、企业介绍站、知识付费平台、
          内容创作助手、团队协作工具
输出形式：独立域名站点 / 微信小程序

Level 4：完整商业应用（4～8 小时，多轮迭代）
──────────────────────────────────
特征：有用户系统、支付能力、数据后台、完整商业闭环
典型产品：电商小程序、课程平台、预约系统、
          SaaS 轻应用、垂直行业工具
输出形式：小程序 + Web 端双端，支持支付变现
```

---

### 13.3 平台界面详解：双区域协作模式

打开秒哒创建/编辑应用后，界面分为两个核心区域：

```
┌────────────────────────────────────────────────────────┐
│  顶部工具栏：撤销 | 重做 | 历史版本 | 发布 | 应用设置   │
├─────────────────────────┬──────────────────────────────┤
│                         │                              │
│   区域 ①               │   区域 ②                    │
│   无代码画布            │   自然语言对话框              │
│                         │                              │
│   所见即所得的可视化编辑  │   直接用文字描述修改需求      │
│                         │                              │
│   ▸ 点击任意元素直接修改  │   ▸ "把按钮改成蓝色"        │
│   ▸ 拖拽组件调整布局     │   ▸ "增加手机号验证"         │
│   ▸ 上传自定义图片/图标  │   ▸ "再加一个导出功能"       │
│   ▸ 修改文字/颜色/字体   │   ▸ "参考XXX风格重新设计"    │
│                         │                              │
└─────────────────────────┴──────────────────────────────┘
         预览区（实时同步更新，手机/PC双视图切换）
```

**两个区域的使用时机：**

| 操作类型 | 用哪个区域 | 原因 |
|--------|----------|------|
| 改一个按钮的颜色 | 区域 ① 画布 | 直接点击更快 |
| 改一段文字内容 | 区域 ① 画布 | 所见即所得 |
| 增加一个新功能模块 | 区域 ② 对话 | AI 生成比手动搭建快 |
| 调整整体风格 | 区域 ② 对话 | 描述意图，AI 全局处理 |
| 修复一个逻辑 Bug | 区域 ② 对话 | AI 理解上下文，精准修复 |
| 换图标 | 区域 ① 画布 | 直接上传 SVG |

**版本管理（重要！）：**
- 每次大改前建议先**创建新版本**（顶部工具栏 → 历史版本 → 创建新版本）
- 支持随时一键回滚到任意历史版本
- 快捷键：`Ctrl+Z` 撤销，`Ctrl+Shift+Z` 重做

---

### 13.4 Prompt 写法完全指南

Prompt 是决定秒哒输出质量的核心。本节从最简单到最复杂，分级讲解写法。

#### 13.4.1 Level 1：简单工具的 Prompt（一段话搞定）

适合单页 H5 工具，直接写清楚功能即可。

**公式：做什么 + 输入是什么 + 输出是什么 + 界面风格**

```
示例 1：BMI 计算器
────────────────
做一个 BMI 计算器 H5 页面。
用户输入身高（cm）和体重（kg），
点击计算后显示 BMI 数值和健康状态（偏瘦/正常/超重/肥胖）。
界面简洁，配色清新，支持手机端访问。
```

```
示例 2：随机点名器
────────────────
做一个课堂随机点名工具。
老师可以输入学生名单（每行一个名字），
点击"随机点名"按钮，大屏动画展示被选中的学生名字。
界面要有趣，像抽奖一样有转动动效，配色活泼。
```

```
示例 3：倒计时工具
────────────────
做一个活动倒计时 H5 页面。
标题写"距XXX活动还有"，倒计时显示天/时/分/秒。
目标时间设为 2026 年 6 月 18 日 0 点。
背景用渐变红色，数字大而醒目，适合在大屏上展示。
```

---

#### 13.4.2 Level 2：带后端功能的 Prompt（分模块描述）

当应用涉及数据存储、AI 生成时，需要**明确说清楚数据流**。

**公式：功能模块 + 数据是什么/怎么存 + AI 做什么 + 谁能看到什么**

```
示例：活动报名系统
────────────────
做一个活动报名页面，分两部分：

【报名页面（用户侧）】
- 填写：姓名、手机号、所在公司、职位（下拉：产品/技术/运营/其他）
- 手机号格式验证，不能重复报名
- 提交成功后显示："报名成功！活动时间：5月20日 14:00，地点：XX大厦B座"

【管理后台（管理员侧）】
- 访问密码：[自定义密码]
- 展示报名列表（姓名/手机/公司/职位/报名时间）
- 支持按职位筛选、按时间排序
- 底部显示"当前报名人数：XX人"
- 可以导出 Excel

界面风格：报名页简洁大方，管理后台数据清晰。
```

```
示例：AI 会议纪要助手（带后端版）
────────────────
做一个 AI 会议纪要生成工具：

【用户端】
- 文本框：粘贴会议记录（最多 5000 字）
- 下拉选择：会议类型（产品评审/周例会/客户沟通/其他）
- 点击「生成纪要」：
  - AI 自动提取：会议结论（3 条以内）、行动项（含责任人+截止日期）、待讨论事项
  - 流式逐字输出效果
  - 输出区域可直接编辑
  - 一键复制全文按钮

【后端数据】
- 每次生成的纪要自动保存（无需登录，用浏览器本地标识）
- 页面下方展示「最近 5 次」历史纪要，可点击查看

界面：白底，主色调蓝色，简洁专业。
```

---

#### 13.4.3 Level 3：多页面站点的 Prompt（结构化分页描述）

多页面应用需要先描述**整体结构**，再逐页说明。

**公式：整体定位 → 页面列表 → 每页详细功能 → 全局规则**

```
示例：AI 产品工具站
────────────────
做一个面向产品经理的 AI 效率工具网站，包含以下 4 个工具页面：

【整体设计】
- 顶部导航：网站 Logo（文字"PM Tools"）+ 4个工具入口 + 颜色切换（白天/夜间）
- 主色调：深蓝 + 白，字体清晰，专业感
- 每个工具页有独立的 URL 路径

【页面 1：会议纪要生成】路径 /meeting
- 粘贴会议记录 → AI 生成结论+行动项+待讨论
- 支持下载 .txt 文件

【页面 2：用户故事生成】路径 /userstory
- 填写：用户角色、使用场景、想要达成的目标
- AI 生成符合 INVEST 原则的用户故事 + 验收标准（AC）
- 支持复制单条或全部

【页面 3：竞品分析助手】路径 /competitor
- 输入：你的产品名称、竞品名称（最多3个）、分析维度（可多选）
- AI 生成结构化竞品对比表格
- 支持导出 Markdown

【页面 4：PRD 大纲生成】路径 /prd
- 输入：功能名称、背景描述、目标用户
- AI 生成标准 PRD 目录结构 + 各章节要点
- 支持一键复制到飞书/钉钉文档

【全局规则】
- 所有 AI 功能调用文心大模型
- 所有页面响应式，手机端正常使用
- 不需要登录和数据存储
```

---

#### 13.4.4 Level 4：商业应用的 Prompt（需配合需求文档）

复杂商业应用**不建议只用 Prompt**，而是写一份需求文档上传给秒哒（详见 13.5 节）。

但如果想用纯 Prompt，要覆盖：**用户体系 + 核心业务流程 + 支付逻辑 + 权限设计 + 数据后台**。

```
示例：付费简历优化工具
────────────────
做一个付费的简历优化 SaaS 小工具，完整商业闭环：

【用户流程】
1. 进入首页 → 看到产品介绍和价格（9.9元/次，99元/月无限次）
2. 点击「立即优化」→ 进入编辑页
3. 粘贴简历文本（纯文字，最多 3000 字）
4. 选择目标职位（产品经理/运营/设计师/前端开发/后端开发）
5. 选择优化方向（多选：措辞优化/结构调整/亮点提炼/关键词补充）
6. 点击「生成预览」→ 展示 AI 分析摘要（前 100 字可见，后面模糊）
7. 点击「解锁完整优化」→ 唤起微信/支付宝支付弹窗
8. 支付成功 → 完整优化建议解锁，可复制/下载

【管理后台】
- 管理员密码登录
- 查看订单列表（时间/金额/产品/状态）
- 查看今日收入、本月收入汇总
- 退款操作（标记退款状态）

【技术要求】
- 接入微信支付和支付宝支付（沙箱模式先测试）
- 每次优化结果保存 7 天，支付用户可回看
- 手机端优化体验（主要场景是手机打开）

【界面风格】
- 首页：干净、专业、信任感强，有用户案例展示区（伪造 3 个好评）
- 工具页：简洁，步骤感清晰，降低用户焦虑
- 主色：深蓝 + 金黄色（付费按钮用金黄色突出）
```

---

#### 13.4.5 Prompt 写法的通用避坑清单

```
❌ 常见错误 → ✅ 正确做法

太模糊：
"做个好看的工具"
→ "做一个白底简洁风格的工具，主色调蓝色，无过度动效"

只说功能，不说数据：
"有个列表"
→ "列表展示：字段包含姓名/手机/提交时间，支持按时间倒序排列"

没说权限边界：
"管理员能看"
→ "管理员通过密码 [自定义密码] 访问后台，普通用户只能提交数据，看不到其他人的记录"

不说异常处理：
（不提）
→ "手机号格式错误时提示'请输入正确的11位手机号'；已报名重复提交时提示'该手机号已报名'"

把所有功能堆在一起：
"然后还要可以...还要可以...还要可以..."
→ 分【核心功能】【进阶功能】【暂不需要】三层，帮 AI 理清优先级
```

---

### 13.5 复杂应用：用需求文档替代 Prompt

当应用复杂到 Prompt 写起来超过 500 字，**改用需求文档**是更好的方式。

#### 13.5.1 为什么需求文档比长 Prompt 更有效？

| 对比项 | 超长 Prompt | 需求文档 |
|------|-----------|--------|
| 结构清晰度 | 容易混乱 | 有目录，层次分明 |
| AI 理解准确性 | 关键信息可能被忽略 | 结构化，易于 AI 解析 |
| 修改方便性 | 要重新写 | 局部修改即可 |
| 团队协作 | 无法复用 | 可共享给团队 |
| 迭代追踪 | 历史版本难追溯 | 文档可版本管理 |

#### 13.5.2 面向秒哒的需求文档模板

以下模板专门为秒哒设计，AI 能高效解析：

```markdown
# [应用名称] 需求文档 v1.0

## 一、产品定位（一句话）
[面向XXX用户，解决XXX问题的XXX工具]

## 二、目标用户
- 主要用户：[描述]
- 使用场景：[在什么时候、什么地方、为了什么使用]

## 三、页面结构
[列出所有页面，注明路径]
- /index：首页
- /tool：工具主页
- /admin：管理后台（密码保护）

## 四、功能详细说明

### 4.1 [页面/功能名称]

**用户流程：**
1. 用户进入页面，看到...
2. 用户操作...，系统响应...
3. 完成后，用户看到...

**界面元素：**
- 标题文字：[具体文字内容]
- 输入框：[标签/占位符/字数限制]
- 按钮：[文字/颜色/点击行为]
- 列表：[字段/排序/筛选]

**业务规则：**
- 规则1：[具体判断逻辑]
- 规则2：...

**异常处理：**
- 情况1：当[条件]时，提示"[具体文案]"
- 情况2：...

### 4.2 [下一个功能模块]
...（同上结构）

## 五、数据说明
| 字段名 | 类型 | 说明 | 是否必填 |
|------|------|------|--------|
| 姓名 | 文本 | 用户真实姓名 | 是 |
| 手机号 | 数字 | 11位，格式验证 | 是 |
| ...  | ...  | ...           | ... |

## 六、界面风格
- 整体风格：[简洁/科技/活泼/专业...]
- 主色调：[颜色描述或色值]
- 字体偏好：[无特殊要求/具体要求]
- 参考风格：[类比产品，如"参考 Notion 的简洁感"]

## 七、不需要的功能（边界声明）
- 不需要：用户注册/登录系统
- 不需要：社交分享功能
- 不需要：[其他明确排除的功能]

## 八、发布要求
- 发布形式：[ H5链接 / 微信小程序 / Web站点 ]
- 是否需要自定义域名：[是/否]
- 是否需要接入支付：[是/否，如是则说明金额]
```

#### 13.5.3 完整示例：「团队日报系统」需求文档

```markdown
# 团队日报系统 需求文档 v1.0

## 一、产品定位
面向 5～20 人小团队的每日工作进展收集和汇总工具，
替代在群里发文字日报的方式，提升日报质量和管理效率。

## 二、目标用户
- 主要用户：团队成员（提交日报）、Team Leader（查看汇总）
- 使用场景：每天下班前（17:30～18:30），用手机或电脑提交当天工作进展

## 三、页面结构
- /：提交日报页（默认入口）
- /summary：日报汇总页（需密码）

## 四、功能详细说明

### 4.1 提交日报页（/）

**用户流程：**
1. 进入页面，顶部显示当日日期（如"2026年5月2日 星期六"）
2. 填写姓名（文本框）
3. 填写今日完成事项（多行文本，最多500字）
4. 填写明日计划（多行文本，最多300字，可选填）
5. 填写风险和阻塞（多行文本，可选填）
6. 点击「提交日报」按钮
7. 提交成功显示：「日报已提交，感谢今天的努力！」

**界面元素：**
- 页面标题：「📋 每日日报」
- 日期自动显示当天
- 姓名输入框（下拉列表，预设成员名单）
  - 成员名单：张三、李四、王五、赵六、陈七（共5人）
- 「今日完成」文本域：高度4行，占位符「今天完成了什么？」
- 「明日计划」文本域：高度3行，占位符「明天打算做什么？（可不填）」
- 「风险阻塞」文本域：高度2行，占位符「有什么需要协助解决的？（可不填）」
- 提交按钮：绿色，文字「✅ 提交日报」

**业务规则：**
- 同一天同一姓名只能提交一次（重复提交提示"今日日报已提交，如需修改请联系管理员"）
- 「今日完成」为必填项，其余可选

**异常处理：**
- 姓名未选：「请选择你的姓名」
- 「今日完成」为空：「请填写今日完成的工作内容」

### 4.2 日报汇总页（/summary）

**用户流程：**
1. 访问 /summary，弹出密码框（密码：team2026）
2. 验证通过，进入汇总页
3. 默认显示今日所有日报
4. 可切换查看历史日期（日历控件选择日期）
5. 点击某人日报条目，展开详情
6. 页面底部显示 AI 生成的今日团队摘要

**界面元素：**
- 顶部：日期选择器 + 「刷新」按钮
- 每人日报卡片：姓名 + 提交时间 + 今日完成（摘要，超过50字折叠）
- 今日未提交成员：灰色显示「未提交」
- 底部 AI 摘要区：点击「生成今日团队摘要」按钮，AI 整合所有日报生成一段 150 字的综合摘要

**AI 摘要格式：**
今日团队完成情况：[整体进展描述]
主要亮点：[2～3条]
需关注风险：[如有，列出1～2条；如无，写"无明显风险"]

## 五、数据说明
| 字段 | 类型 | 说明 | 必填 |
|-----|------|------|-----|
| 姓名 | 文本 | 从预设列表选择 | 是 |
| 日期 | 日期 | 自动取当天 | 自动 |
| 今日完成 | 长文本 | 500字上限 | 是 |
| 明日计划 | 长文本 | 300字上限 | 否 |
| 风险阻塞 | 长文本 | 200字上限 | 否 |
| 提交时间 | 时间戳 | 自动记录 | 自动 |

## 六、界面风格
- 整体：温暖、轻松，不要冷冰冰的企业感
- 主色调：橙色（#FF8C00）+ 白色
- 字体：大而清晰，适合手机端
- 参考：类似"打卡工具"的轻松感

## 七、不需要的功能
- 不需要用户注册/登录（用姓名+密码简单区分即可）
- 不需要邮件通知
- 不需要移动端 App（H5 够用）
- 不需要图表统计

## 八、发布要求
- 发布形式：H5 链接，方便微信群分享
- 不需要自定义域名
- 不需要支付功能
```

**如何把文档上传给秒哒：**
```
1. 把需求文档保存为 .md 或 .txt 文件
2. 打开秒哒创建页面
3. 在对话框旁边找到「上传文件」图标
4. 上传文档后，发送一句话：
   "请根据我上传的需求文档，生成这个应用"
5. 秒哒会解析文档，自动生成完整应用
```

---

### 13.6 分级实战：从 H5 到站点的完整制作流程

#### 13.6.1 Level 1：5 分钟做一个 H5 小工具

以「工作压力测评」为例，走完最短路径：

**Step 1：一句话描述（30 秒）**
```
做一个工作压力自测 H5，包含 8 道题（每题4个选项，分值1-4），
提交后根据总分显示压力等级（低/中/高/极高）和对应的减压建议。
界面治愈风格，绿色系配色，支持微信分享结果。
```

**Step 2：等待生成（3～5 分钟）**
- 看秒哒的生成过程，了解它在做什么
- 不要在生成过程中打断

**Step 3：快速测试（2 分钟）**
- 做一遍测评，验证题目是否合理
- 检查分数计算是否正确
- 检查结果展示是否清晰

**Step 4：微调（2 分钟）**
```
修改：题目内容我来改一下，请把 8 道题换成：
1. 你每周加班超过10小时吗？
2. 你经常在非工作时间处理工作消息吗？
3. 你对工作完成质量感到满意吗？
（以下5道题省略）
```

**Step 5：发布（1 分钟）**
点击「发布」→ 复制分享链接 → 发到微信群测试

**总耗时：约 15 分钟**

---

#### 13.6.2 Level 3：2 小时做一个多页面工具站

以「AI 写作助手站点」为例：

**阶段一：提交需求文档（10 分钟准备 + 5 分钟生成）**

先用 13.5.2 的模板写需求文档，包含：
- 首页（产品介绍 + 工具入口）
- 工具页 A：AI 标题生成（输入主题 → 输出 5 个标题选项）
- 工具页 B：AI 文章开头生成（输入标题 → 输出 3 种开头风格）
- 工具页 C：AI 内容扩写（输入关键词 → 输出 200 字段落）

上传文档，等待生成（约 5～10 分钟）。

**阶段二：分页验证（20 分钟）**

每个页面依次测试，记录问题：
```
问题清单（测试时记录）：
□ 首页导航点击跳转是否正确？
□ 工具页 A 生成的标题质量是否可接受？
□ 各页面在手机端显示是否正常？
□ 加载时间是否过长？
```

**阶段三：逐条修复（30 分钟）**

针对问题依次修复，每次只改一个地方：
```
问题1：手机端首页图片显示不完整
→ 对话框输入："首页顶部的图片在手机端显示被截断了，请适配手机端宽度"

问题2：工具页 A 的生成结果没有换行
→ 对话框输入："标题生成结果每条之间加一个分隔线，更清晰"

问题3：整体字体太小
→ 画布区域 → 全局样式 → 字体大小调整
```

**阶段四：上线前检查（10 分钟）**

```
发布前检查清单：
□ 所有页面跳转链接正常
□ 手机端各页面正常显示
□ AI 生成功能在手机上可用
□ 发布链接可以直接分享
□ 自定义域名是否需要绑定（可选）
```

**阶段五：发布（5 分钟）**

点击「发布」→ 配置分享信息（标题/描述/封面图）→ 复制链接 → 上线

**总耗时：约 1.5～2 小时**

---

### 13.7 发布、域名与分享配置

发布后，应用管理里有 5 个关键设置：

#### 基本设置
- 修改应用名称、描述、封面图
- 上传浏览器 Tab 图标（favicon）

#### 分享信息配置（影响微信分享效果）
```
分享标题：[简洁描述，建议20字以内]
分享描述：[吸引点击的一句话，建议30字以内]
分享封面：[建议尺寸 400×300，清晰吸引眼球]
```
> 这三项配置好，微信分享时的卡片才好看，**直接影响点击率**。

#### 域名管理（可选）
- 如果有自己的域名，可以绑定到秒哒应用
- 让应用链接从 `app-xxx.appmiaoda.com` 变成 `your-domain.com`
- 配置步骤：域名管理 → 添加域名 → 按提示在域名商处添加 CNAME 解析

#### 访问设置
- 可以设置应用的访问权限（公开/密码访问/付费访问）
- 付费访问：设置解锁价格，用户付费后才能使用核心功能

#### 发布到应用广场
- 上架后其他用户可发现你的应用
- 允许他人复制你的应用（可选开关）
- 获得第一个点赞奖励 50 积分

---

### 13.8 积分与定价

秒哒采用积分制：

| 使用方式 | 积分消耗 |
|--------|--------|
| 新用户注册 | 赠送积分（具体以官网为准） |
| 每日免费额度 | 100 积分/天 |
| 生成一个应用 | 约 15 积分 |
| 对话修改一次 | 约 3～5 积分 |

> 每天 100 积分可以生成约 6 个新应用，或进行约 20 次修改迭代。对于偶尔使用完全够用；重度使用或商业项目可购买付费套餐。

---

### 13.9 秒哒 vs 自己写代码：完整对比与选型指南

**按应用规模选择方案：**

```
H5 小工具（单页、无存储）
    → 首选秒哒，15 分钟搞定，无需写代码

带后端的轻应用（有数据存储、AI 功能）
    → 首选秒哒，30～60 分钟，代码版要 2 天

多页面站点（完整产品体验）
    → 秒哒优先（1～4 小时）；
      若对数据安全要求高，考虑代码版

大型商业应用（高并发、复杂业务、私有部署）
    → 代码版（本教程前十二章）
```

**全维度对比：**

| 对比维度 | 秒哒 | 写代码 |
|--------|------|------|
| 上手门槛 | 零门槛 | 需要编程基础 |
| 开发速度 | 分钟～小时 | 天～周 |
| 功能边界 | 覆盖 80% 常见场景 | 无上限 |
| 数据控制权 | 在秒哒平台 | 完全自有 |
| 性能上限 | 中等（平台共享资源）| 按需扩容 |
| 私有部署 | 不支持 | 支持 |
| 定制灵活性 | ★★★ | ★★★★★ |
| 维护成本 | 极低 | 需持续投入 |
| 适合阶段 | 验证想法、MVP、内部工具 | 正式产品、企业系统 |

**最佳实践路径：**

```
有新想法
    ↓
用秒哒做出 MVP（当天上线）
    ↓
给真实用户测试，验证核心假设
    ↓
需求确认 + 规模扩大 + 数据安全要求提升
    ↓
按照本教程前十二章，用代码重建完整版本
    ↓
两个版本并行，秒哒版继续迭代前端体验，
代码版负责核心业务逻辑
```

> **一句话总结：秒哒是最快的起点，代码是最强的终点。
> 不要用写代码的复杂度，去做一个秒哒就能搞定的事；
> 也不要用秒哒去硬撑一个需要完整工程体系的产品。**

---

---

## 附录：扩展方向参考

完成 MVP 后，可以沿以下方向继续扩展：

| 方向 | 功能点 | 难度 |
|-----|-------|-----|
| 存储功能 | 历史纪要记录（LocalStorage / 数据库） | ★★ |
| 分享功能 | 生成可分享的链接 | ★★★ |
| 模板系统 | 不同会议类型使用不同 Prompt 模板 | ★★ |
| 音频输入 | 上传音频文件自动转文字再生成纪要 | ★★★★ |
| 协作编辑 | 多人同时编辑纪要（WebSocket） | ★★★★★ |
| 语言支持 | 英文会议记录生成中文/英文纪要 | ★（改 Prompt 即可） |

---

**🎉 恭喜！你现在已经有了一个可以真实使用的 AI 应用。**

> 做完第一个之后，你会发现"做 AI 应用"的本质是：
> **用 Prompt 定义 AI 的行为** + **用工程手段处理输入输出** + **不断通过真实反馈改进**
>
> 和做传统软件的区别只是：把确定性的"函数"换成了不确定性的"模型"。
> 其余的产品思维、工程方法、迭代逻辑都是一样的。

---

---

**作者信息**

- 作者：UCL（社交名流 UCL）
- 联系方式：1030504893@qq.com
- 原创声明：本教程由 UCL 基于实战经验原创撰写，内容包括产品方法论、技术路径选择、Prompt 工程实践及秒哒平台应用全流程，版权归作者所有。
- 转载须知：转载请保留作者署名及联系方式，禁止用于商业用途（需事先获得书面授权）。

*文档版本：v1.2 | 更新日期：2026-05-02 | v1.1 新增：第十三章「百度秒哒零代码制作应用」| v1.2 新增：第十四～二十一章「进阶补充」| v1.3：按学习路径重排章节顺序*

*© 2026 UCL（社交名流 UCL）. All rights reserved. Contact: 1030504893@qq.com*
