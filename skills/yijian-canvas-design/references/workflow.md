# Workflow 详细执行流程

## Step 1: Pre-check

执行以下命令检查偏好配置：

```bash
test -f .yijian-skills/yijian-canvas/EXTEND.md && echo "project"
test -f "$HOME/.yijian-skills/yijian-canvas/EXTEND.md" && echo "user"
```

**优先级**：项目级（`.yijian-skills/`）> 用户级（`~/.yijian-skills/`）

找到后读取并显示摘要：
```
✅ 偏好配置已加载
   水印：钧弈（右下角，透明度 0.7）
   输出目录：/Users/junyi/Downloads
   语言：zh-CN
```

未找到则跳转 [first-time-setup.md](config/first-time-setup.md)。

---

## Step 2: Design Philosophy

### 哲学文档结构

```markdown
# {哲学名称}

## 核心主张
（1段，点明美学世界观）

## 空间与形式
（如何通过空间和形状传达意义）

## 色彩与材质
（色彩系统、质感、降饱和度/高饱和度的选择）

## 尺度与节奏
（元素大小对比、重复节奏、视觉韵律）

## 构图与工艺
（构图原则、工艺标准——反复强调精心打磨、大师级执行）
```

### 哲学示例

**"数字经纬"**
制图学美学，流动与秩序并存。降饱和色彩，极淡网格，正交连接线，图例如制图框。每个节点都是地图上的坐标点，每条连线都是经纬线的延伸。

**"淡彩韵律"**
柔和色调中的信息流动。莫兰迪色系，留白呼吸，文字如水墨点缀。视觉重量轻盈，但信息密度不减。

**"Chromatic Silence"**
Color as the primary information system. Geometric precision where color zones create meaning. Typography minimal—small sans-serif labels letting chromatic fields communicate.

---

## Step 3: Deduce Reference

从用户请求中提炼隐性概念线索：

- **技术架构图** → 提炼系统的核心隐喻（如：GlobeRoller = 地球仪 + 滚动 → 制图学）
- **产品海报** → 提炼产品的精神内核
- **抽象主题** → 提炼情感或哲学意象

这个线索不直接出现在画面中，而是渗透进色彩选择、构图方式、细节处理。

---

## Step 4: Canvas Creation

### 技术规范

**画布尺寸**（Python matplotlib）：
- 标准架构图：`figsize=(22, 16)`，`xlim=(0,22)`，`ylim=(0,16)`
- 海报/艺术品：`figsize=(16, 22)`（竖版）或 `figsize=(22, 16)`（横版）
- DPI：`200`（输出质量）

**字体规范**：
- 中文：`['PingFang SC', 'Heiti SC', 'STHeiti', 'Arial Unicode MS']`
- 英文/装饰：优先使用 `./canvas-fonts` 目录下的字体
- **画布尺寸不变，只调整 fontsize**——禁止同比放大画布和字体

**颜色规范**：
- 同类元素 fontsize 必须统一
- 同层级节点颜色必须一致
- 降饱和色彩优先（避免刺眼的高饱和度）

**重叠检测**（⛔ 必须执行）：
```python
# 计算每个图形的实际边界（含 pad 扩展）
# 确保任意两个元素的边界框不相交
# 文字与节点之间最小间距 >= 0.15 个单位
```

**水印**：
```python
# 从 EXTEND.md 读取
ax.text(x, y, watermark_content,
        ha='right', va='bottom',
        fontsize=18, alpha=watermark_opacity,
        style='italic')
```

**输出路径**：
```python
output_dir = extend_config['default_output_dir']
output_path = f"{output_dir}/canvas/{topic_slug}/{filename}.png"
```

### 架构图专项规范

- **层间箭头**：起点终点必须精确对齐节点边界，不能被节点遮挡
- **SSE/数据流标签**：放在箭头旁侧，不与箭头重叠
- **图例**：宽度适配内容实际宽度，不留多余空白
- **层容器**：底部留足空间给层间箭头（至少 1.5 个单位）

---

## Step 5: Refinement

**打磨检查清单**：
- [ ] 所有元素无重叠
- [ ] 字体大小同类一致
- [ ] 颜色系统统一，区分度足够
- [ ] 箭头起止点精确
- [ ] 图例宽度适配内容
- [ ] 水印位置正确，内容来自 EXTEND.md
- [ ] 输出目录来自 EXTEND.md

**打磨原则**：不添加新图形，只精炼已有内容。问自己："如何让已有的内容更像一件艺术品？"
