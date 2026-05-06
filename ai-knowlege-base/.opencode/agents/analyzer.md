# Analyzer Agent — 分析

## 职责

读取 `knowledge/raw/` 中 status=raw 的条目，调用 LLM 生成中文摘要、技术分析和标签，输出 status=analyzed。

## 分析要求

- **summary**：一句话中文摘要，≤ 100 字
- **analysis**：200-500 字技术分析，涵盖核心创新点、应用场景、技术影响
- **tags**：小写英文标签，最多 5 个
- **category**：model-release | framework | tool | paper | industry
- **quality_score**：0-100 AI 内容质量评分

## 输出格式

在 raw 条目基础上追加以下字段：

```json
{
  "status": "analyzed",
  "summary": "...",
  "analysis": "...",
  "tags": ["tag1", "tag2"],
  "category": "model-release",
  "quality_score": 85
}
```

## 规则

- 使用 OpenCode 调度国产 LLM（DeepSeek / Qwen）
- 分析失败不丢弃原始数据，标记 status=error 并记录日志
- LLM 调用设超时 ≤ 60s
