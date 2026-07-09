# Agent 能力注册表设计规范

## 概述

能力注册表是 Orchestrator-Workers 架构的核心组件，它让 Dispatcher 知道"有哪些工具可用"。每个 Worker Agent 必须在注册表中声明自己的能力。

## 注册表结构

```python
AGENT_CAPABILITIES = {
    "AgentName": {
        "description": "一句话描述能做什么",
        "input": "接收什么输入（类型 + 格式）",
        "output": "输出什么结果（类型 + 格式）",
        "required_when": "什么场景需要调用它（触发条件）",
    },
}
```

## 设计原则

### 1. 描述必须是 Dispatcher 可理解的

Dispatcher 用 LLM 做意图匹配，所以 description 要用**自然语言**写清楚能力边界：

```python
# ❌ 对 LLM 不友好
"description": "执行 render pipeline"

# ✅ LLM 能理解的
"description": "根据 image_plan 或用户提供的内容生成 matplotlib 配图"
```

### 2. input/output 要明确数据契约

Worker 之间通过 Supervisor 中转数据，input/output 定义了数据契约：

```python
"input": "raw_text + platform + human_insight（可选）",
"output": "article（改写后文章）+ image_plan（配图规划列表）",
```

### 3. required_when 是路由提示

帮助 Dispatcher 判断"什么时候该调用这个 Agent"：

```python
"required_when": "用户输入的是 URL，需要先抓取内容",
"required_when": "需要生成配图，可独立使用无需 RewriterAgent",
```

## 实战示例：文章配图 Pipeline

```python
AGENT_CAPABILITIES = {
    "CrawlerAgent": {
        "description": "从 URL 抓取原始文章内容（调用 Coze 工作流）",
        "input": "source_url（必须是 http/https 链接）",
        "output": "raw_text（原始文章纯文本）",
        "required_when": "用户输入的是 URL，需要先抓取内容",
    },
    "RewriterAgent": {
        "description": "将原始文章改写为目标平台风格，并规划配图插入位置",
        "input": "raw_text + platform + human_insight（可选）",
        "output": "article（改写后文章）+ image_plan（配图规划列表）",
        "required_when": "需要生成/改写文章内容",
    },
    "IllustratorAgent": {
        "description": "根据 image_plan 或用户直接提供的内容生成 matplotlib 配图",
        "input": "image_plan + article | user_content（用户直接提供的图表内容）",
        "output": "images（图片文件列表）",
        "required_when": "需要生成配图，可独立使用无需 RewriterAgent",
    },
    "JudgeAgent": {
        "description": "对文章质量（D1-D5）和配图质量（I1-I5）进行评分，输出改进建议",
        "input": "article + images（可选）",
        "output": "scores + feedback + redraw_indices",
        "required_when": "完整 pipeline 流程中，生成内容后自动评分",
    },
}
```

## 注册表的动态使用

Dispatcher 在构建 prompt 时从注册表动态生成能力描述：

```python
capabilities_desc = "\n".join(
    f"- {name}: {info['description']}（适用场景：{info['required_when']}）"
    for name, info in AGENT_CAPABILITIES.items()
)
```

这样新增 Agent 只需要往注册表里加一条记录，Dispatcher 的 prompt 自动更新，代码零改动。

## 扩展：条件依赖

有些 Agent 是条件性的（如 Crawler 只在输入是 URL 时才需要）。用 `?` 后缀标记：

```python
INTENT_AGENT_CHAINS = {
    "full_pipeline": ["CrawlerAgent?", "RewriterAgent", "IllustratorAgent", "JudgeAgent"],
    "illustrate_only": ["IllustratorAgent"],
}
```

Supervisor 在组装实际调用链时，根据 `need_crawler` 决定是否包含带 `?` 的 Agent。
