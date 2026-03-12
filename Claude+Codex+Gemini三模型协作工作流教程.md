# Claude + Codex + Gemini 三模型协作工作流教程

> 三英战吕布 —— 以 Claude Code 为主控，通过 MCP 协议联合 Gemini 和 Codex，各取所长，协作完成复杂开发任务。

---

## 一、工作流简介

### 1.1 为什么要三模型协作？

单一模型在处理复杂项目时各有局限：

- **Claude**：编码能力强，但上下文有限，成本较高
- **Gemini**：超大上下文窗口，擅长规划、UI/UX 设计和视觉分析，免费额度充足
- **Codex**：OpenAI 出品，擅长代码审核和技术实现建议，成本可控

三者通过 MCP（Model Context Protocol）协议联合，优势互补：

| 角色 | 职责 | 优势 |
|------|------|------|
| **Claude Code** | 主控协调、综合决策、编写代码 | 编码质量高，工具链成熟 |
| **Gemini** | 规划、UI/UX 设计、视觉分析、前端建议 | 超大上下文、视觉能力强、免费 |
| **Codex** | 代码审核、技术实现细节、架构建议 | 代码理解深入、成本低 |

### 1.2 工作流程图

```
用户提出需求
    │
    ▼
┌─────────────────────────┐
│      Claude Code        │  ◄── 主控
│   (分析需求，分配任务)    │
└────┬──────────┬─────────┘
     │          │
     ▼          ▼
┌─────────┐ ┌─────────┐
│ Gemini  │ │  Codex  │
│  (MCP)  │ │  (MCP)  │
│         │ │         │
│ UI/UX   │ │ 代码审核 │
│ 规划    │ │ 技术实现 │
│ 视觉    │ │ 架构建议 │
└────┬────┘ └────┬────┘
     │           │
     ▼           ▼
┌─────────────────────────┐
│      Claude Code        │
│  (综合意见，编写代码/计划) │
└─────────────────────────┘
     │
     ▼
  输出结果（代码 / Plan.md / 完整项目）
```

---

## 二、环境准备

### 2.1 前置条件

- 已安装 **Claude Code**（Anthropic 官方 CLI）
- 已安装 **Node.js**（v18+）和 **npm**
- 需要以下 API Key：
  - **Gemini API Key**：从 [Google AI Studio](https://aistudio.google.com/apikey) 免费获取
  - **OpenAI API Key**：从 [OpenAI Platform](https://platform.openai.com/api-keys) 获取（Codex 使用）

### 2.2 确认 Claude Code 可用

```bash
claude --version
```

如果还没安装 Claude Code：

```bash
npm install -g @anthropic-ai/claude-code
```

---

## 三、配置 Gemini MCP Server

### 3.1 一键安装

```bash
claude mcp add gemini -s user -- env GEMINI_API_KEY=你的Gemini_API_Key npx -y @rlabs-inc/gemini-mcp
```

> `-s user` 表示全局生效（所有项目可用），也可以改为 `-s local` 只在当前项目生效。

### 3.2 高级配置（可选）

指定模型和工具集：

```bash
claude mcp add gemini -s user -- env \
  GEMINI_API_KEY=你的KEY \
  GEMINI_PRO_MODEL=gemini-2.5-pro \
  GEMINI_FLASH_MODEL=gemini-2.5-flash \
  GEMINI_TOOL_PRESET=full \
  VERBOSE=false \
  npx -y @rlabs-inc/gemini-mcp
```

#### 可用环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `GEMINI_API_KEY` | **必填**，Gemini API 密钥 | - |
| `GEMINI_PRO_MODEL` | Pro 模型 ID | `gemini-3-pro-preview` |
| `GEMINI_FLASH_MODEL` | Flash 模型 ID | `gemini-3-flash-preview` |
| `GEMINI_IMAGE_MODEL` | 图像生成模型 | `gemini-3-pro-image-preview` |
| `GEMINI_TOOL_PRESET` | 工具预设：`minimal`/`text`/`image`/`research`/`full` | - |
| `VERBOSE` | 详细日志 | `false` |

---

## 四、配置 Codex MCP Server

### 4.1 安装 Codex CLI

```bash
npm install -g @openai/codex
```

### 4.2 配置 OpenAI API Key

```bash
codex login --api-key "你的OpenAI_API_Key"
```

### 4.3 注册 MCP Server

```bash
claude mcp add codex-cli -- npx -y codex-mcp-server
```

---

## 五、验证安装

### 5.1 检查 MCP 状态

启动 Claude Code，输入：

```
/mcp
```

你应该能看到两个 MCP 服务器都处于活跃状态：

```
MCP Servers:
  ✓ gemini        (active)
  ✓ codex-cli     (active)
```

### 5.2 简单测试

在 Claude Code 中输入：

```
请用 Gemini 分析一下当前项目的目录结构，给出改进建议。
```

如果 Gemini MCP 正常工作，你会看到 Claude 调用 `gemini - gemini (MCP)` 工具并返回结果。

---

## 六、实际使用方法

### 6.1 自动协作模式

直接描述需求，Claude 会自动判断何时调用 Gemini 和 Codex：

```
帮我开发一个 Vue 3 + TypeScript 的待办事项应用，要求有良好的 UI 设计和完整的状态管理。
```

Claude 会自动：
1. 调用 Gemini 进行 UI/UX 设计建议
2. 调用 Codex 进行技术架构审核
3. 综合两者意见编写代码

### 6.2 显式引导模式（推荐）

你可以在 prompt 中明确指定各模型的分工，效果更好：

```
我需要开发一个阴阳师手游自动化脚本。请按以下流程进行：

1. 先让 Gemini 从 UI/UX 和开发流程角度分析：
   - 开发阶段划分（Phase 1~5）
   - 界面设计建议
   - 原型演示方案

2. 再让 Codex 从技术实现角度分析：
   - 核心类的实例化和依赖注入
   - TypeScript 技术选型
   - JS Bridge 通信方案
   - Hello World 验证代码

3. 最后综合两者意见，编写完整的开发计划到 Plan.md
```

### 6.3 代码审核模式

写完代码后，让 Codex 审核：

```
我刚写完 src/services/ 下的所有服务类，请让 Codex 审核代码质量，
然后让 Gemini 检查 UI 组件的设计是否合理。
```

### 6.4 分阶段开发模式

```
我们现在进入 Phase 2 的开发。请：
- 让 Gemini 确认 Phase 2 的 UI 需求和交互流程
- 让 Codex 审核 Phase 1 的代码，确认可以安全进入下一阶段
- 然后开始编写 Phase 2 的代码
```

---

## 七、进阶技巧

### 7.1 在 CLAUDE.md 中预设协作规则

在项目根目录创建 `CLAUDE.md`，写入协作规则让 Claude 每次自动遵循：

```markdown
## 开发规范

### 多模型协作规则
- 涉及 UI/UX 设计时，必须先咨询 Gemini
- 完成核心模块后，必须让 Codex 进行代码审核
- 编写开发计划时，需综合 Gemini（规划）和 Codex（技术）两方意见
- 每个 Phase 完成后，让 Codex 审核再进入下一阶段

### 分工定义
- Gemini：规划、UI 设计、视觉分析、前端架构
- Codex：代码审核、技术实现、架构优化、测试建议
- Claude：协调、综合决策、代码编写、文件操作
```

### 7.2 降低成本的方案

帖子作者提到几种降低成本的方式：

| 方案 | 说明 |
|------|------|
| Gemini 免费额度 | Google AI Studio 提供免费 API 调用额度 |
| Codex 低成本 | OpenAI Codex 的 API 价格相对较低 |
| Claude 中转 | 使用 iflow 等反代服务降低 Claude 调用成本 |
| 公益站 | 社区提供的免费 API 服务（稳定性不保证） |

### 7.3 替代方案

如果不想为 Claude 付费，可以考虑替换主控模型，但效果会打折扣：

- `minimax/glm/kimi-k2-thinking` 等国产模型可以作为替代
- 但帖子作者实测，这些模型与 Claude 之间**差距悬殊，效果差很多**

---

## 八、常见问题

### Q1: MCP 服务器连接失败？

```bash
# 检查 MCP 状态
claude mcp list

# 移除后重新添加
claude mcp remove gemini
claude mcp add gemini -s user -- env GEMINI_API_KEY=你的KEY npx -y @rlabs-inc/gemini-mcp
```

### Q2: Gemini 返回超时？

Gemini Pro 模型响应可能较慢，可以切换为 Flash 模型：

```bash
claude mcp remove gemini
claude mcp add gemini -s user -- env \
  GEMINI_API_KEY=你的KEY \
  GEMINI_PRO_MODEL=gemini-2.5-flash \
  npx -y @rlabs-inc/gemini-mcp
```

### Q3: Codex 报错 "not authenticated"？

重新登录：

```bash
codex login --api-key "你的OpenAI_API_Key"
```

### Q4: Windows 上环境变量设置问题？

Windows 的 `env` 命令可能不可用，改用 `cross-env`：

```bash
npm install -g cross-env
claude mcp add gemini -s user -- cross-env GEMINI_API_KEY=你的KEY npx -y @rlabs-inc/gemini-mcp
```

或者直接编辑配置文件 `~/.claude/settings.json`，手动添加 MCP 配置。

### Q5: 如何查看/编辑 MCP 配置文件？

Claude Code 的 MCP 配置存储在：

- **全局配置**：`~/.claude/settings.json`
- **项目配置**：`.claude/settings.local.json`

---

## 九、参考资源

- [RLabs-Inc/gemini-mcp](https://github.com/RLabs-Inc/gemini-mcp) — Gemini MCP Server
- [tuannvm/codex-mcp-server](https://github.com/tuannvm/codex-mcp-server) — Codex MCP Server
- [jamubc/gemini-mcp-tool](https://github.com/jamubc/gemini-mcp-tool) — 另一个 Gemini MCP 实现
- [Claude Code MCP 完整指南](https://ctok.ai/en/claude-code-mcp-server-guide)
- [原帖地址](https://www.deepflood.com/post-35309-1) — DeepFlood 论坛

---

*本教程基于 DeepFlood 论坛 KateTseng 的帖子"三英战吕布，Claude+Codex+Gemini联手"复原整理。*
