# 质量闭环设计规范

## 概述

质量闭环（Judge → 精确重做）是多 Agent 系统保证输出质量的核心机制。核心思想：**不是"整体重做"，而是"精确修复"**。

## 架构模式

```
Worker 输出产物（文章/图片/代码）
      │
      ▼
JudgeAgent 逐项评分
      │
      ├── 全部 pass → 结束
      └── 部分 fail → 精确重做 fail 项 → 重新评分
                        │
                        └── 最多 N 轮（避免无限循环）
```

## 评分设计

### 多维度评分 + 逐项 pass/fail

```json
{
  "D1": 4.5, "D2": 3.0, "D3": 4.0,
  "composite": 3.8,
  "feedback": "一句话改进建议",
  "global_issue": "order | content | null",
  "per_item": [
    {"index": 0, "pass": true},
    {"index": 1, "pass": false, "issue": "该项的具体问题描述"}
  ]
}
```

### 关键字段说明

| 字段 | 作用 |
|------|------|
| `composite` | 综合分，决定是否触发重做 |
| `global_issue` | 意图识别：是顺序问题（不需要重做内容）还是质量问题 |
| `per_item` | 逐项评判，精确定位哪些需要重做 |
| `feedback` | 传给 Worker 的改进方向 |

## 路由决策

```python
def decide_action(text_score, image_score):
    text_weak = [k for k, v in text_dims.items() if v < THRESHOLD]
    image_weak = [k for k, v in image_dims.items() if v < THRESHOLD]

    if not text_weak and not image_weak:
        return "pass"
    elif text_weak and image_weak:
        return "rewrite_and_illustrate"
    elif text_weak:
        return "rewrite_only"
    else:
        return "illustrate_only"
```

## 精确重做策略

### 策略 1：per_item 精确重做

只重做 `pass=false` 的项，其余直接复用：

```python
redraw_indices = [item["index"] for item in per_image if not item.get("pass", True)]
# 只传 redraw_indices 给 IllustratorAgent
# 不在列表中的图直接 shutil.copy2 复用
```

### 策略 2：global_issue 意图识别

区分"内容问题"和"排列问题"，避免误判：

```python
if global_issue == "order":
    # 图片顺序不对但内容本身没问题 → 不需要重绘
    pass
elif global_issue == "content":
    # 内容质量问题 → 精确重做 fail 项
    redraw(redraw_indices)
```

### 策略 3：feedback 精确传递

整体 feedback 拆分为 per-item 级别，传给 Worker：

```python
per_image_issues = {item["index"]: item.get("issue", "") for item in per_image if not item["pass"]}
image_feedback = "; ".join(f"图{idx}: {issue}" for idx, issue in per_image_issues.items())
```

## 防护机制

### 最大轮次限制

```python
MAX_REWRITE_ATTEMPTS = 2  # 最多重做 2 轮
for attempt in range(MAX_REWRITE_ATTEMPTS):
    result = worker.execute(...)
    score = judge.evaluate(result)
    if score.passed:
        break
# 超过最大轮次 → needs_human_review
```

### 评分不可用时的降级

```python
try:
    score = judge.evaluate(result)
except Exception:
    # Judge 评分失败不阻断流程，标记为需人工审核
    score = None
    status = "needs_human_review"
```

### 避免"好图被误杀"

per_item 全部 pass 但整体分低时，跳过重做：

```python
if all(item.get("pass", True) for item in per_image):
    # 所有单项都 pass，整体分低可能是主观偏差
    print("per_image 全部 pass 但整体扣分，跳过重绘")
```

## 实战教训

| 问题 | 解决方案 |
|------|---------|
| 全量重做导致好结果变差 | per_item 精确定位 + 复用好结果 |
| Judge 和 Worker 用同一个模型导致"自己给自己打高分" | Judge 和 Worker 用不同模型 |
| 重做后分数反而更低 | 设置"分数不升则停"的刹车机制 |
| 无限循环卡死 | 最大轮次 + 超时兜底 |
| thinking 模型做 Judge 太慢 | VL 评分用非 thinking 模型，或加大 timeout |
