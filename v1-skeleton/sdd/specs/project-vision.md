# AI 知识库 · 项目愿景 v1.0

## 要做什么

### 1. 数据采集
- 抓取 **GitHub Trending 全语言**，由 Agent 判断是否 **AI 相关**（不依赖 GitHub topic 过滤）
- 每日采集上限 **Top 25**（AI 相关的 Top 25，非全量 Top 25）
- 架构上预留多数据源接口（Hacker News、arXiv），v1 仅接 GitHub Trending

### 2. Agent 分析
- **引擎**：LLM API（OpenAI / Claude 等）
- **输入**：repo 描述 + star 数
- **处理**：
  - 判断是否 AI 相关（首轮过滤）
  - 打标签/分类（所属技术领域）
  - 分析技术难度
  - 分析创新点
  - 评估置信度
- **输出**：判断值不值得学习 + 精简要点摘要

### 3. 知识条目输出
- **主存储**：每天一个 JSON 文件（`data/YYYY-MM-DD.json`），独立、按日期命名
- **阅读层**：每天一个 Markdown digest（`output/YYYY-MM-DD.md`），供人直接阅读
- **去重机制**：同一 repo 不重复入库（跨天比对 `repo_name`），记录 `first_seen_date` 和 `trending_days` 计数
- **条目字段**：
  - `repo_name` - 仓库名
  - `description` - 描述
  - `stars` - star 数
  - `tags` - 技术标签/分类（数组）
  - `difficulty` - 技术难度：`beginner` / `intermediate` / `advanced`
  - `difficulty_reason` - 难度判定理由（一句话）
  - `innovation` - 创新点（简述）
  - `confidence` - 分析置信度：`high` / `medium` / `low`
  - `confidence_reason` - 置信度判定理由（一句话）
  - `recommendation` - 学习推荐度：`must-learn` / `worth-a-look` / `skip`
  - `recommendation_reason` - 推荐理由（一句话）
  - `summary` - 精简要点摘要
  - `first_seen_date` - 首次发现日期（YYYY-MM-DD）
  - `trending_days` - 连续 trending 天数（整数）

## 不做什么
- ❌ 深度代码分析（clone 源码、跑测试）
- ❌ 个性化推荐（根据个人技术栈筛选）
- ❌ 社区互动功能（评论、收藏、分享）

## 要做（但非 v1）
- 🕐 多数据源接入（Hacker News、arXiv）
- 🕐 Agent 审核流程完善，持续提高分析可信度
- 🕐 cron 定时调度（v1 手动触发）

## 边界 & 验收
| 项 | 说明 |
|---|---|
| v1 数据源 | 仅 GitHub Trending（架构预留扩展接口） |
| AI 判定方式 | 抓全语言 trending → Agent 判断是否 AI 相关 |
| Agent 引擎 | LLM API（OpenAI / Claude） |
| 运行方式 | 本地 Python 脚本，手动触发 |
| 输出位置 | 本地文件：`data/YYYY-MM-DD.json` + `output/YYYY-MM-DD.md` |
| 多语言 | 支持分析非英文 repo |

## 怎么验证
- 执行 `python main.py`，脚本正常退出
- 本地生成当日 `data/YYYY-MM-DD.json` + `output/YYYY-MM-DD.md`
- JSON 包含 ≤ 25 条去重后的 AI 相关 repo 条目
- 每条包含完整字段，枚举字段取值合法（difficulty ∈ {beginner, intermediate, advanced} 等）
- 首版接受少量分析不准确，后续通过 Agent 迭代完善审核流程

