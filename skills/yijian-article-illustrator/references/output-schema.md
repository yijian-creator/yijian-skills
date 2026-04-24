# Output Schema — 输出契约

---

## 一、文件结构

```
~/Downloads/{文章标题-slug}/
├── article.md           # 默认产物：可直接复制粘贴发布的正文
├── images/              # 图片（由 yijian-canvas-design 生成）
│   ├── 00-cover.png
│   ├── 01-flow.png
│   ├── 02-comparison.png
│   └── ...
└── article.json         # 可选：仅当用户加 --with-json 时生成
```

### 1.1 标题 slug 化规则

```python
def slugify(title: str) -> str:
    # 1. 去除 markdown 标记和首尾空白
    s = re.sub(r"[#*`>\-]", "", title).strip()
    # 2. 去除标点（中英文）
    s = re.sub(r"[，。！？、；：""''（）【】《》,.!?;:\"'()\[\]<>/\\]", "", s)
    # 3. 空格 → -
    s = re.sub(r"\s+", "-", s)
    # 4. 截断到 30 字符
    s = s[:30]
    # 5. 重名时追加 -2 -3
    return ensure_unique(s)
```

### 1.2 图片命名

```
{NN}-{role}.png
```
- `NN`：00 起的两位序号，与正文出现顺序一致
- `role`：`cover` / `flow` / `comparison` / `data` / `structure` / `timeline` / `concept`

---

## 二、article.md（默认产物）

直接是可发布的 markdown 正文，图片用相对路径引用：

```markdown
# Vibe Coding 的 3 步工作流

![](images/00-cover.png)

> 写在前面：本文献给所有想用 AI 写代码、但不知道从哪开始的朋友。

## 一、为什么需要工作流

...正文...

![](images/01-flow.png)

## 二、三个核心动作

...正文...
```

> 用户复制粘贴到任意平台（小红书图文、公众号编辑器、Hexo/Notion）都能直接渲染。

---

## 三、article.json（可选产物）

仅当用户在指令中加 `--with-json` 时生成。Schema 如下：

```json
{
  "$schema": "https://yijian-skills.dev/article-illustrator/v1",
  "meta": {
    "title": "Vibe Coding 的 3 步工作流",
    "title_slug": "Vibe-Coding-的-3-步工作流",
    "channel": "wechat",
    "word_count": 1842,
    "image_count": 3,
    "generated_at": "2026-04-24T15:00:00+08:00",
    "skill_version": "1.0.0"
  },
  "illustrations": [
    {
      "index": 0,
      "role": "cover",
      "anchor": {
        "type": "before_h1",
        "offset": 0
      },
      "title": "Vibe Coding 工作流",
      "chart_type": "cover",
      "size": "900x383",
      "prompt": "封面：左侧大字标题『Vibe Coding 工作流』，右侧抽象几何元素表达流动感，淡彩色系",
      "canvas_brief": {
        "philosophy_hint": "淡彩韵律",
        "key_elements": ["流动", "节奏", "现代感"],
        "palette_hint": "莫兰迪",
        "size": "900x383",
        "output_path": "images/00-cover.png"
      },
      "image_path": "images/00-cover.png",
      "status": "rendered"
    },
    {
      "index": 1,
      "role": "flow",
      "anchor": {
        "type": "before_h2",
        "section_title": "三个核心动作",
        "offset": 642
      },
      "chart_type": "flow_chart",
      "size": "1200x800",
      "prompt": "三步流程图：拆解需求 → 编写 Prompt → 增量验证，每步配 1 行说明",
      "canvas_brief": {
        "philosophy_hint": "数字经纬",
        "key_elements": ["拆解需求", "编写 Prompt", "增量验证"],
        "size": "1200x800",
        "output_path": "images/01-flow.png"
      },
      "image_path": "images/01-flow.png",
      "status": "rendered"
    }
  ],
  "rendered_article_with_placeholders": "# Vibe Coding...\n![](placeholder_00.png)\n...",
  "publish_ready_text": "# Vibe Coding...\n![](images/00-cover.png)\n..."
}
```

### 3.1 字段说明

| 字段 | 说明 |
|------|------|
| `meta.channel` | `xiaohongshu` / `wechat` / `blog` |
| `illustrations[].role` | `cover` / `flow` / `comparison` / `data` / `structure` / `timeline` / `concept` |
| `illustrations[].anchor.type` | `before_h1` / `before_h2` / `before_paragraph` / `after_list` |
| `illustrations[].anchor.offset` | 在原文中的字符偏移量 |
| `illustrations[].chart_type` | 《用图表说话》七种图表之一 |
| `illustrations[].canvas_brief` | 给 yijian-canvas-design 的完整任务包 |
| `illustrations[].status` | `planned` / `rendered` / `failed` |
| `rendered_article_with_placeholders` | 占位符版正文（图片未渲染前的中间态） |
| `publish_ready_text` | 与 article.md 内容一致 |

---

## 四、不输出的东西

- ❌ 不输出原文备份（如需保留请用户自行 git）
- ❌ 不输出渲染日志（除非画图失败，失败时把错误写入 article.json 的 `illustrations[].error`）
