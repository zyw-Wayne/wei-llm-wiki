# Wiki 页面格式规范

## 标准页面结构

```markdown
---
title: 页面标题
tags: [tag1, tag2]
sources: [raw/文章标题/article.md]
updated: YYYY-MM-DD
---

# 标题

## 定义
一段话清晰定义这个概念/主题是什么。

## 核心观点
- 观点一（来源：[[来源文章名]]）
- 观点二

## 详细内容
（根据需要展开，可分小节）

## 矛盾与争议
> ⚠️ 矛盾：来源A说...，来源B说...，尚未统一。

## 相关概念
- [[相关页面1]] — 一句话说明关系
- [[相关页面2]]

## 来源
- [文章标题](../raw/文章标题/article.md) — 该文章提供了哪方面的信息
```

## index.md 结构

```markdown
# Wiki 索引

_最后更新：YYYY-MM-DD | 共 N 篇文章，M 个知识页面_

## 分类一

- [页面名](页面名.md) — 一句话摘要
- [页面名2](页面名2.md) — 一句话摘要

## 分类二
...

## 综合分析（Query 存档）
- [问题标题](query-xxx.md) — 查询日期 · 核心结论一句话
```

## log.md 结构

追加式，不修改已有条目。每条以 `## [日期] 操作类型 | 标题` 开头，便于 grep：

```markdown
# Wiki 操作日志

## [2026-04-12] ingest | 如果你还没学 RAG，可以不学了
- 新建：`LLM Wiki.md`、`RAG 对比.md`
- 更新：`Karpathy.md`（新增 LLM Wiki 章节）
- index.md 新增 3 条

## [2026-04-12] query | Claude Code 和 Codex 的适用场景对比
- 读取：`Claude Code.md`、`Codex.md`
- 存档：`query-claude-vs-codex.md`

## [2026-04-13] lint | 发现 4 个问题
- 孤儿页面：`旧工具评测.md`
- 概念缺页：提到 5 次"Skills 体系"但无独立页面
- 过时内容：`Cursor.md` 中定价信息已过时（标记 STALE）
```

grep 快速查看最近操作：`grep "^## \[" wiki/log.md | tail -10`

## 命名规范

| 类型 | 命名示例 |
|------|---------|
| 概念页 | `Claude Code.md` 或 `claude-code.md` |
| 工具/产品 | `Cursor.md` |
| Query 存档 | `query-claude-vs-codex.md` |
| 综合分析 | `analysis-ai-coding-tools.md` |

## 引用语法

- 内部链接：`[[页面名]]`（不含 .md）
- 来源引用（正文链接）：`[文章标题](../raw/文章标题/article.md)`（相对于 `wiki/` 目录，需要 `../` 前缀）
- frontmatter 中的 `sources` 字段：`[raw/文章标题/article.md]`（相对于 wiki-root，无需 `../`，仅作元数据记录）
- 引用原文：使用 blockquote `> 原文内容`

## 质量标准

- 每个页面聚焦单一概念，避免"大而全"
- 定义部分必须有，不超过 3 句话
- 来源必须可追溯到 raw/ 中的具体文章
- 超过 500 行考虑拆分
