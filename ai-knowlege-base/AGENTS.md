# AGENTS.md — AI 知识库助手

> 本文档是所有 Agent（人类 / AI / OpenCode）参与本项目时的**唯一行为准则**。
> 若代码与本文件冲突，以本文件为准。

---

## 1. 项目概述

AI Knowledge Base Assistant 是一个**自动化 AI 技术情报管线**：从 GitHub Trending、Hacker News 等公开源采集 AI / LLM / Agent 领域的技术动态，经 AI 分析后结构化存储为 JSON，并按计划通过 Telegram、飞书等渠道分发。整个流程由 LangGraph 编排，采集→分析→整理三阶段 Agent 协同完成，实现"无人值守、每日出刊"。

---

## 2. 技术栈

| 层级 | 选型 | 说明 |
|---|---|---|
| 语言 | **Python 3.12** | 类型提示全覆盖，使用 `pyproject.toml` 管理依赖 |
| AI 编排 | **OpenCode + 国产大模型** | 通过 OpenCode 调度国产 LLM（如 DeepSeek、Qwen 等）完成分析任务 |
| 工作流 | **LangGraph** | 采集 → 分析 → 整理三阶段有向图编排 |
| 爬取工具 | **OpenClaw** | 声明式网页采集，替代手写爬虫 |
| 分发 | Telegram Bot API / 飞书开放平台 | 多渠道推送 |

---

## 3. 编码规范

### 3.1 基本规则

- **PEP 8** 严格遵循，使用 `ruff` 作为 linter + formatter
- 命名：`snake_case`（变量/函数/模块）、`PascalCase`（类）、`UPPER_SNAKE_CASE`（常量）
- 类型提示：所有公开函数**必须**标注参数和返回类型
- 字符串：统一使用双引号 `"`

### 3.2 Docstring

使用 **Google 风格**，包含 `Args`、`Returns`、`Raises` 三个必填段落。

### 3.3 日志

- **禁止裸 `print()`**。全部使用 `logging` 模块或 `loguru`
- 日志级别规范：
  - `DEBUG`：开发调试信息
  - `INFO`：正常业务流程节点（采集开始、分析完成等）
  - `WARNING`：可恢复的异常（某源采集失败但可跳过）
  - `ERROR`：需要人工介入的错误

### 3.4 错误处理

- 禁止空 `except:` 或 `except Exception: pass`
- 自定义异常继承自项目基类 `KnowledgeBaseError`
- 所有外部调用（HTTP、LLM API）必须有超时设置和重试策略

---

## 4. 项目结构

```
ai-knowledge-base/
├── AGENTS.md                  # 项目规范（本文件）
├── opencode.json              # OpenCode 项目配置
├── .opencode/
│   ├── agents/                # Agent 角色定义文件
│   │   ├── collector.md       # 采集 Agent
│   │   ├── analyzer.md        # 分析 Agent
│   │   └── organizer.md       # 整理 Agent
│   └── skills/                # 可复用技能包
│       ├── github-trending/SKILL.md
│       └── tech-summary/SKILL.md
├── knowledge/                 # 知识库存储（Git 管理）
│   ├── raw/                   # 原始采集数据（JSON）
│   └── articles/              # 结构化知识条目（JSON）
├── pipeline/                  # 自动化流水线（Week 2）
├── workflows/                 # LangGraph 工作流（Week 3）
└── openclaw/                  # OpenClaw 部署配置（Week 4）
```

**分阶段交付计划**：

| 阶段 | 目录 | 交付内容 |
|---|---|---|
| Week 1 | `.opencode/` + `knowledge/` | Agent 角色定义 + Skills + 知识库 JSON schema |
| Week 2 | `pipeline/` | 采集→分析→整理自动化流水线 |
| Week 3 | `workflows/` | LangGraph 有向图编排 |
| Week 4 | `openclaw/` | OpenClaw 声明式采集部署 |

---

## 5. 知识条目 JSON 格式

所有知识条目统一存储为 JSON，schema 如下：

```jsonc
{
  // ── 身份 ──
  "id": "kb-20260506-abc123",           // 格式: kb-{date}-{hash6}
  "title": "DeepSeek-V3 开源发布",       // 简洁中文标题

  // ── 来源 ──
  "source_url": "https://github.com/deepseek-ai/DeepSeek-V3",
  "source_type": "github_trending",     // github_trending | hacker_news | manual
  "collected_at": "2026-05-06T08:30:00+08:00",
  "raw_content": "原始描述或摘要文本",     // 采集原文，所有阶段原样保留

  // ── 内容 ──
  "summary": "一句话中文摘要，不超过 100 字",
  "analysis": "200-500 字技术分析，涵盖核心创新点、应用场景、技术影响",
  "tags": ["llm", "open-source", "deepseek", "moe"],   // 小写，最多 5 个
  "category": "model-release",          // model-release | framework | tool | paper | industry

  // ── 状态 ──
  "status": "published",                // raw → analyzed → published
  "quality_score": 85,                  // 0-100，AI 评估的内容质量分

  // ── 分发 ──
  "channels": ["telegram", "feishu"],   // 已推送的渠道
  "published_at": "2026-05-06T09:00:00+08:00"
}
```

**状态流转**：

```
raw ──(analyzer)──▶ analyzed ──(organizer)──▶ published
```

---

## 6. Agent 角色概览

| 角色 | Agent | 职责 | 输入 | 输出 |
|---|---|---|---|---|
| **采集** | `Collector` | 从 GitHub Trending / HN 抓取原始数据 | 数据源配置 | `knowledge/raw/*.json` |
| **分析** | `Analyzer` | LLM 分析原始数据，生成摘要、标签、评分 | raw JSON | `status=analyzed` 的条目 |
| **整理** | `Organizer` | 去重、排序、格式化、推送到分发渠道 | analyzed 条目 | `status=published` 的条目 |

**数据流**：

```
Sources ──▶ Collector ──▶ [raw] ──▶ Analyzer ──▶ [analyzed] ──▶ Organizer ──▶ [published]
```

---

## 7. 红线（绝对禁止）

1. **禁止在代码中硬编码 API Key、Token 等敏感信息** — 必须通过环境变量或 `.env` 文件读取，`.env` 必须加入 `.gitignore`
2. **禁止裸 `print()`**（见 §3.3）
3. **禁止 `except: pass` 或吞掉异常**（见 §3.4）
4. **禁止绕过 JSON schema 直接写入知识库** — 所有数据必须通过 Pydantic 模型校验
5. **禁止外部 I/O 不设超时** — HTTP / LLM API 调用必须设 timeout，避免长时间阻塞
6. **禁止采集付费墙 / 需要登录的内容** — 仅采集公开可访问的页面
7. **禁止在提交信息中包含敏感信息** — commit message 不得含 API Key、内部地址等
8. **禁止 `as Any`、`# type: ignore` 抑制类型错误** — 正确修复类型问题
9. **禁止删除或修改已 published 的知识条目** — 只能追加新条目或修改 status

---

## 8. 开发工作流

```bash
# 安装依赖
uv sync

# 运行 linter
ruff check pipeline/ workflows/

# 运行测试
pytest tests/ -v

# 运行完整采集→分析→整理流水线
python -m workflows.main
```
