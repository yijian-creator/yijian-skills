# Dispatcher 意图识别设计规范

## 概述

Dispatcher 是 Orchestrator-Workers 架构的"大脑"，负责理解用户意图并输出结构化的 TaskPlan，告诉 Supervisor 调用哪些 Worker、传什么参数。

## 分层设计：规则层 + LLM 层

### 第一层：InputTypeDetector（规则判断，零 LLM 开销）

用正则/规则判断输入类型，决定某些 Worker 是否需要调用：

```python
def detect_input_type(user_input: str) -> dict:
    """规则判断输入类型，不消耗 LLM token"""
    import re
    url_pattern = re.compile(r'https?://\S+')
    urls = url_pattern.findall(user_input)
    if urls:
        return {"type": "url", "value": urls[0], "need_crawler": True}
    return {"type": "text", "value": user_input, "need_crawler": False}
```

**设计原则**：能用规则判断的，绝不浪费 LLM 调用。URL 识别、文件类型判断、指令前缀匹配等都属于规则层。

### 第二层：DispatcherAgent（LLM 语义意图分类）

规则层无法判断的语义差异（如"帮我画图" vs "帮我改写文章"），由 LLM 来分类：

```python
class DispatcherAgent:
    def dispatch(self, user_input, platform, human_insight) -> TaskPlan:
        input_info = detect_input_type(user_input)  # 规则层
        intent, extras = self._llm_detect_intent(user_input, input_info)  # LLM 层
        return self._build_task_plan(intent, input_info, extras)
```

## TaskPlan 结构

```json
{
  "intent": "full_pipeline | rewrite_only | illustrate_only | single_image | judge_only",
  "input_type": "url | text",
  "need_crawler": true,
  "sub_agents": ["CrawlerAgent", "RewriterAgent", "IllustratorAgent", "JudgeAgent"],
  "params": {
    "source_url": "https://...",
    "raw_text": "...",
    "platform": "xiaohongshu",
    "image_type": "structure",
    "redraw_index": 2
  }
}
```

## Dispatcher Prompt 设计

### 结构

```
系统 Prompt：
├── 角色定义（任务调度器）
├── 可用 Agent 能力（从注册表动态生成）
├── 意图分类规则（每种 intent 的触发条件）
└── 输出格式（严格 JSON schema）
```

### 关键设计点

1. **能力描述从注册表动态生成**，不硬编码在 prompt 里
2. **JSON schema 要声明所有字段**，包括条件字段（如 redraw_index 仅 single_image 时有值）
3. **给清晰的分类规则**，减少 LLM 的歧义空间

### 实战 Prompt 模板

```
你是一个任务调度器，负责理解用户意图并输出结构化 JSON 调用计划。

## 可用 Agent 能力
{从 AGENT_CAPABILITIES 动态生成}

## 意图分类规则
- full_pipeline：用户提供文章 URL 或长文本，希望生成完整的平台文章 + 配图
- rewrite_only：只需改写文章，不需要配图
- illustrate_only：用户直接提供了结构化内容（表格/列表/数据），只需要画图
- single_image：用户要求重绘某张具体的图
- judge_only：用户希望对已有内容进行评分

## 输出格式（严格 JSON）
{
  "intent": "意图类型",
  "image_content": "illustrate_only 时提取用户内容原文，否则 null",
  "image_type": "illustrate_only 时判断图表类型，否则 null",
  "redraw_index": "single_image 时提取图片编号，否则 null"
}
```

## 降级机制

LLM 可能返回格式错误或意图识别失败，必须有降级：

```python
try:
    parsed = json.loads(result)
    intent = parsed.get("intent", "full_pipeline")
    if intent not in VALID_INTENTS:
        intent = "full_pipeline"  # 未知意图降级为全链路
except (json.JSONDecodeError, Exception):
    intent = "full_pipeline"  # 解析失败降级为全链路
```

**核心原则**：系统永远不会因为意图识别错误而卡死或崩溃。

## 意图扩展

新增意图只需 3 步：
1. 在 `INTENT_AGENT_CHAINS` 里加一条链路
2. 在 Dispatcher prompt 的分类规则里加一条描述
3. 在 `run_from_input()` 里加一个 `elif intent == "xxx"` 分支
