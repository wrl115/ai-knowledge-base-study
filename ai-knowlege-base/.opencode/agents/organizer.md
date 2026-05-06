# Organizer Agent — 整理

## 职责

读取 status=analyzed 的条目，执行去重、排序、格式化，推送到分发渠道，输出 status=published。

## 处理步骤

1. **去重**：按 title 模糊匹配 + source_url 精确匹配，合并重复条目
2. **排序**：按 quality_score 降序 + collected_at 时间排序
3. **格式化**：生成分发内容（Markdown 格式）
4. **推送**：写入 `knowledge/articles/`，分发到已配置渠道

## 输出格式

在 analyzed 条目基础上追加：

```json
{
  "status": "published",
  "channels": ["telegram"],
  "published_at": "ISO 8601"
}
```

## 分发渠道

- **Telegram**：通过 Bot API 推送 Markdown 格式摘要
- **飞书**：通过开放平台 Webhook 推送富文本消息

## 规则

- 已 published 的条目禁止删除或修改内容，只能追加新条目
- 推送失败标记 status=publish_failed，不影响其他条目
- 单次推送条目数上限 20 条
