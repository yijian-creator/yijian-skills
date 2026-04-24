# Canvas Handoff — 与 yijian-canvas-design 的交接协议

本 skill **不自己画图**。Step 5 阶段把每张图作为子任务委托给 `yijian-canvas-design` skill 执行。

---

## 一、交接时机

`Step 4: Rewrite` 完成后，正文里有 N 个 `![](placeholder_NN.png)` 占位符。
对每个占位符，构造 `canvas_brief`，然后**逐张调起** `yijian-canvas-design`。

> 串行调用，避免并发抢资源；如未来 canvas-design 支持批处理可改并行。

---

## 二、canvas_brief 结构

这是本 skill 唯一对外的"画图任务包"，字段必须完整：

```yaml
canvas_brief:
  # —— 必填 ——
  index: 1                              # 图片序号，与文件名 NN 对齐
  role: flow_chart                      # 图表角色（决定 canvas-design 用什么模板）
  size: "1200x800"                      # 像素尺寸；由渠道 + role 决定
  title: "Vibe Coding 的 3 步工作流"     # 图片主标题
  prompt: |                             # 自然语言描述，越具体越好
    用三个相连的卡片表达三个步骤：
    1. 拆解需求（图标：放大镜）
    2. 编写 Prompt（图标：对话框）
    3. 增量验证（图标：勾选）
    卡片之间用细箭头连接，整体淡彩色系，留白充足。

  # —— 强烈建议填 ——
  philosophy_hint: "数字经纬"            # 给 canvas-design 的设计哲学倾向
  key_elements:                         # 必须出现在画面里的关键元素（文字/概念）
    - "拆解需求"
    - "编写 Prompt"
    - "增量验证"
  palette_hint: "莫兰迪"                # 调色板倾向（可选）

  # —— 输出控制 ——
  output_path: "~/Downloads/{title-slug}/images/01-flow.png"
  watermark: false                      # 是否加水印；默认 false（由本 skill 控制，不让 canvas 自己加）
```

---

## 三、role → 尺寸映射

| role | 小红书 | 公众号 | 博客 |
|------|--------|--------|------|
| cover | 1080×1440 | 900×383 | 1600×900 |
| flow_chart | 1080×1080 | 900×500 | 1200×800 |
| comparison | 1080×1080 | 900×600 | 1200×800 |
| data_chart | 1080×1080 | 900×500 | 1200×600 |
| structure | 1080×1440 | 900×600 | 1600×1000 |
| timeline | 1080×1440 | 900×400 | 1600×600 |
| concept | 1080×1080 | 900×500 | 1200×800 |

---

## 四、role → philosophy_hint 推荐映射

仅作建议，最终由 `yijian-canvas-design` 自行决定：

| role | 推荐哲学 |
|------|---------|
| cover | 淡彩韵律 / Chromatic Silence |
| flow_chart | 数字经纬 |
| comparison | 数字经纬 / 淡彩韵律 |
| data_chart | Chromatic Silence |
| structure | 数字经纬 |
| timeline | 淡彩韵律 |
| concept | Chromatic Silence |

---

## 五、调起话术示例

把 `canvas_brief` 转成自然语言任务，触发 `yijian-canvas-design`：

> 请用 yijian-canvas-design 画一张图：
> - 角色：flow_chart
> - 尺寸：1200×800
> - 标题：Vibe Coding 的 3 步工作流
> - 设计哲学倾向：数字经纬
> - 关键元素：拆解需求、编写 Prompt、增量验证
> - 描述：用三个相连的卡片表达三个步骤……
> - 输出到：~/Downloads/Vibe-Coding-的-3-步工作流/images/01-flow.png

---

## 六、配置隔离

> ⛔ 本 skill **不读取** `yijian-canvas-design` 的 EXTEND.md，反之亦然。
> 当 `yijian-canvas-design` 被调起后，由它自己读取它自己的 EXTEND.md（处理水印、字体、设计偏好等）。
> 本 skill 仅传递任务参数（尺寸、prompt、输出路径）。

---

## 七、失败处理

| 场景 | 处理 |
|------|------|
| canvas-design 报错 | 把错误写入 `article.json` 的 `illustrations[i].error`；正文里保留 `![](placeholder_NN.png)` 占位符并附文字说明 |
| 用户中断 | 已渲染的图保留，未渲染的标记 `status: planned`，可后续续跑 |
| 输出路径冲突 | canvas-design 自身处理；本 skill 不重试 |
