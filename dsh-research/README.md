# 🔬 DeepSeek Harness (`dsh`) 专属研究工作区

欢迎来到 DeepSeek Harness 专属研究工作区！本目录旨在为深入研读源码、实验新特性、定制专属 Agent 插件及多端协同提供完整、独立、零冲突的文档与可视化工具支撑。

---

## 📁 目录结构与快速导航

```text
c:\DeepSeek\deepseek-harness\dsh-research\
├── README.md               # 📌 [当前文件] 本研究目录总览与结构导引
├── ARCHITECTURE.md         # 🏛️ [架构全景与研究指南] 核心设计哲学、Agent Loop、Monorepo与六大专题
├── OPERATIONS.md           # 🚀 [常用命令与上游同步] 日常运行模式、3步同步官方代码、质量门禁与排障
├── index.html              # 🔗 根目录快捷入口（自动跳转至 html/index.html）
└── html/                   # 🎨 [交互式网页仪表盘] 本地可视化控制台与交互式文档中心
    ├── README.md           # 📜 HTML 目录作用说明与视觉设计/维护规范
    ├── index.html          # 🌟 仪表盘主入口（自动呈现架构全景）
    ├── architecture.html   # 🏛️ 架构全景交互式仪表盘（含 Mermaid 时序、Tab 切换、代码追踪）
    └── operations.html     # ⚡ 常用命令与上游同步操作面板（含一键复制、仓库拓扑、排障表）
```

---

## 📚 文档职责与分工

| 文档名称 | 定位与核心内容 | 对应可视化网页 |
| :--- | :--- | :--- |
| [ARCHITECTURE.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/ARCHITECTURE.md) | **架构认知与深度原理**：Cordis 插件机制、本地 vs 云端职责对比、Agent Loop 轮次生命周期时序、Monorepo 核心包映射、六大研究路线。 | [html/architecture.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/architecture.html) |
| [OPERATIONS.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/OPERATIONS.md) | **日常运维与操作手册**：Web 控制台/Headless 启动命令、`upstream` 官方代码 3 步同步流、多设备研究部署、质量门禁与排障指南。 | [html/operations.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/operations.html) |
| [html/README.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/README.md) | **HTML 维护与设计规范**：说明 HTML 文件夹的作用、现代设计系统 Token 定义、以及从 Markdown 同步更新 HTML 的 AI 规范。 | [html/index.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/index.html) |

---

## 💡 为什么建立独立的 `dsh-research/` 目录？

1. **零冲突合并（Zero-conflict Sync）**：
   - DeepSeek 官方开源仓库不会包含 `dsh-research/` 目录。
   - 当你在本地执行 `git merge upstream/master` 同步官方最新代码时，你的所有研究笔记、个人修改和自定义仪表盘**永远不会产生代码冲突**。
2. **多设备无缝漫游（Multi-device Roaming）**：
   - 本目录已关联到你的个人 GitHub Fork 仓库（`git@github.com:YanmingChu/deepseek-harness.git`）。
   - 在任何新电脑上只需 `git clone` 你的个人仓库并切换到 `my-branch`，即可瞬间恢复完整的研究上下文。
3. **双模态阅读体验（Markdown + HTML）**：
   - **Markdown**：作为文本基线与 Git 版本追踪源，适合在编辑器内快速查阅与编辑。
   - **HTML Dashboard**：提供现代化深浅色主题切换、一键复制代码、Mermaid 动态图表渲染及分类 Tab 导航，提供媲美生产级产品文档的可视化体验。
