# DeepSeek Harness 常用命令与上游同步操作手册

> 💡 本手册用于汇总 DeepSeek Harness (`dsh`) 的日常高频运行命令、上游官方代码同步工作流、工程门禁与多端协作操作指南。
> 架构原理与核心设计请参考：[MY_README.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/MY_README.md)
> 网页版交互式命令面板请打开：[OPERATIONS.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/OPERATIONS.html)

---

## 目录
- [1. 环境初始化与凭据配置](#1-环境初始化与凭据配置)
- [2. 核心启动与运行模式](#2-核心启动与运行模式)
- [3. 🔄 上游代码同步与 Git 多端协作流程](#3--上游代码同步与-git-多端协作流程)
- [4. 工程质量检测与门禁检查](#4-工程质量检测与门禁检查)
- [5. 调试诊断与排错指南](#5-调试诊断与排错指南)

---

## 1. 环境初始化与凭据配置

### 1.1 前置运行环境要求
* **Node.js**：`^22.19 || >=24`（推荐 LTS 22.x 或 24.x）
* **包管理器**：`pnpm`（>= 9.0）
* **网络代理**：国内环境推荐开启本地代理（默认 SOCKS5/HTTP 端口 `127.0.0.1:7890`）

### 1.2 依赖安装与全量构建

```sh
# 1. 安装整个 monorepo 工作区依赖
pnpm install

# 2. 全量编译（tsc 生成类型定义，tsdown 打包运行时）
pnpm run build

# 3. 清理构建产物与临时缓存（排查幽灵编译问题时使用）
pnpm run clean
```

### 1.3 环境变量与 API Key 配置
在项目根目录创建或编辑 `.env` 文件：

```ini
# DeepSeek 官方 API Key (必需，用于模型调用)
DEEPSEEK_API_KEY="sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# 可选：自定义模型 Base URL (如使用本地反代或中转)
# DEEPSEEK_BASE_URL="https://api.deepseek.com/v1"

# 可选：第三方搜索服务 API Key (用于 Web 搜索插件)
# PERPLEXITY_API_KEY="pplx-xxxxxxxxxxxxxxxx"
# EXA_API_KEY="xxxxxxxxxxxxxxxx"
```

---

## 2. 核心启动与运行模式

### 2.1 常用启动命令汇总

| 启动模式 | 命令 | 说明 | 适用场景 |
| :--- | :--- | :--- | :--- |
| **Web 控制台** | `pnpm dsh web` | 启动完整的 Agent Web 界面（默认 `http://127.0.0.1:3080`） | 交互式日常使用与会话测试 |
| **前端独立热重载** | `cd apps/web && pnpm run dev` | 单独启动前端 Vite 开发服务器，支持毫秒级 UI 热更新 | 定制或修改 Web 界面 |
| **Headless 命令行** | `pnpm dsh --profile headless "任务描述"` | 无 UI 快速单次执行任务并直接在控制台输出结果 | 脚本自动化、批量测试 |
| **ACP 协议服务端** | `pnpm run demo:acp` | 启动 Agent Client Protocol 服务端 | 外部编辑器或自动化宿主对接 |
| **自修改演示** | `pnpm run demo:cordis` | 演示 Agent 运行时动态检查并加载自身插件 | 研究动态插件热挂载机制 |
| **配置树诊断** | `pnpm dsh --profile web --dump-config` | 打印解析后的 Cordis 完整插件配置树 | 诊断插件装配与配置覆盖问题 |

### 2.2 启动示例

```sh
# 示例 1：无界面分析当前仓库架构
pnpm dsh --profile headless "分析 packages/core/ 目录的设计模式"

# 示例 2：导出完整插件依赖配置供调试
pnpm dsh --profile web --dump-config > cordis-dump.json
```

---

## 3. 🔄 上游代码同步与 Git 多端协作流程

### 3.1 仓库拓扑关系

```text
┌──────────────────────────────────────────────────────────┐
│  DeepSeek 官方主仓库 (Upstream)                           │
│  https://github.com/deepseek-ai/deepseek-harness          │
└────────────────────────────┬─────────────────────────────┘
                             │
                      git fetch upstream
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│  本地工作区分支 (Local Branch)                             │
│  c:\DeepSeek\deepseek-harness (分支: my-branch)          │
│  包含研究笔记: dsh-research/                             │
└────────────────────────────┬─────────────────────────────┘
                             │
                      git push origin
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│  你的个人 GitHub Fork 仓库 (Origin)                       │
│  git@github.com:YanmingChu/deepseek-harness.git           │
└──────────────────────────────────────────────────────────┘
```

---

### 3.2 日常 3 步同步 DeepSeek 官方最新代码

当你需要获取官方最新的 bug 修复、架构升级或新插件时，在本地分支执行：

```sh
# 第 1 步：拉取 DeepSeek 官方主仓的最新提交
git fetch upstream

# 第 2 步：将官方 master 分支的更新合并到你当前的研究分支
git merge upstream/master

# 第 3 步：将合并后的最新代码推送到你的个人 GitHub 仓库
git push origin my-branch
```

> 💡 **解决合并冲突提示**：
> 我们的研究笔记均保存在独立的 `dsh-research/` 目录下，因此日常合并官方代码**几乎不会产生任何文件冲突**。

---

### 3.3 在新电脑上同步并继续研究

换到另一台新电脑时，按以下步骤快速拉取环境：

```sh
# 1. 克隆你的个人 Fork 仓库
git clone git@github.com:YanmingChu/deepseek-harness.git
cd deepseek-harness

# 2. 切换到你的专属研究分支
git checkout my-branch

# 3. 添加官方上游源 (用于后续拉取更新)
git remote add upstream https://github.com/deepseek-ai/deepseek-harness

# 4. 安装依赖并构建
pnpm install
pnpm run build
```

---

### 3.4 日常保存与提交研究成果

```sh
# 1. 查看当前修改状态
git status

# 2. 暂存改动
git add .

# 3. 规范化提交信息 (遵循 Conventional Commits)
git commit -m "docs(research): 更新架构研究笔记与操作手册"

# 4. 推送到云端个人仓库
git push
```

---

### 3.5 SSH 代理与网络排障备忘

* **测试 GitHub SSH 连通性**：
  ```sh
  ssh -T git@github.com
  # 成功返回: Hi YanmingChu! You've successfully authenticated...
  ```
* **SSH 代理配置文件路径**：`C:\Users\Naselit\.ssh\config`
  ```text
  Host github.com
      HostName github.com
      User git
      IdentityFile C:/Users/Naselit/.ssh/id_ed25519
      ProxyCommand "D:/Naselit/Git/mingw64/bin/connect.exe" -S 127.0.0.1:7890 %h %p
  ```

---

## 4. 工程质量检测与门禁检查

在提交重要代码改动前，可按需运行以下质量门禁指令：

```sh
# 1. 单元测试 (Vitest)
pnpm run test

# 2. CI 全量测试覆盖率门禁 (要求 100% 覆盖)
pnpm run test:coverage

# 3. 全局 TypeScript 类型检查
pnpm run typecheck

# 4. 代码风格与 Lint 检查 (Oxlint)
pnpm run lint

# 5. 重复代码克隆检测
pnpm run duplication

# 6. 工作区依赖与架构约束合规检查 (Knip + Publint)
pnpm run hygiene

# 7. 文档与双语同步门禁检查
pnpm run doc-sync

# 8. 文档网站本地构建与死链检测
pnpm run website:build
```

---

## 5. 调试诊断与排错指南

### 5.1 会话日志与持久化数据存储位置
* **SQLite 数据库**：默认位于本地用户缓存目录（通过 `dsh-session-persistence-sqlite` 插件写入）。
* **诊断日志**：可通过设置环境变量 `DEBUG=cordis*` 或 `NODE_ENV=development` 获取详细的插件装配与事件流日志。

### 5.2 常见问题快速诊断

| 现象 | 可能原因 | 解决办法 |
| :--- | :--- | :--- |
| `fatal: unable to access ... Connection reset` | Git 未走代理或端口错误 | 检查 Clash 是否开启 `7890` 端口，测试 `ssh -T git@github.com` |
| `npm warn Unknown env config` | npm/pnpm 版本提示 | 属于良性警告，不影响构建 |
| `typecheck` 报找不到模块声明 | 尚未生成全局类型定义 | 先执行 `pnpm run build` 生成 `lib/` 产物 |
| Web 控制台无法连接模型 | 未配置 API Key | 检查根目录 `.env` 中是否配置了有效的 `DEEPSEEK_API_KEY` |
