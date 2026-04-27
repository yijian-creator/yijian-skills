---
name: yijian-article-illustrator
author: yijian
description: 文章总结与配图规划技能。当用户提供一段原文/草稿/笔记，并希望（1）润色发小红书/公众号/博客，（2）为文章配图但不想自己想图，（3）一键产出可直接复制粘贴发布的正文 + 配图方案 时自动激活。本 skill 会动态决定图片数量与插入位置（不固定节点数），按目标渠道适配文风，输出 .md 可发布正文，并把每张图的画图任务委托给 yijian-canvas-design 执行。
---

# 一见 Article Illustrator

把任意原文 → **目标渠道风格的可发布正文 + 动态配图方案**。
图片数量与位置由原文内容决定（不写死），画图本身委托给 `yijian-canvas-design`。

---

## ⛔ 全局硬约束

- **语言：所有产出必须为简体中文**。包括：改写后的正文、标题、图片 `title` / `prompt` / `key_elements`、`article.json` 中所有面向人阅读的字段，以及与用户的对话回复。
- 即使原文是英文/混合语种，也必须翻译/改写为中文输出（仅当原文是代码块、技术专有名词、人名、品牌名时保留原文）。
- 传给 `yijian-canvas-design` 的 `prompt` 字段也必须是中文（保证画面里的文字是中文）。

---

## Workflow

```
- [ ] Step 1: Pre-check（读取 EXTEND.md 偏好配置）
- [ ] Step 2: Intake（确认原文 + 目标渠道）
- [ ] Step 3: Analyze（语义分析 + 触发器扫描 + 决定图片数量与位置）
- [ ] Step 4: Rewrite（按渠道风格改写正文，留出图片占位）
- [ ] Step 5: Illustrate（生成 canvas_brief，调起 yijian-canvas-design 画图）
- [ ] Step 6: Assemble（回填图片路径，输出 article.md 到下载目录）
```

---

### Step 1: Pre-check ⛔ BLOCKING

```bash
test -f .yijian-skills/yijian-article-illustrator/EXTEND.md && echo "project"
test -f "$HOME/.yijian-skills/yijian-article-illustrator/EXTEND.md" && echo "user"
```

| 结果 | 操作 |
|------|------|
| 找到 | 读取并显示摘要（默认渠道、署名、是否默认带封面） |
| 未找到 | 使用内置默认值（无署名、不强制封面、首次使用反问渠道）；不阻塞流程 |

> **本 skill 的 EXTEND.md 与 yijian-canvas-design 的 EXTEND.md 完全独立**，互不读取。当 Step 5 调起 yijian-canvas-design 时，由那个 skill 自行读取它自己的 EXTEND.md。

---

### Step 2: Intake

确认两件事：
1. **原文** — 文本本体或文件路径
2. **目标渠道** ∈ `{xiaohongshu | wechat | blog}`

如果用户没指定渠道，**必须反问**（不要默认），话术示例：
> 这篇你想发到哪？(1) 小红书 (2) 公众号 (3) 博客 — 不同渠道我会换不同的语气和排版

**文章标题**：如果原文没有显式标题，从首段或核心论点提炼一个，并征求用户确认（用于命名输出目录）。

---

### Step 3: Analyze（核心动态决策）

**这一步不写死任何数量，全部由内容驱动。**

执行三件事：

**3.1 触发器扫描**（决定"必须出图"的最小集）
扫描原文，命中以下任意一类即记一次必出图：
- 流程/步骤（"第一步…第二步…"、有序列表表达过程）
- 对比/分类（"A vs B"、"三种类型"、对比表格）
- 数据/趋势（≥3 个百分比/数字、增长描述、占比说明）
- 结构/层级（系统架构、组织结构、嵌套关系）
- 时间线（年份序列、阶段演进）

**3.2 计算图片数量**
```
min_count = 渠道最小出图门槛  # 见 references/styles.md
trigger_count = 触发器命中数
image_count = max(min_count, trigger_count)
```

**3.3 选定插入位置**
扫描"语义边界"，给每个候选点打分，贪心挑前 `image_count` 个，并保证两图间距 ≥ 200 字。

详细算法见 [references/decision-algorithm.md](references/decision-algorithm.md)。

> ⛔ 严禁使用"每 N 段插一张"这种固定步长策略。位置必须由内容语义决定。

---

### Step 4: Rewrite

按目标渠道风格改写原文：

| 渠道 | 语气 | 段落 | Emoji | 标题 |
|------|------|------|-------|------|
| 小红书 | 口语化、感染力强 | 短段、留白多 | 适度使用 | 带钩子 |
| 公众号 | 专业有温度 | 中等长度 | 节制 | 信息明确 |
| 博客 | 结构化、思辨 | 可长 | 几乎不用 | 客观 |

完整规范：[references/styles.md](references/styles.md)

改写后的正文中，按 Step 3.3 选定的位置插入占位符：
```
![](placeholder_00.png)
![](placeholder_01.png)
```

---

### Step 5: Illustrate

为每张图生成一个 `canvas_brief`，逐张调起 `yijian-canvas-design`：

```yaml
canvas_brief:
  index: 1
  role: flow_chart                      # cover | flow_chart | comparison | data_chart | structure | timeline | concept
  size: 1080x1440                       # 由渠道决定
  title: "Vibe Coding 的 3 步工作流"
  philosophy_hint: "淡彩韵律 / 数字经纬"  # 给 canvas-design 的风格提示
  key_elements:
    - "需求拆解"
    - "Prompt 编写"
    - "增量验证"
  prompt: "用三个相连的卡片表达流程……"     # 自然语言提示词
  output_path: "~/Downloads/{title-slug}/images/01-flow.png"
```

完整交接规范：[references/canvas-handoff.md](references/canvas-handoff.md)

> 本 skill **不自己画图**。生成 brief 后必须显式触发 `yijian-canvas-design`。

---

### Step 6: Assemble

把 Step 5 拿到的实际图片路径回填到正文，输出最终文件。

**输出位置**（硬规则，不读 EXTEND.md）：
```
~/Downloads/{文章标题-slug}/
├── article.md           # 可直接复制粘贴发布的正文
├── images/
│   ├── 00-cover.png
│   ├── 01-flow.png
│   └── ...
└── article.json         # 仅当用户加 --with-json 时输出
```

文章标题 slug 化规则：
- 中文保留，去除标点
- 空格替换为 `-`
- 长度截断到 30 字符
- 重名时追加 `-2`、`-3`

完整 schema：[references/output-schema.md](references/output-schema.md)

---

## Output Contract

- **默认产物**：`article.md`（已嵌入图片相对路径，可直接发布）
- **可选产物**：`article.json`（仅 `--with-json` 触发；为未来 API 化预留）
- **图片产物**：由 `yijian-canvas-design` 写入 `images/` 子目录

---

## References

| 文件 | 内容 |
|------|------|
| [references/decision-algorithm.md](references/decision-algorithm.md) | 图片数量公式 + 语义边界识别 + 触发器规则（参考《用图表说话》） |
| [references/styles.md](references/styles.md) | 三渠道风格规范（语气/排版/emoji/最小出图） |
| [references/output-schema.md](references/output-schema.md) | article.md 与 article.json 的输出契约 |
| [references/canvas-handoff.md](references/canvas-handoff.md) | 与 yijian-canvas-design 的交接协议 |

---

## Non-Goals

- ❌ 不实际渲染图片（交给 `yijian-canvas-design`）
- ❌ 不做事实核查、SEO 优化、多语种翻译
- ❌ 不做视频/音频生成
- ❌ 不读取 `yijian-canvas-design` 的 EXTEND.md（两个 skill 配置完全独立）
