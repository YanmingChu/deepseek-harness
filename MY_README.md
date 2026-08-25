# DeepSeek Harness (dsh) 个人学习与速查笔记

本文档用于整理和记录 **DeepSeek Harness (`dsh`)** 的核心概念、架构设计、技术模块、二次开发定制及常用操作，方便个人快速查阅与后续探索。

---

## 📌 1. 项目核心定位

* **项目名称**：DeepSeek Harness（简称 `dsh`）
* **开发团队**：[DeepSeek AI（深度求索）](https://deepseek.com)
* **开源协议**：[MIT 许可证](file:///c:/DeepSeek/deepseek-harness/LICENSE)（开源且支持任意修改、定制与二次分发）
* **定位**：面向大语言模型（LLM）的**开源 Agent 运行底座与执行框架（Agent Harness）**。
* **目标**：为 LLM 提供全功能、高扩展、强确定性、安全隔离的通用智能体执行环境与调度平台。

---

## 🌟 2. 核心架构与设计哲学

### 2.1 “一切皆插件”（Everything is a Plugin）
* 基于 **[Cordis](https://github.com/cordiverse/cordis)** 微内核设计。
* 系统内**不存在特权内核**：从 LLM 适配器、工具注册表、文件系统、持久化终端、沙箱隔离，到 Agent 的调度循环（[agent-loop](file:///c:/DeepSeek/deepseek-harness/packages/core/agent-loop)）本身，全部作为插件挂载在上下文中，均可热插拔或通过配置替换。

### 2.2 “模型可见即已记录”（Model-visible ⟺ Logged）
* 采用事件溯源（Event Sourcing）设计模式。
* 抵达模型请求的所有上下文（输入、思维链、工具调用及返回结果）均持久化记录为 `SessionEvent`。
* 确保 Agent 行为具备完全的可追溯性、会话断点恢复、Fork 分支派生与历史回放能力。

### 2.3 能力解耦（Capability Seam）
* 严格遵循三层角色分离：
  * **接口定义（Service Definition）**：声明抽象能力接口。
  * **服务实现（Service Provider）**：具体后端实现（如本地进程、E2B 远程沙箱、Docker 等）。
  * **模型消费端（Consumer）**：面向模型调用的工具（Tools）。
* 切换底层运行环境无需重构上层工具或重新训练/提示模型。

---

## 📂 3. 核心技术模块与代码结构

代码主要存放在 [`packages/`](file:///c:/DeepSeek/deepseek-harness/packages) 目录下：

| 模块类别 | 对应包路径 | 主要职责与功能说明 |
|---|---|---|
| **核心调度 (Core)** | [`packages/core/`](file:///c:/DeepSeek/deepseek-harness/packages/core) | 会话管理 (`session`)、提示词组装 (`system-prompt`)、工具流水线 (`tools`) 与核心驱动循环 (`agent-loop`) |
| **模型适配 (LLM)** | [`packages/llm/`](file:///c:/DeepSeek/deepseek-harness/packages/llm) | 统一模型流式通信与各类 LLM 提供方适配器 |
| **环境与执行 (Execution)** | [`packages/fs/`](file:///c:/DeepSeek/deepseek-harness/packages/fs)<br>[`packages/shell/`](file:///c:/DeepSeek/deepseek-harness/packages/shell)<br>[`packages/terminal/`](file:///c:/DeepSeek/deepseek-harness/packages/terminal)<br>[`packages/code-runtime/`](file:///c:/DeepSeek/deepseek-harness/packages/code-runtime) | 文件系统操作、本地/PowerShell/Bash 命令执行、持久化 PTY 终端会话与独立代码运行环境 |
| **安全沙箱 (Sandbox)** | [`packages/sandbox/`](file:///c:/DeepSeek/deepseek-harness/packages/sandbox) | Bubblewrap、Landlock、Seatbelt 等底层安全沙箱技术隔离 |
| **代码服务 (LSP)** | [`packages/lsp/`](file:///c:/DeepSeek/deepseek-harness/packages/lsp) | 语言服务器协议（LSP）集成，支持代码跳转与定义诊断 |
| **进阶协作 (Agentic)** | [`packages/plan/`](file:///c:/DeepSeek/deepseek-harness/packages/plan)<br>[`packages/subagent/`](file:///c:/DeepSeek/deepseek-harness/packages/subagent)<br>[`packages/workflow/`](file:///c:/DeepSeek/deepseek-harness/packages/workflow)<br>[`packages/skill/`](file:///c:/DeepSeek/deepseek-harness/packages/skill) | Plan 计划协作模式、多 Agent 协作委派（Subagents/Teams）、工作流引擎与动态技能扩展 |
| **网络与搜索 (Web)** | [`packages/web/`](file:///c:/DeepSeek/deepseek-harness/packages/web) | 网页搜索、网页内容抓取与解析工具 |
| **交互与集成 (Interfaces)** | [`packages/host/`](file:///c:/DeepSeek/deepseek-harness/packages/host)<br>[`packages/client/`](file:///c:/DeepSeek/deepseek-harness/packages/client)<br>[`packages/acp/`](file:///c:/DeepSeek/deepseek-harness/packages/acp)<br>[`packages/sdk/`](file:///c:/DeepSeek/deepseek-harness/packages/sdk) | Web UI 客户端与服务端网关、ACP 自动化协议及 TypeScript/Python SDK |

---

## 💻 4. 源码修改、UI 定制与桌面端开发

当前仓库包含完整的源码，你可以根据个人喜好进行全方位的二次开发与定制：

### 4.1 前端 UI 架构分布
* **Web 应用入口**：[`apps/web/`](file:///c:/DeepSeek/deepseek-harness/apps/web)（基于 React 18 + Vite 构建）
* **UI 模块与插件集**：[`packages/client/`](file:///c:/DeepSeek/deepseek-harness/packages/client)
  * 侧边栏布局：[`packages/client/ui-sidebar`](file:///c:/DeepSeek/deepseek-harness/packages/client/ui-sidebar)
  * 对话流呈现：[`packages/client/ui-conversation`](file:///c:/DeepSeek/deepseek-harness/packages/client/ui-conversation)
  * 工具交互卡片：[`packages/client/ui-tool`](file:///c:/DeepSeek/deepseek-harness/packages/client/ui-tool)
  * 设置面板：[`packages/client/ui-settings`](file:///c:/DeepSeek/deepseek-harness/packages/client/ui-settings)
  * 界面主题与配色：[`packages/client/ui-theme`](file:///c:/DeepSeek/deepseek-harness/packages/client/ui-theme)
* **后端通信网关**：[`packages/host/`](file:///c:/DeepSeek/deepseek-harness/packages/host) 与 [`packages/api/`](file:///c:/DeepSeek/deepseek-harness/packages/api)

### 4.2 桌面客户端化（Desktop App）方案
* 目前默认启动方式是通过浏览器访问本地 Web 服务。
* **打包为桌面应用**：由于前端是标准的 React + Vite 架构，后端为 Node.js 服务，你可以通过引入 **Tauri** 或 **Electron** 轻松将当前项目封装为独立的桌面应用程序（`.exe` / `.dmg`），并与原生桌面窗口深度集成。

### 4.3 常见定制方向速查
| 定制目标 | 涉及目录 / 文件 | 说明 |
|---|---|---|
| **改动 UI 视觉与交互** | [`packages/client/`](file:///c:/DeepSeek/deepseek-harness/packages/client)<br>[`apps/web/`](file:///c:/DeepSeek/deepseek-harness/apps/web) | 修改聊天气泡、侧边栏、状态栏、配色与交互动效 |
| **增加自定义执行工具** | [`packages/fs/`](file:///c:/DeepSeek/deepseek-harness/packages/fs)<br>[`packages/shell/`](file:///c:/DeepSeek/deepseek-harness/packages/shell) | 为 Agent 注入操作本地软件、自定义脚本或私有 API 的能力 |
| **接入其它大模型** | [`packages/llm/`](file:///c:/DeepSeek/deepseek-harness/packages/llm) | 编写适配器接入 OpenAI、Claude、Ollama 或本地自建模型 |
| **定制 Prompt 与思考循环** | [`packages/core/agent-loop/`](file:///c:/DeepSeek/deepseek-harness/packages/core/agent-loop)<br>[`packages/core/system-prompt/`](file:///c:/DeepSeek/deepseek-harness/packages/core/system-prompt) | 修改系统提示词策略、Agent 决策和多步反思流程 |

---

## 🚀 5. 常用运行与开发命令

### 5.1 启动与使用
```sh
# 方式 1: 直接使用 npx 启动 Web UI (默认端口 3080)
npx @deepseek-ai/dsh web

# 方式 2: 从本地源码运行 Web 界面
pnpm install
pnpm run build
pnpm dsh web

# 方式 3: 前端 UI 独立热更新开发调试 (修改前端代码时推荐)
cd apps/web
pnpm run dev

# 方式 4: 命令行无界面 (Headless) 模式运行单次任务 (需设置 DEEPSEEK_API_KEY)
pnpm dsh --profile headless "请帮我分析当前目录下的项目结构"

# 方式 5: 启动 ACP 自动化服务
pnpm run demo:acp
```

### 5.2 常用开发与质量检查命令
```sh
pnpm run test            # 运行 Vitest 单元测试
pnpm run typecheck       # TypeScript 全局类型检查
pnpm run lint            # 代码规范与静态检查
pnpm run hygiene         # 依赖与工作区规范检查
pnpm run doc-sync        # 文档同步检查
```

---

## 📚 6. 扩展与开发参考指引

* **架构深入**：[`docs/architecture.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/architecture.zh.md)
* **Cordis 插件机制教程**：[`docs/cordis-primer.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/cordis-primer.zh.md)
* **实操扩展手册 (Cookbook)**：[`docs/cookbook/extension-cookbook.zh.md`](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/extension-cookbook.zh.md)
  * [添加新工具 (Tool)](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-a-tool.zh.md)
  * [添加 LLM 模型适配器](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-an-llm-adapter.zh.md)
  * [添加新 Package](file:///c:/DeepSeek/deepseek-harness/docs/cookbook/adding-a-package.zh.md)

---

## 📝 7. 个人学习与测试随手记

> 💡 *可以在这里记录你个人在体验、调试或定制过程中的想法与待办事项：*

- [ ] 配置 `.env` 环境变量中的 `DEEPSEEK_API_KEY`
- [ ] 体验 `pnpm dsh web` 本地启动与 Web 控制台交互
- [ ] 体验 `cd apps/web && pnpm run dev` 热更新前端修改
- [ ] 探索 `packages/client/ui-theme` 与 `packages/client/ui-conversation` UI 视觉定制
- [ ] 探索 `packages/sandbox` 安全沙箱隔离机制
- [ ] 尝试编写一个自定义的 Cordis 扩展工具或技能插件
