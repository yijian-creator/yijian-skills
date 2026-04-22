---
name: yijian-canvas
description: Create beautiful visual art in .png and .pdf documents using design philosophy. Use when the user asks to create any static visual output, including: (1) posters, covers, banners; (2) infographics, data visualizations, charts; (3) architecture diagrams, flowcharts, technical diagrams; (4) social media images (小红书, WeChat covers, Twitter cards); (5) presentation slides or section covers; (6) brand identity visuals, logos, icons; (7) any piece of art or abstract design. Create original visual designs, never copying existing artists' work.
license: Complete terms in LICENSE.txt
---

# 一见 Canvas

通过「设计哲学 → 视觉表达」两步流程，将任意主题转化为具有艺术质感的视觉作品。

## Workflow

```
- [ ] Step 1: Pre-check（读取 EXTEND.md 偏好配置）
- [ ] Step 2: Design Philosophy（创作设计哲学 .md 文件）
- [ ] Step 3: Deduce Reference（提炼主题的隐性概念线索）
- [ ] Step 4: Canvas Creation（在画布上表达，输出 .png / .pdf）
- [ ] Step 5: Refinement（二次打磨，精益求精）
```

---

### Step 1: Pre-check ⛔ BLOCKING

**读取用户偏好配置：**

```bash
test -f .yijian-skills/yijian-canvas/EXTEND.md && echo "project"
test -f "$HOME/.yijian-skills/yijian-canvas/EXTEND.md" && echo "user"
```

| 结果 | 操作 |
|------|------|
| 找到 | 读取并解析，显示摘要（水印、输出目录、语言） |
| 未找到 | ⛔ 执行 [first-time-setup](references/config/first-time-setup.md) |

配置规范见：[references/config/preferences-schema.md](references/config/preferences-schema.md)

---

### Step 2: Design Philosophy

创作一个**视觉哲学**（不是布局模板），它将通过形式、空间、色彩、构图来诠释。

**命名运动**（1-2个词）："数字经纬" / "淡彩韵律" / "Chromatic Silence"

**撰写哲学**（4-6段，简洁而完整），涵盖：
- 空间与形式
- 色彩与材质
- 尺度与节奏
- 构图与平衡
- 视觉层级

**关键准则：**
- **避免冗余**：每个设计维度只提一次
- **反复强调工艺**：最终作品必须看起来像耗费无数小时精心打磨的，来自顶尖领域的专家之手——"精心雕琢"、"深度专业"、"大师级执行"
- **留有创作空间**：方向明确，但足够简洁，让执行阶段有诠释余地

哲学文档保持通用，不提及具体创作意图，输出为 `.md` 文件。

详细指南与示例：[references/workflow.md](references/workflow.md#step-2-design-philosophy)

---

### Step 3: Deduce Reference

**关键步骤**：在创作画布前，从原始请求中识别隐性概念线索。

主题是**嵌入艺术本身的微妙、精准引用**——不总是字面的，始终是精致的。熟悉该主题的人会直觉感受到，其他人只是欣赏一幅精妙的抽象构图。

设计哲学提供美学语言，推演出的主题提供灵魂——那条悄悄编织进形式、色彩与构图中的概念 DNA。

---

### Step 4: Canvas Creation

以哲学和概念框架为基础，在画布上表达。

- 单页，高度视觉化，设计感优先（除非要求多页）
- 使用重复图案和完美形状，借鉴系统性观察的视觉语言
- 稀疏的排版，系统性参考标记，有限但意图明确的色彩
- **文字极简**：文字是视觉元素，不是说明文字；字体优先细体
- **无重叠**：所有元素必须在画布边界内，有呼吸空间，这是不可妥协的专业标准
- **字体选择**：读取 [references/fonts.md](references/fonts.md) 按风格分类选择合适字体；让排版成为艺术本身的一部分
- **中文内容**：`canvas-fonts/` 均为英文字体，中文字符必须使用系统字体兜底（macOS: `/System/Library/Fonts/PingFang.ttc`），英文装饰元素可继续使用 `canvas-fonts/` 字体

**常用画布尺寸预设**（用户未指定时根据场景自动选择）：

| 场景 | 尺寸（px） | 比例 |
|------|-----------|------|
| 小红书竖图 | 1080 × 1440 | 3:4 |
| 小红书方图 | 1080 × 1080 | 1:1 |
| 微信公众号封面 | 900 × 383 | 约 2.35:1 |
| 微信朋友圈封面 | 1080 × 608 | 16:9 |
| 演示文稿 / 16:9 | 1920 × 1080 | 16:9 |
| 技术架构图 | 1600 × 1000 | 8:5 |
| 海报竖版 | 1080 × 1920 | 9:16 |
| 通用方图 | 1200 × 1200 | 1:1 |

**水印**：从 EXTEND.md 读取 `watermark` 配置，按指定位置和透明度添加。

**输出目录**：从 EXTEND.md 读取 `default_output_dir`，输出到 `{output_dir}/canvas/{topic-slug}/`。

输出最终 `.png` 或 `.pdf` 文件，以及设计哲学 `.md` 文件。

详细执行规范：[references/workflow.md](references/workflow.md#step-4-canvas-creation)

---

### Step 5: Refinement

**用户已经说过**："还不够完美，必须是精品，如同即将在美术馆展出。"

**打磨原则**：不要添加更多图形，而是精炼已有的内容，使其极度清晰。不要调用新函数或绘制新形状，而是问自己："如何让已有的内容更像一件艺术品？"

二次检查：构图、间距、色彩、排版——一切都要体现顶级工艺水准。

---

## Output Structure

```
{output_dir}/canvas/{topic-slug}/
├── {philosophy-name}_设计哲学.md
└── {NN}-{slug}.png / .pdf
```

## References

| 文件 | 内容 |
|------|------|
| [references/workflow.md](references/workflow.md) | 详细执行流程 |
| [references/fonts.md](references/fonts.md) | 字体选择指南（按风格分类 + 搭配建议 + 中文处理） |
| [references/config/preferences-schema.md](references/config/preferences-schema.md) | 偏好配置规范 |
| [references/config/first-time-setup.md](references/config/first-time-setup.md) | 首次初始化引导 |

## Multi-Page Option

创建多页时，沿同一设计哲学但各具特色。将页面打包为同一 `.pdf` 或多个 `.png`。将第一页视为整本咖啡桌书的开篇，后续页面是对原作的独特变奏，以优雅的方式讲述一个故事。