# 字体选择指南

> 所有字体位于 `./canvas-fonts/` 目录。根据设计风格和内容语言选择合适的字体。

---

## 快速选择：按风格分类

### 🎨 优雅 / 高端 / 艺术（Elegant & Editorial）

适合：艺术海报、品牌视觉、高端封面、美学内容

| 字体 | 文件 | 特点 |
|------|------|------|
| **Gloock** | `Gloock-Regular.ttf` | 衬线，古典优雅，适合大标题 |
| **Italiana** | `Italiana-Regular.ttf` | 细衬线，意式高端感 |
| **CrimsonPro** | `CrimsonPro-Regular/Bold/Italic.ttf` | 书籍衬线，文学气质 |
| **LibreBaskerville** | `LibreBaskerville-Regular.ttf` | 经典衬线，沉稳权威 |
| **Lora** | `Lora-Regular/Bold/Italic.ttf` | 温润衬线，适合正文与标题 |
| **YoungSerif** | `YoungSerif-Regular.ttf` | 现代衬线，时尚感强 |
| **PoiretOne** | `PoiretOne-Regular.ttf` | 几何装饰风，轻盈优雅 |

---

### 💼 现代 / 专业 / 商务（Modern & Professional）

适合：技术架构图、信息图、商业报告、产品介绍

| 字体 | 文件 | 特点 |
|------|------|------|
| **BricolageGrotesque** | `BricolageGrotesque-Regular/Bold.ttf` | 现代无衬线，科技感 |
| **InstrumentSans** | `InstrumentSans-Regular/Bold/Italic.ttf` | 清晰专业，适合正文 |
| **WorkSans** | `WorkSans-Regular/Bold/Italic.ttf` | 工整易读，商务首选 |
| **Outfit** | `Outfit-Regular/Bold.ttf` | 圆润现代，友好感 |
| **Jura** | `Jura-Light/Medium.ttf` | 细腻精准，科技仪表感 |
| **SmoochSans** | `SmoochSans-Medium.ttf` | 流畅无衬线，活泼专业 |

---

### 🖥️ 技术 / 代码 / 数据（Technical & Monospace）

适合：技术架构图、代码展示、数据可视化、极客风格

| 字体 | 文件 | 特点 |
|------|------|------|
| **GeistMono** | `GeistMono-Regular/Bold.ttf` | Vercel 出品，现代等宽 |
| **JetBrainsMono** | `JetBrainsMono-Regular/Bold.ttf` | 开发者首选等宽字体 |
| **IBMPlexMono** | `IBMPlexMono-Regular/Bold.ttf` | IBM 风格，工业感强 |
| **DMMono** | `DMMono-Regular.ttf` | 简洁等宽，低调专业 |
| **RedHatMono** | `RedHatMono-Regular/Bold.ttf` | 开源工程感 |
| **IBMPlexSerif** | `IBMPlexSerif-Regular/Bold/Italic.ttf` | 技术衬线，报告感 |

---

### 🎯 标题 / 展示 / 冲击（Display & Impact）

适合：海报大标题、封面主视觉、强调性文字

| 字体 | 文件 | 特点 |
|------|------|------|
| **BigShoulders** | `BigShoulders-Regular/Bold.ttf` | 超宽展示字，力量感 |
| **Boldonse** | `Boldonse-Regular.ttf` | 粗黑展示，极强冲击力 |
| **EricaOne** | `EricaOne-Regular.ttf` | 紧凑粗体，海报感 |
| **NationalPark** | `NationalPark-Regular/Bold.ttf` | 户外探险风，自然感 |
| **Tektur** | `Tektur-Regular/Medium.ttf` | 科技展示字，游戏感 |
| **ArsenalSC** | `ArsenalSC-Regular.ttf` | 小型大写，军事/正式感 |

---

### ✍️ 手写 / 创意 / 个性（Handwritten & Creative）

适合：个人品牌、创意海报、温暖风格内容

| 字体 | 文件 | 特点 |
|------|------|------|
| **NothingYouCouldDo** | `NothingYouCouldDo-Regular.ttf` | 手写草书，自然随性 |
| **PixelifySans** | `PixelifySans-Medium.ttf` | 像素风，复古游戏感 |
| **Silkscreen** | `Silkscreen-Regular.ttf` | 像素点阵，8-bit 风格 |

---

## 字体搭配建议

### 经典搭配公式

| 场景 | 标题 | 正文 | 辅助 |
|------|------|------|------|
| 高端艺术海报 | Gloock | CrimsonPro | Italiana |
| 技术架构图 | BricolageGrotesque Bold | InstrumentSans | GeistMono |
| 商业信息图 | WorkSans Bold | WorkSans Regular | IBMPlexMono |
| 科技产品封面 | BigShoulders Bold | Jura | JetBrainsMono |
| 小红书配图 | YoungSerif / Outfit Bold | WorkSans | — |
| 极简海报 | Gloock | PoiretOne | — |

---

## 中文内容处理

**⚠️ 重要**：`canvas-fonts/` 目录中的字体均为英文字体，不支持中文字符渲染。

中文内容处理规则：
1. **英文标题 + 中文正文**：标题用 `canvas-fonts/` 字体，中文正文用系统字体栈
2. **纯中文内容**：使用系统字体栈，在代码中指定：
   ```python
   # matplotlib / PIL
   font = ImageFont.truetype("/System/Library/Fonts/PingFang.ttc", size)
   # 或使用 macOS 系统中文字体
   # /System/Library/Fonts/STHeiti Light.ttc（黑体-简）
   # /System/Library/Fonts/Hiragino Sans GB.ttc（冬青黑体）
   ```
3. **混排设计**：英文元素用 `canvas-fonts/` 字体增加设计感，中文部分用系统字体保证可读性

---

## 使用方式

```python
# 加载 canvas-fonts 字体示例
from PIL import ImageFont
font_path = "./canvas-fonts/Gloock-Regular.ttf"
font = ImageFont.truetype(font_path, size=48)
```
