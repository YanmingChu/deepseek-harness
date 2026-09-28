# 🎨 `dsh-research/html/` 交互式网页仪表盘目录

本目录存放 DeepSeek Harness 专属研究工作区的**交互式网页仪表盘（Interactive Dashboards）**。所有 HTML 文件均采用现代化极客科技风设计，并支持深浅色主题无缝切换与动态渲染。

---

## 📁 页面清单

| 页面文件 | 对应 Markdown 基线 | 说明 |
| :--- | :--- | :--- |
| **[index.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/index.html)** | [../README.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/README.md) | **仪表盘总入口**，默认展示架构全景与快速导航 |
| **[architecture.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/architecture.html)** | [../ARCHITECTURE.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/ARCHITECTURE.md) | **架构全景仪表盘**：包含 Mermaid 时序图、本地 vs 云端职责分栏、Monorepo 对照表与六大研究专题 Tab 切换 |
| **[operations.html](file:///c:/DeepSeek/deepseek-harness/dsh-research/html/operations.html)** | [../OPERATIONS.md](file:///c:/DeepSeek/deepseek-harness/dsh-research/OPERATIONS.md) | **运维与同步控制台**：包含官方代码 3 步同步、启动模式速查、一键复制代码块、Git 拓扑图与排障表 |

---

## 🌟 设计系统规范 (Design System Tokens)

所有本目录下的 HTML 页面均遵循以下 CSS 变量与设计规范：

```css
:root {
  --bg-primary: #0b0f19;                         /* 背景底色 */
  --bg-secondary: #111827;                       /* 侧边栏与卡片底色 */
  --bg-card: rgba(17, 24, 39, 0.75);             /* 毛玻璃半透明卡片 */
  --border-color: rgba(255, 255, 255, 0.08);     /* 极细微边框 */
  --text-primary: #f3f4f6;                       /* 主文字白色 */
  --text-secondary: #9ca3af;                     /* 次级文字灰色 */
  --accent-cyan: #38bdf8;                        /* 强调色 亮青 */
  --accent-purple: #818cf8;                      /* 强调色 紫 */
  --accent-emerald: #34d399;                     /* 强调色 翠绿 */
}
```

---

## 🛠️ AI 维护与同步协议

后续当上级目录的 Markdown 文档（`ARCHITECTURE.md` 或 `OPERATIONS.md`）发生内容变更时：
1. **保持同构**：Markdown 永远作为信息基线，HTML 作为交互展示层。
2. **样式一致性**：更新 HTML 内容时，必须严格沿用现有卡片、Tab 切换器、代码复制框及 Mermaid 图表结构，严禁破坏主题系统。
3. **路径约定**：HTML 页面内所有跳转链接需使用相对路径或标准 `file:///c:/DeepSeek/deepseek-harness/...` 绝对路径。
