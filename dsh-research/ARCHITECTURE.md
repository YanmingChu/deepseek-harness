# 🚀 DeepSeek Harness (`dsh`) 架构研究与开发全景指南

> 💡 **一句话核心定位**：DeepSeek 官方开源的 Agent 运行底座与执行中枢（Everything is a Plugin），以微内核架构驱动大语言模型在本地环境感知、决策与执行。
>
> 📌 **相关资源导航**：
> - 常用命令与上游同步手册：[OPERATIONS.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/OPERATIONS.md)
> - 交互式网页仪表盘：[html/architecture.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/architecture.html)
> - 研究工作区总目录：[README.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/README.md)

---

## 📑 目录导航

- [1. 架构哲学与核心设计](#-1-架构哲学与核心设计)
  - [1.1 核心设计三原则](#11-核心设计三原则)
  - [1.2 本地 Harness vs 云端大脑职责划分](#12-本地-harness-vs-云端大脑职责划分)
  - [1.3 数据隐私与执行边界](#13-数据隐私与执行边界)
- [2. 代码仓库与 Monorepo 全景](#-2-代码仓库与-monorepo-全景)
  - [2.1 Monorepo 机制说明](#21-monorepo-机制说明)
  - [2.2 仓库完整目录树](#22-仓库完整目录树)
  - [2.3 核心功能模块对照表](#23-核心功能模块对照表)
- [3. 智能体核心调度循环 (Agent Loop)](#-3-智能体核心调度循环-agent-loop)
  - [3.1 轮次生命周期时序 (Mermaid)](#31-轮次生命周期时序)
  - [3.2 阶段核心逻辑解析](#32-阶段核心逻辑解析)
- [4. 快速运行与常用命令](#-4-快速运行与常用命令)
- [5. 六大专题研究与二次开发路线](#-5-六大专题研究与二次开发路线)
- [6. 核心官方文档与扩展手册](#-6-核心官方文档与扩展手册)
- [7. 🎨 HTML 文档生成规范与样式设计准则 (AI 维护协议)](#-7--html-文档生成规范与样式设计准则-ai-维护协议)

---

## 📌 1. 架构哲学与核心设计

### 1.1 核心设计三原则

* 🧩 **一切皆插件（Everything is a Plugin）**
  * 基于 [Cordis](https://github.com/cordiverse/cordis) 微内核框架。
  * 系统**无特权内核**：从 LLM 适配器、文件系统、终端沙箱，到智能体调度循环（[`agent-loop`](file:///c:/DeepSeek/deepseek-harness/packages/core/agent-loop)）本身全都是插件，支持热插拔与配置覆写（Patch）。
* 📜 **模型可见即已记录（Model-visible ⟺ Logged）**
  * 采用 **事件溯源（Event Sourcing）** 机制。
  * 抵达模型的每一段输入、思维链、工具调用及结果均被严格持久化为 [`SessionEvent`](file:///c:/DeepSeek/deepseek-harness/packages/core/session)，确保 100% 可回溯、可重放与支持会话 Fork。
* ✂️ **能力解耦（Capability Seam）**
  * 严格遵循三层角色分离：**接口定义（Service Definition）** $\to$ **服务实现（Service Provider）** $\to$ **模型消费端（Consumer / Tools）**。
  * 切换底层运行环境（如将本地执行切换至 Docker/远程沙箱）无需重写工具或重训模型。

---

### 1.2 本地 Harness vs 云端大脑职责划分

> 🧠 **形象比喻**：DeepSeek 官方云端是大脑（负责思考推理），本地 `deepseek-harness` 是身体与双手（负责感知环境、写代码、运行命令与安全防护）。

```text
┌─────────────────────────────────────────────────────────────┐
│                    你的本地电脑 (Local Machine)             │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Web 浏览器 / 命令行 (UI & 交互)        │   │
│   └──────────────────────────┬──────────────────────────┘   │
│                              │                              │
│   ┌──────────────────────────▼──────────────────────────┐   │
│   │           deepseek-harness (dsh 本地运行引擎)       │   │
│   │                                                     │   │
│   │  • 提示词组装 (System Prompt + 工作区 AGENTS.md)    │   │
│   │  • 任务与事件调度 (Agent Loop 轮次控制)             │   │
│   │  • 本地文件读写 (FS / Diff / Search)                │   │
│   │  • 本地命令执行 (Shell / PowerShell / PTY 终端)     │   │
│   │  • 安全沙箱隔离与用户审批 (Sandbox & Permission)    │   │
│   │  • 会话历史持久化存储 (本地 SQLite / JSONL 日志)    │   │
│   └──────────────▲───────────────────────┬──────────────┘   │
└──────────────────┼───────────────────────┼──────────────────┘
                   │                       │
       1. 发送上下文 & 提示词              2. 返回思考过程 & 工具调用指令
          (HTTPS POST API 请求)            (流式返回 Tokens / Tool Call)
                   │                       │
┌──────────────────┴───────────────────────▼──────────────────┐
│              DeepSeek 官方云端后台 (Cloud LLM API)          │
│                                                             │
│  • 运行大语言模型 (如 DeepSeek-V3 / DeepSeek-R1 推理集群)  │
│  • 理解上下文与用户意图                                     │
│  • 进行深度推理与思考 (Chain-of-Thought / CoT)              │
│  • 决定下一步该调用什么工具、传入什么参数                   │
└─────────────────────────────────────────────────────────────┘
```

#### 📋 职责划分明细对照表

| 维度 | 💻 本地 `deepseek-harness` 负责 | ☁️ DeepSeek 云端后台负责 |
| :--- | :--- | :--- |
| **界面交互** | 提供 React Web 界面、侧边栏、对话流、代码 Diff 卡片与 CLI 终端 | 完全不负责（仅提供无状态 HTTP/JSON API） |
| **文件操作** | 本地代码扫描、文件读写、修改 Diff 计算、AST 分析 | 无法直接访问你的硬盘，仅接收请求中的文本片段 |
| **命令执行** | 在本地实际拉起 PowerShell / Bash 进程执行测试、编译与构建 | 不提供代码运行环境，不执行任何用户代码 |
| **思考推理** | 不具备推理能力，仅负责拼接上下文发起 API 请求 | **大模型计算集群**：理解意图、长链思考与工具决策 |
| **数据持久化** | 会话历史、断点恢复记录保存在本地硬盘（`~/.dsh`） | 不存储你的项目源码文件 |
| **安全审批** | 底层进程沙箱防护（Landlock/Bubblewrap），拦截危险指令 | 输出建议调用的工具，最终执行权在本地 Harness |

---

### 1.3 数据隐私与执行边界

1. 🔒 **代码零静默上传**：云端没有文件系统权限，只有 Agent 明确读取并放入当前轮次 Prompt 的代码片段才会随请求发送。
2. 💻 **命令 100% 本地执行**：模型只返回 JSON 动作意图（如 `execute_shell`），本地 Harness 收到后在你的电脑上本地启动进程执行。
3. 🌐 **支持完全离线运行**：利用 [`packages/llm/`](file:///c:/DeepSeek/deepseek-harness/packages/llm) 的适配机制，可一键切换为本地 **Ollama / vLLM** 离线大模型，实现 0 数据上云的完全私密运行。

---

## 📂 2. 代码仓库与 Monorepo 全景

### 2.1 Monorepo 机制说明
本项目基于 **pnpm workspaces** 构建单代码库（Monorepo），通过根目录 [`pnpm-workspace.yaml`](file:///c:/DeepSeek/deepseek-harness/pnpm-workspace.yaml) 统一管理 50+ 个子包。子包间通过 `workspace:^` 软链接实时联动，修改任意子包本地立即生效。

### 2.2 仓库完整目录树

```text
deepseek-harness/
├── 📱 应用端与业务包 (Product & Modules)
│   ├── apps/                    # 交付入口（apps/cli 命令行与 apps/web 网页前端）
│   ├── packages/                # 核心业务包集合（50+ 个插件与能力，如 core, llm, shell, fs 等）
│   ├── examples/                # 开箱即用的配置样例（Web-Cordis, ACP Agent, Headless 等）
│   ├── docs/                    # 官方中英双语架构文档与扩展指南
│   └── website/                 # 官方双语 VitePress 文档网站工程
│
├── ⚙️ 底层内核与系统级扩展 (Core & Native)
│   ├── vendor/                  # 本地内嵌修改版的 Cordis 微内核框架源码
│   ├── native/                  # 操作系统底层原生扩展（如 Linux Landlock 安全沙箱）
│   ├── python/                  # Python SDK 与单文件运行时的 Python 绑定
│   └── patches/                 # 对第三方依赖打的临时补丁（如 node-pty 的 Windows 修复）
│
├── 🛠️ 工程化脚本与自动化 (Scripts & CI)
│   ├── scripts/                 # 仓库级自动化脚本（50+ 个门禁检测、构建、打包脚本）
│   ├── .github/                 # GitHub Actions 持续集成 (CI/CD) 与自动化规则
│   └── .gitlab-ci.yml           # GitLab 自动化流水线配置
│
├── 🤖 智能体辅助与规范 (Agent Workflows)
│   ├── .agents/                 # AI 智能体上下文、架构决策笔记（notes/）与预设技能（skills/）
│   ├── AGENTS.md                # 整个仓库最核心的架构与开发规范总则（AI 助手修改本仓库时遵守）
│   └── CLAUDE.md                # 软链接指向 AGENTS.md
│
├── 🔬 专属研究与架构笔记 (Research Workspace)
│   └── dsh-research/            # 本项目专属研究目录 (README.md、ARCHITECTURE.md 与 HTML 仪表盘)
│
└── 📄 根目录全局配置文件 (Configs)
    ├── pnpm-workspace.yaml      # 声明当前 Monorepo 包含的所有工作区子包路径
    ├── package.json             # 根目录全局开发依赖与总控构建命令 (build, test, lint, hygiene 等)
    ├── pnpm-lock.yaml           # 全局依赖锁定文件
    ├── tsconfig.json            # TypeScript 根配置（配合 host 与 client 子配置）
    ├── vitest.config.ts         # Vitest 全局单元测试配置
    ├── lefthook.yml             # Git 提交流程本地钩子（提交前自动校验）
    └── .oxlintrc.json           # 超快速 Rust 代码规范检查器配置
```

---

### 2.3 核心功能模块对照表

| 模块分类 | 源码包路径 | 对应 `ctx` 属性 | 核心职责 |
|---|---|---|---|
| **会话与持久化** | [`packages/core/session`](file:///c:/DeepSeek/deepseek-harness/packages/core/session) | `ctx.sessions` | 仅追加日志、状态恢复与分支 Fork |
| **提示词组装** | [`packages/core/system-prompt`](file:///c:/DeepSeek/deepseek-harness/packages/core/system-prompt) | `ctx.systemPrompt` | 动态拼接 Prompt 片段与 Tool Schema |
| **工具流水线** | [`packages/core/tools`](file:///c:/DeepSeek/deepseek-harness/packages/core/tools) | `ctx.tools` | 工具注册、参数校验、前/后拦截执行流水线 |
| **驱动循环** | [`packages/core/agent-loop`](file:///c:/DeepSeek/deepseek-harness/packages/core/agent-loop) | `ctx.agentLoop` | 驱动单轮次/多步骤的 Agent 决策引擎 |
| **模型通信** | [`packages/llm/`](file:///c:/DeepSeek/deepseek-harness/packages/llm) | `ctx.llm` | 统一模型词汇表与各 Provider 适配器 |
| **环境与执行** | [`packages/fs/`](file:///c:/DeepSeek/deepseek-harness/packages/fs)<br>[`packages/shell/`](file:///c:/DeepSeek/deepseek-harness/packages/shell)<br>[`packages/terminal/`](file:///c:/DeepSeek/deepseek-harness/packages/terminal) | `ctx.fs`<br>`ctx.shell`<br>`ctx.terminals` | 跨平台文件读写、命令执行与持久化 PTY 终端 |
| **安全与沙箱** | [`packages/sandbox/`](file:///c:/DeepSeek/deepseek-harness/packages/sandbox)<br>[`packages/interaction/permission`](file:///c:/DeepSeek/deepseek-harness/packages/interaction/permission) | `ctx.sandbox` | 底层沙箱隔离与敏感操作权限审批 |
| **进阶协作** | [`packages/subagent/`](file:///c:/DeepSeek/deepseek-harness/packages/subagent)<br>[`packages/plan/`](file:///c:/DeepSeek/deepseek-harness/packages/plan)<br>[`packages/workflow/`](file:///c:/DeepSeek/deepseek-harness/packages/workflow) | `ctx.subagents`<br>`ctx.agentTeams`<br>`ctx.workflows` | 任务委派、多 Agent 团队协作与工作流编排 |

---

## 🔄 3. 智能体核心调度循环 (Agent Loop)

Agent 的单次交互为一个 **轮次（Turn）**，一个轮次内部由一个或多个 **步骤（Step）** 串联执行。

### 3.1 轮次生命周期时序

```mermaid
sequenceDiagram
    autonumber
    participant User as 用户 / Inbox
    participant Loop as Agent Loop (调度中枢)
    participant Prompt as System Prompt (组装器)
    participant LLM as 模型服务 (ctx.llm)
    participant Tools as 工具流水线 (ctx.tools)
    participant Log as 会话日志 (Session Log)

    Note over Loop: 🎬 turn/start (开启轮次)
    User->>Loop: 领取输入消息 (Inbox Claim)
    Loop->>Prompt: 组装 Prompt 片段 + Tool Schemas
    Loop->>Loop: 触发 waterfall: agent/pre-step (可改写/拦截)

    rect rgb(240, 248, 255)
        Note over Loop: 🔄 step/start (进入单步执行)
        Loop->>Log: 追加 user/message 持久化事件
        Loop->>Log: deriveMessages() 派生历史上下文
        Loop->>LLM: 发起请求 (agent/request -> llm/stream)
        LLM-->>Loop: 流式返回 assistant/chunk* -> assistant/message

        opt 存在工具调用 (Tool Calls)
            Loop->>Tools: tools/pre-execute (权限校验/审批)
            Tools->>Tools: tools/execute (执行底层文件/命令)
            Tools->>Tools: tools/post-execute (结果过滤)
            Tools->>Log: 追加 tool/result* 持久化事件
        end
        Note over Loop: 🏁 step/end (单步结束)
    end

    Note over Loop: 若工具产生后续需求，循环进入下一个 Step
    Loop->>Loop: 触发 serial: agent/turn-stopping
    Note over Loop: 🛑 turn/end (轮次关闭)
```

---

## 🚀 4. 快速运行与常用命令

> 📖 **完整运维与命令手册**：
> 详细指令清单、环境变量配置、全套 Git 上游同步与排障指南请直接参阅：
> - Markdown 版：[OPERATIONS.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/OPERATIONS.md)
> - 网页交互版：[html/operations.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/operations.html)

### 4.1 常用启动模式速查

```sh
# 1. 编译构建整个工作区
pnpm install && pnpm run build

# 2. 启动 Web 控制台 (默认 http://127.0.0.1:3080)
pnpm dsh web

# 3. 前端界面独立热更新开发 (推荐开发定制 UI 时使用)
cd apps/web && pnpm run dev

# 4. 命令行 Headless 模式 (无界面单次分析任务)
pnpm dsh --profile headless "请分析当前项目的架构特点"

# 5. 打印解析后的 Cordis 完整插件配置树 (诊断装配问题)
pnpm dsh --profile web --dump-config

# 6. 启动 ACP 自动化协议服务
pnpm run demo:acp
```

### 4.2 🔄 日常 3 步同步 DeepSeek 官方最新代码

```sh
# 1. 获取官方最新提交
git fetch upstream

# 2. 将官方 master 合并到当前研究分支
git merge upstream/master

# 3. 推送更新到个人云端 Fork 仓库
git push origin my-branch
```

### 4.3 工程与质量检查

```sh
pnpm run test            # Vitest 单元测试
pnpm run typecheck       # TypeScript 全局类型检查
pnpm run lint            # Oxlint 快速代码静态检查
pnpm run hygiene         # 依赖、工作区约束与规范检查
pnpm run doc-sync        # 文档链接与双语同步检查
```

---

## 🔬 5. 六大专题研究与二次开发路线

```text
                                 ┌─► ① Agent Loop 调度与上下文控制
                                 ├─► ② 扩展自定义 Tool 与 Skill
                                 ├─► ③ 多模型接入 (OpenAI/Claude/Ollama)
  DeepSeek Harness 核心研究路线 ──┼─► ④ Web UI 定制与跨平台桌面客户端
                                 ├─► ⑤ 多 Agent 协作与工作流编排
                                 └─► ⑥ 安全沙箱隔离与权限控制
```

---

### 专题 ①：Agent Loop 核心调度与上下文控制
* 🎯 **研究目标**：掌握 Agent 如何在多步骤中保持记忆、处理上下文溢出以及动态拦截 Prompt。
* 📂 **核心源码**：
  * [`packages/core/agent-loop`](file:///c:/DeepSeek/deepseek-harness/packages/core/agent-loop)
  * [`packages/core/system-prompt`](file:///c:/DeepSeek/deepseek-harness/packages/core/system-prompt)
  * [`packages/core/session`](file:///c:/DeepSeek/deepseek-harness/packages/core/session)
* 🔍 **核心机制**：
  1. `agent/pre-step` 瀑布流拦截器：如何在模型请求前动态改写或拒绝消息。
  2. `deriveMessages()`：如何从纯追加的 `SessionEvent` 日志高效投影出模型历史。
  3. 长对话压缩（[`packages/compaction`](file:///c:/DeepSeek/deepseek-harness/packages/compaction)）：当 Token 逼近上限时自动触发历史压缩。

---

### 专题 ②：扩展自定义 Tool 与 Skill 插件
* 🎯 **研究目标**：为 Agent 注入操作本地软件、自定义数据库、私有 API 的专属能力。
* 📂 **核心源码**：
  * [`packages/core/tools`](file:///c:/DeepSeek/deepseek-harness/packages/core/tools)
  * [`packages/skill`](file:///c:/DeepSeek/deepseek-harness/packages/skill)
  * [`docs/cookbook/adding-a-tool.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-a-tool.zh.md)
* 🔍 **核心机制**：
  1. 使用 `ctx.tools.register()` 声明 Schema 与执行函数。
  2. 执行流水线把关：`tools/pre-execute` $\to$ `tools/execute` $\to$ `tools/post-execute`。
  3. 声明前端渲染意图（`generic` / `terminal` / `diff`）。

---

### 专题 ③：多模型 LLM 适配器接入 (OpenAI / Claude / 本地 Ollama)
* 🎯 **研究目标**：摆脱单一模型依赖，接入第三方云端 API 或本地部署模型。
* 📂 **核心源码**：
  * [`packages/llm/`](file:///c:/DeepSeek/deepseek-harness/packages/llm)
  * [`docs/cookbook/adding-an-llm-adapter.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-an-llm-adapter.zh.md)
* 🔍 **核心机制**：
  1. 在 `ctx.llm` 注册统一适配器接口。
  2. 将各模型流式输出转换为 `assistant/chunk`、`tool/call` 等标准事件。
  3. 支持本地部署框架（vLLM / Ollama / SGLang）的私密流式推理。

---

### 专题 ④：Web UI 定制与跨平台桌面客户端开发
* 🎯 **研究目标**：改造现有的 Web 控制台或将其封装为 Electron/Tauri 桌面端。
* 📂 **核心源码**：
  * [`apps/web/`](file:///c:/DeepSeek/deepseek-harness/apps/web)
  * [`packages/api/`](file:///c:/DeepSeek/deepseek-harness/packages/api)
  * [`packages/sdk/`](file:///c:/DeepSeek/deepseek-harness/packages/sdk)
* 🔍 **核心机制**：
  1. 基于 Typert RPC 与 BFF 层的强类型 WebSocket 通信。
  2. 自定义 UI 卡片（代码对比、命令输出、审批弹窗）。
  3. 响应式布局与深浅色主题。

---

### 专题 ⑤：多 Agent 团队协作与工作流编排
* 🎯 **研究目标**：实现复杂软件工程任务的主从协同（Planner + Coder + Reviewer）。
* 📂 **核心源码**：
  * [`packages/subagent/`](file:///c:/DeepSeek/deepseek-harness/packages/subagent)
  * [`packages/workflow/`](file:///c:/DeepSeek/deepseek-harness/packages/workflow)
  * [`packages/plan/`](file:///c:/DeepSeek/deepseek-harness/packages/plan)
* 🔍 **核心机制**：
  1. 子 Agent 派生与上下文隔离。
  2. 基于 Worker Thread 的并发工作流。
  3. 团队状态感知与进度汇总。

---

### 专题 ⑥：安全沙箱隔离与权限控制
* 🎯 **研究目标**：保障 Agent 在本地环境执行操作的绝对安全性，防止越权。
* 📂 **核心源码**：
  * [`packages/sandbox/`](file:///c:/DeepSeek/deepseek-harness/packages/sandbox)
  * [`packages/interaction/permission`](file:///c:/DeepSeek/deepseek-harness/packages/interaction/permission)
  * [`packages/guard/`](file:///c:/DeepSeek/deepseek-harness/packages/guard)
* 🔍 **核心机制**：
  1. 细粒度工具执行审批拦截。
  2. 原生沙箱防护与路径隔离。
  3. 死循环防护与调用超时熔断。

---

## 📚 6. 核心官方文档与扩展手册

* 📘 **架构原理深度**：[`docs/architecture.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/architecture.zh.md)
* 🧩 **Cordis 插件框架入门**：[`docs/cordis-primer.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/cordis-primer.zh.md)
* 🔌 **能力 Seams 完整图解**：[`docs/capability-seams.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/capability-seams.zh.md)
* 📖 **扩展开发实操手册 (Cookbook)**：[`docs/cookbook/extension-cookbook.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/extension-cookbook.zh.md)
  * [添加新工具 (Tool)](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-a-tool.zh.md)
  * [添加 LLM 适配器](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-an-llm-adapter.zh.md)
  * [添加对话流渲染节点 (Conversation Node)](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-a-conversation-node.zh.md)
  * [添加新 Package](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-a-package.zh.md)

---

## 🎨 7. HTML 文档生成规范与样式设计准则 (AI 维护协议)

> ⚠️ **AI 助手必读准则**：后续当用户修改或扩充 `ARCHITECTURE.md` 内容，并要求同步更新 `html/architecture.html` 时，**必须严格遵循以下样式与组件规范，严禁破坏已确立的设计语言与主题风格！**

### 7.1 视觉设计系统 (Design Tokens)

```css
/* 必须保持的核心 CSS 变量 (暗黑科技极客风格) */
:root {
  --bg-primary: #0b0f19;                         /* 背景底色 */
  --bg-secondary: #111827;                       /* 侧边栏与卡片底色 */
  --bg-card: rgba(17, 24, 39, 0.75);             /* 毛玻璃半透明卡片 */
  --bg-card-hover: rgba(31, 41, 55, 0.85);        /* 悬浮卡片色 */
  --border-color: rgba(255, 255, 255, 0.08);     /* 极细微边框 */
  --border-focus: rgba(56, 189, 248, 0.5);       /* 聚焦点亮边框 */

  --text-primary: #f3f4f6;                       /* 主文字白色 */
  --text-secondary: #9ca3af;                     /* 次级文字灰色 */
  --text-muted: #6b7280;                         /* 弱化标注文字 */

  --accent-blue: #0ea5e9;                        /* 品牌主色 蓝 */
  --accent-cyan: #38bdf8;                        /* 强调色 亮青 */
  --accent-purple: #818cf8;                      /* 强调色 紫 */
  --accent-emerald: #34d399;                     /* 强调色 翠绿 */
  --accent-amber: #fbbf24;                       /* 强调色 琥珀金 */

  --gradient-brand: linear-gradient(135deg, #0ea5e9 0%, #6366f1 50%, #a855f7 100%);
  --glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
  --sidebar-width: 280px;
}
```

### 7.2 核心组件与布局映射规则

1. **页面布局骨架**：
   - 固定左侧 280px 侧边栏（`aside.sidebar`），包含品牌徽标、快速目录与主题切换按钮。
   - 右侧为主内容区域（`main.content`），顶部为渐变玻璃态大横幅（`header.hero-banner`）。
   - 背景配置两个固定高斯模糊光晕（`ambient-glow` 与 `ambient-glow-2`），营造层次感。
2. **章节卡片与网格（Grid System）**：
   - 核心原则：采用 `.grid-3` 或 `.grid-2` 网格布局。
   - 比较卡片：采用 `.comparison-container` 左右分栏（本地 `.comp-card.local` vs 云端 `.comp-card.cloud`）。
3. **六大专题选项卡（Tab Switcher）**：
   - 专题模块必须使用选项卡结构（`.tab-navigation` + `.tab-btn` + `.tab-pane`），严禁平铺展开成冗长文本。
   - JavaScript 统一调用 `switchTab(event, 'tab-id')` 驱动切换。
4. **Mermaid 图表渲染**：
   - 必须通过 `<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>` 驱动。
   - 图表必须包裹在 `<div class="mermaid-box"><pre class="mermaid">...</pre></div>` 中。
5. **代码块与一键复制**：
   - 所有命令必须使用 `<div class="code-box"><button class="copy-btn" onclick="copyText('...')">复制</button><code>...</code></div>` 结构。

### 7.3 Markdown 到 HTML 的同步步骤
当用户在 `ARCHITECTURE.md` 中增加或修改内容时，AI 需按如下流程增量更新 `html/architecture.html`：
1. **对比差异**：提取 Markdown 中的新增段落、表格或专题分析点。
2. **组件套用**：将新增内容套入上述对应的 HTML 卡片模板（保留所有 class 属性与样式类）。
3. **保持链接**：所有代码链接使用 `file:///c:/DeepSeek/deepseek-harness/...` 标准格式。
4. **输出验证**：确保 JavaScript 脚本（主题切换、Tabs 切换、复制、ScrollSpy）完整无误。
