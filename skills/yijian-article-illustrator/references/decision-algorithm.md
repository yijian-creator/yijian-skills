# Decision Algorithm — 动态决定图片数量与位置

> 核心原则：**图片是文章的视觉论据，不是装饰**。出图与否、出几张、出在哪，全部由内容本身决定。
> 思想来源：Gene Zelazny《用图表说话》—— 先想清楚"我要表达什么关系"，再选图表类型；没有需要表达的关系时，不要画图。

---

## 一、图片数量

```
image_count = max(min_count_by_channel, trigger_count)
```

### 1.1 渠道最小出图门槛

| 渠道 | 最小出图 | 说明 |
|------|---------|------|
| 小红书 | 1（封面恒定） | 没图等于没流量 |
| 公众号 | 1（封面恒定） | 列表页要看封面 |
| 博客 | 0 | 允许纯文字；触发器命中才出 |

### 1.2 触发器（命中即必出图）

按《用图表说话》的关系分类，五大触发器对应五种图表：

| 触发器 | 识别信号（正则/语义） | 推荐图表 |
|-------|--------------------|---------|
| **流程/步骤** | "第一步…第二步…"、"接着…然后…最后…"、有序列表 ≥3 项描述动作 | 流程图 / 步骤卡片 |
| **对比/分类** | "A vs B"、"X 和 Y 的区别"、"三种…"、并列段落带评价词 | 对比表 / 矩阵图 |
| **数据/趋势** | 同段出现 ≥3 个数字或百分比，"增长 / 下降 / 占比 / 翻倍" | 柱图/线图/饼图（按关系选） |
| **结构/层级** | "由…组成"、"包含…模块"、"分为…层"、嵌套描述 | 架构图 / 思维导图 |
| **时间线** | 年份序列、"2020 年 → 2023 年"、阶段演进 | 时间轴 |

> 触发器由 LLM 语义判断 + 关键词正则双保险。同一段触发多个时只算 1 次（按主导关系挑图）。

### 1.3 图表类型选择规则（《用图表说话》核心）

先问自己想表达哪种**比较关系**：

| 想表达的关系 | 选图 |
|------------|------|
| 成分（占整体的多少） | 饼图 |
| 项目（不同事物的对比） | 柱图 |
| 时间序列（随时间变化） | 线图 |
| 频率分布（多少项落在某区间） | 直方图 |
| 相关性（两变量是否有关） | 散点图 |
| 流程（A 如何变成 B） | 流程图 |
| 结构（系统由什么组成） | 架构图 |

→ 写进 `canvas_brief.role`，传给 `yijian-canvas-design`。

---

## 二、插入位置：语义边界

**语义边界 = 读者注意力会自然停顿的位置**，是图片插入的天然候选点。

### 2.1 边界识别（打分）

| 边界类型 | 识别信号 | 权重 |
|---------|---------|------|
| 结构边界 | `#` `##` 标题、`---` 分割线 | 5 |
| 章节空段 | 连续 ≥2 个空行 | 3 |
| 逻辑转折 | "但是 / 然而 / 不过 / 反过来 / 接下来 / 总结" | 2 |
| 案例引入 | "比如 / 举个例子 / 案例 / 看下面" | 2 |
| 列表起点 | `- ` 或 `1. ` 开头的连续行 | 1 |
| 触发器锚点 | 紧跟在触发段落前 | +3 加成 |

### 2.2 选位算法（贪心 + 间距约束）

```python
def pick_positions(text, image_count):
    boundaries = scan_boundaries(text)        # [(offset, score, type), ...]
    boundaries.sort(key=lambda b: -b.score)

    picks = []
    for b in boundaries:
        if len(picks) >= image_count:
            break
        # 间距约束：与已选位置的字数距离 ≥ MIN_GAP
        if all(abs(b.offset - p.offset) >= MIN_GAP for p in picks):
            picks.append(b)

    return sorted(picks, key=lambda p: p.offset)

MIN_GAP = 200   # 字符；小红书可降到 120，博客可升到 400
```

### 2.3 封面位置（仅小红书/公众号）

封面 = `index=0`，恒定插入到正文最前（H1 标题之后第一个段落之前），不参与上述贪心选位。

---

## 三、降级策略

| 场景 | 处理 |
|------|------|
| 文章 <300 字 | 强制只出 1 张封面（小红书/公众号），博客 0 张 |
| 触发器全部未命中 + 博客渠道 | 出 0 张，纯文字交付，提醒用户："本文未识别到适合图表的关系，已按纯文字输出" |
| 触发器命中数 > 8 | 截断到 8，并提示用户："本文触发了 N 个图表点，已挑选最关键的 8 个；如需全部请告知" |
| 文章长 >5000 字但触发器只有 1 个 | 仍只出 1 张，不为了凑数硬加 |

---

## 四、伪代码总览

```python
def plan_illustrations(text, channel):
    # 1. 触发器扫描
    triggers = detect_triggers(text)        # [Trigger(type, span, chart_type), ...]

    # 2. 数量计算
    min_count = MIN_COUNT_BY_CHANNEL[channel]
    image_count = max(min_count, len(triggers))
    image_count = min(image_count, 8)       # 上限保护

    # 3. 选位
    boundaries = scan_boundaries(text)
    positions = pick_positions(boundaries, image_count - has_cover(channel))

    # 4. 拼装 brief
    briefs = []
    if has_cover(channel):
        briefs.append(make_cover_brief(text, channel))
    for trig, pos in zip_with_position(triggers, positions):
        briefs.append(make_chart_brief(trig, pos, channel))

    return briefs
```
