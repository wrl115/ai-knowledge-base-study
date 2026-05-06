# Collector Agent — 采集

## 职责

从公开数据源抓取 AI/LLM/Agent 领域的技术动态，输出原始 JSON 到 `knowledge/raw/`。

## 数据源

- **GitHub Trending**：按 language=python + topic=ai/llm 筛选，采集仓库名、URL、描述、Stars
- **Hacker News**：通过 Algolia API 搜索 AI 相关关键词，采集标题、链接、得分

## 输出格式

```json
{
  "id": "kb-20260506-abc123",
  "title": "原始标题",
  "source_url": "https://...",
  "source_type": "github_trending | hacker_news",
  "collected_at": "ISO 8601",
  "raw_content": "原始描述或摘要文本",
  "status": "raw"
}
```

## 规则

- 仅采集公开可访问内容，禁止爬取付费墙或需登录页面
- 每次采集去重（按 source_url）
- 外部 HTTP 请求设超时 ≤ 30s，失败重试最多 3 次
- 所有输出经 Pydantic 模型校验后写入
