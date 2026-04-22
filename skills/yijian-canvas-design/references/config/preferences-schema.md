# Preferences Schema

`EXTEND.md` 的完整配置规范。

## 文件格式

```yaml
---
version: 1
watermark:
  enabled: true | false
  content: "水印文字"
  position: bottom-right | bottom-left | top-right | top-left | center
  opacity: 0.0 ~ 1.0
default_output_dir: /path/to/output
language: zh-CN | en-US
default_philosophy: ""   # 可选：默认设计哲学名称，留空则每次新建
canvas_size:
  preset: standard | poster-v | poster-h | custom
  figsize: [22, 16]      # 仅 preset=custom 时生效
---
```

## 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `version` | int | ✅ | 配置版本，当前为 1 |
| `watermark.enabled` | bool | ✅ | 是否启用水印 |
| `watermark.content` | string | ✅ | 水印文字（如：钧弈） |
| `watermark.position` | enum | ✅ | 水印位置 |
| `watermark.opacity` | float | ✅ | 透明度，0.0=不可见，1.0=完全不透明 |
| `default_output_dir` | path | ✅ | 输出根目录，文件保存到 `{dir}/canvas/{slug}/` |
| `language` | enum | ✅ | 界面和输出语言 |
| `default_philosophy` | string | ❌ | 默认复用的设计哲学，留空每次新建 |
| `canvas_size.preset` | enum | ❌ | 画布预设尺寸 |
| `canvas_size.figsize` | array | ❌ | 自定义画布尺寸 `[width, height]` |

## 预设尺寸

| preset | figsize | 适用场景 |
|--------|---------|----------|
| `standard` | [22, 16] | 架构图、信息图（默认） |
| `poster-v` | [16, 22] | 竖版海报 |
| `poster-h` | [22, 16] | 横版海报 |
| `custom` | 自定义 | 特殊需求 |

## 最小配置示例

```yaml
---
version: 1
watermark:
  enabled: true
  content: "钧弈"
  position: bottom-right
  opacity: 0.7
default_output_dir: /Users/junyi/Downloads
language: zh-CN
---
```
