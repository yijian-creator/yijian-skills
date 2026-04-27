# First-Time Setup

首次使用 yijian-canvas 时，未检测到 EXTEND.md，需要初始化用户偏好配置。

## 初始化流程

### 1. 询问用户

向用户提出以下问题（一次性，最多 3 个）：

| 问题 | 默认值 | 说明 |
|------|--------|------|
| **水印文字** | 无 | 显示在画布右下角的署名，如：你的名字或昵称 |
| **输出目录** | `~/Downloads` | 生成文件保存的根目录 |
| **语言** | `zh-CN` | 界面和输出语言 |

### 2. 创建配置文件

根据用户回答，在 `~/.yijian-skills/yijian-canvas/EXTEND.md` 创建配置：

```bash
mkdir -p "$HOME/.yijian-skills/yijian-canvas"
```

写入内容（以用户回答填充）：

```yaml
---
version: 1
watermark:
  enabled: true
  content: "{用户输入的水印文字}"
  position: bottom-right
  opacity: 0.7
default_output_dir: {用户输入的输出目录}
language: {用户选择的语言}
---
```

### 3. 确认并继续

```
✅ 偏好配置已保存至 ~/.yijian-skills/yijian-canvas/EXTEND.md
   水印：{content}（右下角，透明度 0.7）
   输出目录：{default_output_dir}
   语言：{language}

继续执行画布创建任务...
```

## 修改配置

用户可随时直接编辑 `~/.yijian-skills/yijian-canvas/EXTEND.md` 修改偏好。

配置字段完整说明见：[preferences-schema.md](preferences-schema.md)
