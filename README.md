# wei-llm-wiki

**[English](#english) | [中文](#中文)**

---

<a name="english"></a>

## LLM Wiki — Compile Articles into a Structured Knowledge Base

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill based on [Karpathy's LLM Wiki methodology](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Instead of re-deriving knowledge from raw text every time (like RAG), LLM Wiki **pre-compiles** articles into structured, queryable wiki pages. Every new article makes the whole wiki richer. Every good answer gets saved back as a new knowledge node. **Compound knowledge, not repeated inference.**

### Install

```bash
npx skills add https://github.com/zyw-Wayne/wei-llm-wiki
```

### Quick Start

```bash
wiki init ~/my-wiki                              # Set wiki root (persisted)
wiki ingest https://example.com/article           # Ingest an article
wiki query "What are the key takeaways?"          # Query the knowledge base
```

### Multi-Source Ingestion

One command, auto-detects source type:

| Source | Example | Method |
|--------|---------|--------|
| WeChat Articles | `https://mp.weixin.qq.com/s/...` | `wechat-article-down` skill |
| GitHub Doc Repos | `https://github.com/owner/repo` | GitHub MCP scan + batch read |
| Web Pages | `https://blog.example.com/...` | chrome-devtools extraction + local images |
| Local Files | `~/notes/research.md` | Direct read (md/html/txt) |

```bash
# Mixed sources in one command
wiki ingest https://mp.weixin.qq.com/s/aaa ~/notes/b.md https://blog.example.com/c
```

### 8 Operations

| Command | Description |
|---------|-------------|
| `wiki init <path>` | Initialize wiki root, deploy knowledge graph HTML |
| `wiki ingest <source>` | Fetch article → store in `raw/` → compile into `wiki/` pages |
| `wiki query "question"` | Search wiki index → synthesize structured answer → auto-save insights |
| `wiki evolve [category]` | Audit knowledge coverage, track maturity with evolution vectors (🔴→🟡→🟢) |
| `wiki lint` | Health check: contradictions, stale content, orphan pages, missing cross-refs |
| `wiki graph` | Deploy interactive knowledge graph visualization (force-directed, themed, searchable) |
| `wiki refresh` | Regenerate WIKI.md metadata from actual wiki state |
| `wiki log` / `wiki list` | Aggregated operation log / one-screen knowledge overview |

### Knowledge Evolution

Not just storage — **active gap detection**.

```bash
wiki evolve agent         # Audit "Agent Development" category
wiki evolve               # Update all previously evaluated categories
wiki evolve --all         # Full evaluation of all categories
```

**5-dimensional assessment**: Breadth · Depth · Practicality · Timeliness · Cross-references

**Evolution vectors** track each knowledge direction: 🔴 Blank → 🟡 Basic coverage → 🟢 Mature, with specific search leads and suggestions.

**Passive evolution**: During `wiki query`, if knowledge gaps or factual contradictions are discovered, evolution profiles update automatically.

### Directory Structure

```
<wiki-root>/
├── raw/                      # Original articles (read-only)
│   └── <title>/
│       ├── article.md
│       └── images/           # Downloaded images from web articles
├── wiki/                     # LLM-compiled knowledge pages
│   ├── index.md              # Content directory (query entry point)
│   ├── log.md                # Operation log (append-only)
│   └── *.md                  # Concept / topic / query archive pages
├── evolve/                   # Knowledge evolution tracking
│   ├── index.md              # Dashboard: all category scores
│   └── <category>.md         # Per-topic evolution profile
├── knowledge-graph.html      # Interactive knowledge graph visualization
└── WIKI.md                   # Wiki schema
```

### Multi-Wiki Support

**Collection mode**: One parent directory manages multiple independent topic wikis. Auto-detects sub-wikis, generates a `collection` registry. Select target wiki interactively or specify path directly.

### Design Principles

- **Clear division**: You curate sources and ask questions; LLM handles all maintenance — summaries, cross-references, archival, updates, contradiction detection
- **Compound knowledge**: Good query answers are saved back to wiki as reusable knowledge nodes
- **Preserve contradictions**: Marked with `> ⚠️ Contradiction:`, never forcefully unified
- **Smart routing**: Auto-detects wiki-root (persisted config → current dir → subdirectory scan → ask user)

---

<a name="中文"></a>

## LLM Wiki — 将文章编译为结构化知识库

基于 [Karpathy LLM Wiki 方法论](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)的 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 技能。不同于 RAG 每次从原文重新推导知识，LLM Wiki **预编译**文章为结构化、可查询的 wiki 页面。每篇新文章让整个 wiki 更丰富，每个好答案存回 wiki 成为新的知识节点。**知识复利积累，不是每次重新推导。**

### 安装

```bash
npx skills add https://github.com/zyw-Wayne/wei-llm-wiki
```

### 快速开始

```bash
wiki init ~/my-wiki                              # 指定知识库根目录（持久化）
wiki ingest https://mp.weixin.qq.com/s/xxxxx      # 摄入一篇文章
wiki query "这篇文章的核心观点是什么？"              # 查询知识库
```

### 多来源摄入

一个命令，自动识别来源类型并分派获取方式：

| 来源类型 | 示例 | 获取方式 |
|---------|------|---------|
| 微信公众号 | `https://mp.weixin.qq.com/s/...` | 调用 `wechat-article-down` 技能下载 |
| GitHub 文档仓库 | `https://github.com/owner/repo` | GitHub MCP 扫描 + 批量读取，整仓库合并 |
| 普通网页 | `https://blog.example.com/...` | chrome-devtools 抓取正文 + 图片本地化 |
| 本地文件 | `~/notes/research.md` | 直接读取（支持 md/html/txt） |

```bash
# 混合来源批量摄入
wiki ingest https://mp.weixin.qq.com/s/aaa ~/notes/b.md https://blog.example.com/c
```

### 八大操作

| 命令 | 功能 |
|------|------|
| `wiki init <路径>` | 初始化知识库根目录，部署知识图谱 HTML |
| `wiki ingest <来源>` | 获取文章 → 存入 `raw/` → 编译为 `wiki/` 知识页面 |
| `wiki query "问题"` | 检索 wiki 索引 → 综合结构化回答 → 自动存档有价值的洞察 |
| `wiki evolve [分类]` | 覆盖度审计，用进化向量追踪知识成熟度（🔴→🟡→🟢） |
| `wiki lint` | 健康检查：矛盾、过时内容、孤儿页面、缺失交叉引用 |
| `wiki graph` | 部署交互式知识图谱可视化（力导向图、分类着色、搜索过滤） |
| `wiki refresh` | 根据实际状态重新生成 WIKI.md 元信息 |
| `wiki log` / `wiki list` | 聚合操作日志 / 一屏知识概览 |

### 知识进化

不只是"存文章"——**主动识别知识缺口**。

```bash
wiki evolve agent         # 审计「Agent 开发」分类的覆盖度
wiki evolve               # 更新所有已评估分类
wiki evolve --all         # 全量评估所有分类
```

**五维评估体系**：广度 · 深度 · 实用性 · 时效性 · 交叉引用

**进化向量**追踪每个知识方向的成熟度：🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟，并给出具体的搜索线索和补充建议。

**被动进化**：`wiki query` 过程中若发现知识盲区或事实矛盾，自动更新进化档案并提示。

### 目录结构

```
<wiki-root>/
├── raw/                      # 原始文章（只读）
│   └── <文章标题>/
│       ├── article.md
│       └── images/           # 在线文章的本地图片
├── wiki/                     # LLM 编译的知识页面
│   ├── index.md              # 内容目录（查询入口）
│   ├── log.md                # 操作日志（追加式）
│   └── *.md                  # 各概念/主题/查询存档页面
├── evolve/                   # 知识进化追踪
│   ├── index.md              # 总控面板：所有分类评分
│   └── <分类名>.md           # 每个主题的进化档案
├── knowledge-graph.html      # 交互式知识图谱可视化
└── WIKI.md                   # 知识库 schema
```

### 多知识库支持

**Collection 模式**：一个父目录管理多个独立的主题知识库。自动检测子目录中的知识库，生成 `collection` 类型的注册表。操作时自动列出供选择，也可直接指定路径。

### 核心设计

- **分工明确**：用户筛选来源、提问、判断方向；LLM 负责摘要、交叉引用、归档、更新、检查矛盾
- **知识复利**：好的查询答案存回 wiki，成为可复用的知识节点
- **矛盾保留**：用 `> ⚠️ 矛盾：` 标注，不强行统一，保留认知张力
- **被动进化**：查询中自动发现盲区，无需手动审计
- **智能路由**：自动检测 wiki-root（持久化配置 → 当前目录 → 子目录扫描 → 询问用户）

---

## License

MIT
