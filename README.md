# yijian-skills

Skills shared by yijian for improving daily work efficiency with Claude.

---

## Available Skills

### 🎯 vibe-coding-starter

**Vibe Coding 入门教练** —— 帮助零基础小白快速上手 AI 编程。

| 功能 | 说明 |
|------|------|
| 🚀 项目启动引导 | 手把手帮你写出第一个完整的 Prompt，启动第一个 AI 编程项目 |
| 📋 Prompt 模板库 | 7 类可直接复制的模板，覆盖启动、开发、调试、审查全流程 |
| 🔧 问题排查指南 | 代码跑不起来？AI 改坏了？按图索骥，一步步定位解决 |
| 📚 10 条核心规则 | 从实战中提炼，帮你建立正确的 Vibe Coding 工作习惯 |

**触发方式**：安装后直接描述你想做的事，如"帮我做一个待办清单"、"代码跑不起来怎么办"，skill 会自动介入引导。

---

### ✍️ yijian-article-illustrator

**文章总结与配图规划** —— 把任意原文一键变成「目标渠道风格的可发布正文 + 动态配图方案」。

| 功能 | 说明 |
|------|------|
| 🎯 渠道风格适配 | 小红书 / 公众号 / 博客 三种风格，语气、排版、emoji 全部按渠道走 |
| 🧠 动态配图决策 | 不用固定节点数，按内容触发器（流程/对比/数据/结构/时间）和语义边界算出图位与张数，参考《用图表说话》 |
| 🎨 委托画图 | 自动生成 `canvas_brief` 并交给 `yijian-canvas-design` 出图，互不干扰 |
| 📦 即发即用 | 默认输出 `article.md`（已嵌入图片），可选 `article.json` 给 API 调用，统一落到 `~/Downloads/{文章标题}/` |

**触发方式**：把原文丢进对话框，说一句"帮我改成小红书 / 公众号 / 博客"，skill 自动介入。

---

## Installation

### Claude.ai

1. 下载本仓库：点击右上角 **Code → Download ZIP**
2. 解压后找到 `skills/vibe-coding-starter` 文件夹，打包成 `.zip`
3. 打开 [claude.ai](https://claude.ai) → 左侧菜单 → **Skills → Upload Skill**
4. 上传 zip 文件，安装完成 ✅

### Claude Code

```bash
/plugin marketplace add yijian-creator/yijian-skills
```

---

## Author

**yijian** · [github.com/yijian-creator](https://github.com/yijian-creator)
