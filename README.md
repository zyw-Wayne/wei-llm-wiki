# wei-llm-wiki

基于 [Karpathy LLM Wiki 方法论](https://karpathy.ai/)的 Claude Code Skill —— 将**任意来源的文章**预编译为持久、可查询的结构化知识库。

> RAG 每次查询都从原文重新推导知识。LLM Wiki 不同 —— LLM **预先**把文章读完、理解完、整合进结构化 wiki，查询时读的是已编译的笔记，不是原文。每篇新文章入库都让整个 wiki 更丰富；每个好答案存回 wiki 都成为新的知识节点。**知识复利积累，不是每次重新推导。**

## 安装

```bash
npx skills add https://github.com/zyw-Wayne/wei-llm-wiki
```

## 快速开始

```bash
# 1. 指定知识库根目录（持久化，后续操作自动使用）
wiki init ~/my-wiki

# 2. 摄入一篇文章
wiki ingest https://mp.weixin.qq.com/s/xxxxx

# 3. 查询知识库
wiki query "这篇文章的核心观点是什么？"
```

## 多来源摄入

一个 `wiki ingest` 命令，自动识别来源类型并分派获取方式：

| 来源类型 | 示例 | 获取方式 |
|---------|------|---------|
| 微信公众号 | `https://mp.weixin.qq.com/s/...` | 调用 `wechat-article-down` 技能下载 |
| GitHub 文档仓库 | `https://github.com/owner/repo` | GitHub MCP 扫描 + 批量读取，整仓库合并为一篇 |
| 普通网页 | `https://blog.example.com/...` | chrome-devtools 抓取正文 + 图片本地化 |
| 本地文件 | `~/notes/research.md` | 直接读取（支持 md/html/txt） |

```bash
# 混合来源批量摄入
wiki ingest https://mp.weixin.qq.com/s/aaa ~/notes/b.md https://blog.example.com/c
```

### GitHub 仓库摄入

专为**文档型仓库**（书籍、教程、知识库）设计。自动扫描仓库结构，判断是否为文档型，展示文件预览供确认，按目录结构合并为单篇文章后编译入库。支持指定分支：

```bash
wiki ingest https://github.com/owner/repo/tree/dev
```

## 八大操作

### `wiki init` — 初始化

指定知识库根目录并持久化到 `config.json`。自动创建目录结构（`raw/`、`wiki/`、`evolve/`），生成 `WIKI.md` schema，部署交互式知识图谱 HTML。

### `wiki ingest` — 摄入编译

获取文章 → 存入 `raw/` → 提取核心概念/观点/实体 → 生成或更新 `wiki/` 页面 → 更新索引和日志。在线文章的图片会自动下载到本地。

### `wiki query` — 知识查询

先读 `wiki/index.md` 定位相关页面，综合给出结构化答案（对比表、分析页、时间线等）。有洞察价值的回答自动存为 `query-*.md` 知识页面。

**被动进化**：查询过程中若发现知识盲区或事实矛盾，自动更新进化档案并提示。

### `wiki evolve` — 进化追踪

知识库不只是"存文章"——它会**主动识别知识缺口**。

```bash
wiki evolve agent         # 审计「Agent 开发」分类的覆盖度
wiki evolve               # 更新所有已评估分类
wiki evolve --all         # 全量评估所有分类（含未评估的）
```

**五维评估体系**：广度、深度、实用性、时效性、交叉引用

**进化向量**追踪每个知识方向的成熟度：🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟，并给出具体的搜索线索和补充建议。

### `wiki lint` — 健康检查

检测矛盾表述、过时内容、孤儿页面、概念缺页、缺失交叉引用、超大页面、数据空白，输出健康报告。

### `wiki graph` — 知识图谱

部署交互式知识图谱可视化 HTML。浏览器打开后自动解析分类结构和 `[[wikilink]]` 关系，支持力导向图布局、分类着色、搜索过滤、明暗主题切换。

### `wiki refresh` — 刷新元信息

扫描知识库实际状态，重新生成 `WIKI.md` 中的统计概览（文章数、页面数、分类分布、最近活动）。

### `wiki log` / `wiki list` — 日志与概览

```bash
wiki log    # 聚合操作日志，按日期分组，底部统计汇总
wiki list   # 一屏可读的知识库全貌概览
```

`wiki list` 输出示例：

```
📚 知识库概览
   21 篇文章 → 23 个知识页面 | 3 个分类 | 最近更新: 2026-04-14

┌─ 知识管理（3 页）
│  LLM-Wiki · LLM-Wiki-核心思想 · RAG-vs-LLM-Wiki
│
├─ Agent 开发（14 页）
│  Harness-Engineering · Context-Engineering · 代码执行范式
│
└─ Prompt 工程（6 页）
   Prompt语言选择 · 横纵分析法 · Chain-of-Thought
```

## 目录结构

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

## 多知识库支持

支持 **Collection 模式**：一个父目录管理多个独立的主题知识库。自动检测子目录中的知识库，生成 `collection` 类型的 `WIKI.md` 注册表。操作时自动列出子知识库供选择，也可直接指定路径跳过。

## 核心设计

- **分工明确**：用户负责筛选文章来源、提问、判断方向；LLM 负责所有维护 —— 摘要、交叉引用、归档、更新、检查矛盾
- **知识复利**：好的查询答案存回 wiki，成为新的知识节点，供后续查询直接复用
- **矛盾保留**：发现矛盾用 `> ⚠️ 矛盾：` 标注，不强行统一，保留认知张力
- **被动进化**：查询过程中自动发现知识盲区，无需手动审计
- **智能路由**：自动检测 wiki-root（持久化配置 → 当前目录 → 子目录扫描 → 询问用户）

## 许可

MIT
