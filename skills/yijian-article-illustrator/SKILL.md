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

按目标渠道风格改写原文，**必须执行以下三个子步骤**：

#### Step 4.1: 框架提炼（D2 优化）

在动笔改写之前，先完成一次"框架跃迁"：

1. 用一句话概括原文核心冲突（如"模型进化速度 vs 传统PM方法论的失效"）
2. **提出一个新的上位概念/框架**来重新组织全文（不能只是翻译和结构重组）
   - 好的框架示例："Harness Engineering"、"意图经济"、"反脆弱组织"
   - 差的框架：只是按原文章节翻译，或者只加了小标题
3. 用这个框架作为文章的叙事主线，贯穿全文

> ⚠️ 框架不是凭空编造，必须从原文的核心洞察中自然生长出来。如果原文本身已有明确框架（如"四个转变"），则需要在此基础上**升维**——找到"为什么是这四个"背后的统一逻辑。

#### Step 4.2: 改写正文

| 渠道 | 语气 | 段落 | Emoji | 标题 |
|------|------|------|-------|------|
| 小红书 | 口语化、感染力强 | 短段、留白多 | 适度使用 | 带钩子 |
| 公众号 | 专业有温度 | 中等长度 | 节制 | 按 styles.md 标题策略 |
| 博客 | 结构化、思辨 | 可长 | 几乎不用 | 客观 |

完整规范：[references/styles.md](references/styles.md)

#### Step 4.3: 人格注入（D5 优化）

改写完成后，进行一轮"人格化润色"：

1. **独立判断**：在文章中至少 2 处插入作者的独立观点/判断（以"我认为"、"这意味着"、"很多人忽略了"等引导）
2. **关联经验**：如果 EXTEND.md 中有用户的个人背景/阅读收获，将其自然融入正文
3. **情绪标记**：在关键论点处加入轻微的情绪表达（如"这一点让我非常兴奋"、"坦率地说这很反直觉"），但不超过 3 处
4. **结尾升华**：结尾不能只是"总结原文"，必须给出一个指向未来的独立判断或行动建议

> ⚠️ **人格感 ≠ 自我推广**。禁止出现"关注我的公众号"、"转发收藏"等引流话术。人格感来自独立思考，不是营销话术。

---

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

### Step 5.5: Quality Check ⛔ BLOCKING — 强制两轮

图片渲染完成后，**必须运行至少两轮质检**，全部 ✅ 才能进入 Step 6。

> ⚠️ **为什么必须两轮？** 实测经验：第一轮修复后常引入新问题（如修居中时压缩了卡片高度、替换代码时误删标题）。第二轮是验证第一轮修复没有引入回退。

#### 第一轮：渲染后自检

1. 渲染时同步输出 `images/elements.json`（元素坐标清单）
2. 运行 `check.py images/` 自动执行四层自查清单（20 项）
3. **像素级垂直居中检测**（PIL 扫描首末内容行）：`|top_margin - bottom_margin| ≤ 10px`
4. **肉眼逐图检查**：标题是否可见、文字是否截断、卡片内容是否拥挤
5. 如有 ❌，修复 → 重新渲染 → 重新检查（此循环不计入"两轮"）

#### 第二轮：修复后回归验证

6. 第一轮所有修复完成后，**必须再跑一次完整检查**
7. 重点验证：
   - 修复居中时，标题/内容是否仍然完整可见（防误删）
   - 修复字号/间距时，卡片内部空间是否仍然充足（防拥挤）
   - 所有图片的像素居中是否仍然通过
8. 全部 ✅ 后继续

#### 检查不通过的处理

- 单图修复后该图必须重新像素检测
- 连续 3 轮同一项检查不通过 → 停下来询问用户

质检工具包：[references/check-toolkit.md](references/check-toolkit.md)
四层自查清单定义：[references/styles.md](references/styles.md) § 6.7

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
| [references/check-toolkit.md](references/check-toolkit.md) | 通用配图质检工具包（elements.json schema + check.py） |

---

## Non-Goals

- ❌ 不实际渲染图片（交给 `yijian-canvas-design`）
- ❌ 不做事实核查、SEO 优化、多语种翻译
- ❌ 不做视频/音频生成
- ❌ 不读取 `yijian-canvas-design` 的 EXTEND.md（两个 skill 配置完全独立）
