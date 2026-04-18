---
name: wei-llm-wiki
description: >
  当用户要求把文章存进知识库、查询知识库、或维护知识库时使用。
  触发词：wiki init、wiki ingest、wiki query、wiki lint、wiki refresh、wiki log、wiki list、wiki evolve、wiki graph、
  构建知识库、编译文章、公众号文章入库、本地文章入库、网页入库、GitHub仓库入库、文章知识库、article ingest、
  知识库评估、覆盖度审计、知识库进化、进化面板。
  支持所有来源：微信公众号 URL、普通网页 URL、GitHub 文档型仓库、本地文件（md/html/txt）。
  不适用于：项目内部的 .omc/wiki 知识库（那是 oh-my-claudecode:wiki）、单纯的文件下载（用 wechat-article-down）。
---

# LLM Wiki

基于 Karpathy LLM Wiki 方法论，将**任意来源的文章**编译为持久、可查询的结构化知识库。

## 触发条件

**使用场景**：
- 用户给出一个或多个文章链接（微信公众号、博客、技术文档等），要求存入/编译进知识库
- 用户给出 GitHub 仓库链接（文档型仓库，如书籍、教程、知识库），要求整仓库摄入
- 用户给出本地文件路径，要求入库
- 用户要查询已有知识库的内容（"知识库里有什么关于 X 的？"）
- 用户要求对知识库做健康检查、刷新元信息、查看日志或概览

**不使用场景**：
- 仅下载微信文章而不编译入库 → 使用 `wechat-article-down`
- 操作项目级 `.omc/wiki/` 知识库 → 使用 `oh-my-claudecode:wiki`
- 普通的网页浏览或信息搜索 → 使用 `web-access`

## 调用路由

用户不需要显式指定技能名称。以下裸命令直接路由到本技能：

| 用户输入 | 路由到 | 说明 |
|----------|--------|------|
| `wiki init <路径>` | **llm-wiki** | 指定知识库根目录，后续所有操作默认使用该路径 |
| `wiki ingest <URL或路径>` | **llm-wiki** | 根据 URL 类型自动分派获取方式 |
| `wiki query "问题"` | **llm-wiki** | 查询知识库 |
| `wiki lint` | **llm-wiki** | 健康检查 |
| `wiki refresh` | **llm-wiki** | 刷新 WIKI.md 元信息 |
| `wiki log` | **llm-wiki** | 聚合操作日志 |
| `wiki list` | **llm-wiki** | 精简知识概览 |
| `wiki evolve` | **llm-wiki** | 更新所有已评估分类的进化档案 |
| `wiki evolve <分类名>` | **llm-wiki** | 审计指定分类的覆盖度 |
| `wiki evolve --all` | **llm-wiki** | 全量评估所有分类（含未评估的） |
| `wiki graph` | **llm-wiki** | 部署知识图谱可视化 HTML 到知识库根目录 |
| `把这篇文章入库` + URL | **llm-wiki** | 自然语言触发 |
| `知识库里有什么关于X的` | **llm-wiki** | 自然语言查询 |
| `知识库评估` / `覆盖度审计` | **llm-wiki** | 自然语言触发 evolve |

**与其他 wiki 技能的关系**：
- `llm-wiki` 是统一入口，已合并 `wechat-wiki` 和 `article-wiki` 的全部能力
- `wechat-wiki` / `article-wiki` 仍可通过 `/wechat-wiki` 或 `/article-wiki` 显式调用，但裸 `wiki` 命令统一走 `llm-wiki`
- `oh-my-claudecode:wiki` 是完全不同的技能，用于项目级 `.omc/wiki/` 知识库，不会与本技能冲突

**Ingest 内部路由**（根据输入自动判断获取方式）：

```
wiki ingest <来源>
     │
     ├─ https://mp.weixin.qq.com/...          → 调用 wechat-article-down 技能下载
     ├─ https://github.com/{owner}/{repo}...  → GitHub MCP 扫描+批量读取（仅文档型仓库）
     ├─ https://... 或 http://...（其他）      → 用 chrome-devtools 抓取网页正文
     └─ 本地路径（/、~/、./）                   → 用 Read 工具读取文件
```

## 核心哲学

RAG 每次查询都从原文重新推导知识。LLM Wiki 不同——LLM 预先把文章读完、理解完、整合进结构化 wiki，查询时读的是已编译的笔记，不是原文。每篇新文章入库都让整个 wiki 更丰富；每个好答案存回 wiki 都成为新的知识节点。知识复利积累，不是每次重新推导。

**分工**：用户负责筛选文章来源、提问、判断方向；LLM 负责所有维护工作——摘要、交叉引用、归档、更新、检查矛盾。

**支持的来源类型**：
- 微信公众号文章（`https://mp.weixin.qq.com/...`）→ 调用 `wechat-article-down` 技能下载
- GitHub 文档型仓库（`https://github.com/{owner}/{repo}`）→ GitHub MCP 扫描+批量读取，整仓库合并为一篇文章
- 非微信在线文章（其他 `http(s)://...`）→ 用 chrome-devtools 抓取
- 本地文件（`.md`、`.html`、`.txt` 等）→ 用 Read 工具读取

## WIKI.md 的两种类型

WIKI.md 有两种 `type`，通过文件开头的 `**类型**:` 字段区分：

| 类型 | 含义 | 所在目录 |
|------|------|----------|
| `wiki` | 单个知识库的 schema | 知识库根目录（含 `raw/`、`wiki/`） |
| `collection` | 多知识库注册表 | 包含多个知识库子目录的父目录 |

**兼容旧格式**：若 WIKI.md 中没有 `**类型**:` 字段，则通过目录结构推断——同级存在 `raw/` 或 `wiki/` 目录视为 `type: wiki`，否则检查是否有子目录含 WIKI.md 来判断是否为 collection。

## Wiki-Root 定位规则（按优先级）

执行任何操作前，必须先确定 wiki-root。按以下顺序检测，命中即停：

1. **用户显式指定**：命令中给出了路径（如 `wiki ingest ~/my-wiki ...`）→ 使用该路径
2. **持久化配置**：检查本技能目录下的 `config.json` 文件（即 `SKILL.md` 同级目录），若其中存在 `wikiRoot` 字段且该路径存在 → 使用该路径。此配置由 `wiki init` 设置/更新，持久保存知识库根目录
3. **当前目录有 WIKI.md**：读取并检查类型：
   - `type: wiki` → 当前目录就是 wiki-root，直接使用
   - `type: collection` → 读取其中的子知识库列表：
     - **1 个子知识库** → 自动使用，告知用户
     - **多个子知识库** → 列出让用户选择
     - 用户也可选择「创建新知识库」→ 创建新子目录并注册到 collection
4. **扫描子目录**：当前目录无 WIKI.md，但在直接子目录中查找 `*/WIKI.md`：
   - 找到 **1+ 个** → 自动创建 collection 类型的 WIKI.md 注册已有知识库（使用 `references/collection-schema-template.md` 模板），然后按第 3 步的 collection 逻辑处理
   - 找到 **0 个** → 进入第 5 步
5. **均未命中**：询问用户是要在当前目录创建新知识库（type: wiki），还是指定其他路径；确认后再初始化

> **关键**：绝不跳过探测直接创建默认知识库。用户可能在知识库内、在 collection 目录中、或在包含知识库子目录的父目录中。

### 新建子知识库（在 collection 下）

在 collection 目录中创建新知识库时：
1. 询问用户知识库名称和主题
2. 创建子目录及 `raw/`、`wiki/` 结构
3. 用 `references/wiki-schema-template.md` 生成子目录的 WIKI.md（type: wiki）
4. 更新父目录 WIKI.md（type: collection）的子知识库列表

## 目录结构

```
<wiki-root>/           # 按上述规则定位
├── raw/               # 原始文章（LLM 只读不写，除了初次存入）
│   └── <文章标题>/
│       ├── article.md
│       └── images/    # 文章中的图片（本地副本，仅在线文章需要）
├── wiki/              # LLM 编译生成的知识页面（LLM 全权维护）
│   ├── index.md       # 内容目录：每页一行链接+摘要，查询时首先读此文件
│   ├── log.md         # 操作日志：追加式，记录每次 ingest/query/lint/evolve
│   └── *.md           # 各概念/主题/分析/查询存档页面
├── evolve/            # 知识进化追踪（LLM 维护）
│   ├── index.md       # 总控面板：所有分类评分 + 下一步建议
│   └── <分类名>.md    # 每个主题的进化档案（覆盖度评估 + 进化向量 + 日志）
├── knowledge-graph.html  # 交互式知识图谱可视化（由 graph/init 命令部署）
└── WIKI.md            # 知识库 schema：结构约定、工作流规则（与用户协同演化）
```

## 八种操作

### Init（指定知识库目录）

将指定路径持久保存为知识库根目录，后续所有操作自动使用该路径，无需每次指定。

**步骤：**
1. 用户提供路径（如 `wiki init ~/my-wiki`）
2. **路径验证**：
   - 路径不存在 → 询问用户是否创建目录结构（创建后继续）
   - 路径存在但无 WIKI.md → 询问是否初始化为新知识库（按标准初始化流程创建）
   - 路径存在且有 WIKI.md → 直接使用
3. **写入持久化配置**：将路径（绝对路径或 `~` 开头的路径）写入本技能目录下的 `config.json` 文件（即 `SKILL.md` 同级目录），格式为 `{ "wikiRoot": "<路径>" }`
4. **部署知识图谱**：将本技能自带的 `references/knowledge-graph.html` 复制到 `<wiki-root>/knowledge-graph.html`（与 `raw/`、`wiki/`、`WIKI.md` 同级，若已存在则覆盖更新）
5. **输出确认**：显示当前设置的知识库根目录路径、知识库基本信息（文章数、页面数等），提示用户可用浏览器打开 `knowledge-graph.html` 查看知识图谱

**后续效果**：写入 config.json 后，Wiki-Root 定位规则的第 2 步会检测该文件。之后在任意目录执行 `wiki ingest`、`wiki query` 等操作时，都会自动使用该路径作为知识库根目录，无需重复指定。

### Ingest（摄入编译）

用户提供文章来源（URL 或本地文件路径），获取并编译进知识库。建议逐篇摄入，保持参与感。

**步骤：**
1. **定位 wiki-root**：按「Wiki-Root 定位规则」确定路径。若已存在 `WIKI.md`，读取它确认结构，直接进入第 2 步；仅当确认为首次创建时才初始化——创建目录结构，读取**本技能自带**的 `references/wiki-schema-template.md`，复制内容到 `<wiki-root>/WIKI.md`，询问用户知识库主题后填入主题字段

2. **获取文章内容**：根据输入类型分支处理——

   **A. 微信公众号 URL**（`https://mp.weixin.qq.com/` 开头）：
   1. 调用 `wechat-article-down` 技能，**指定 `--output <wiki-root>/raw/`**，使文章直接下载到 `raw/` 目录
   2. 已存在同名子目录则跳过

   **B. GitHub 仓库 URL**（`https://github.com/{owner}/{repo}` 开头）：
   详见下方「GitHub 仓库 Ingest 流程」。整仓库合并为一个 `raw/<仓库标题>/article.md`。

   **C. 非微信在线 URL**（其他 `http://...` 或 `https://...`，非 GitHub）：
   1. 使用 `mcp__chrome-devtools__navigate_page` 打开页面
   2. 等待页面加载完成（使用 `mcp__chrome-devtools__evaluate_script` 检查 `document.readyState`）
   3. 滚动页面触发懒加载（执行滚动脚本）
   4. 提取页面正文内容（执行提取脚本，见下方「通用网页提取脚本」）
   5. 从提取结果中获取标题和正文 Markdown
   6. 创建 `raw/<标题>/article.md`，写入内容（若已存在同名子目录则跳过）
   7. **下载图片**（见下方「图片下载流程」）：提取正文中所有图片 URL，下载到 `raw/<标题>/images/`，并将 article.md 中的远程 URL 替换为本地相对路径

   **C. 本地文件**（路径以 `/`、`~/`、`./` 开头，或是已知存在的文件路径）：
   1. 用 Read 工具读取文件内容
   2. 判断格式：
      - `.md` 文件：直接使用内容
      - `.html` 文件：提取正文文本，转为 Markdown
      - `.txt` 文件：直接使用内容
      - 其他文本格式（`.rst`、`.org` 等）：转为 Markdown
   3. 从文件内容中提取或推断标题（优先用文件首行标题，其次用文件名）
   4. 创建 `raw/<标题>/article.md`，写入内容（若已存在同名子目录则跳过）

3. **编译知识**：读取新文章 `raw/*/article.md`，对每篇：
   - 提取核心概念、观点、实体、要点
   - 更新或新建 `wiki/*.md` 页面（格式见本技能自带的 `references/page-format.md`）
   - 发现矛盾用 `> ⚠️ 矛盾：` 标注，不强行统一
4. **更新 index.md**：每个页面一行（链接 + 一句话摘要），按主题分组
5. **追加 log.md**：格式 `## [YYYY-MM-DD] ingest | 文章标题`，记录触及的页面列表

#### 输入类型判断规则

| 模式 | 判定为 | 处理方式 |
|------|--------|----------|
| `https://mp.weixin.qq.com/...` | 微信公众号 URL | 调用 `wechat-article-down` 下载 |
| `https://github.com/{owner}/{repo}...` | GitHub 仓库 | GitHub MCP 扫描+批量读取（见下方详细流程） |
| `http://...` 或 `https://...`（非微信、非 GitHub） | 通用在线 URL | 用 chrome-devtools 抓取 |
| `/`、`~/`、`./` 开头 | 本地文件路径 | 用 Read 工具读取 |
| 无前缀但文件存在 | 本地文件路径 | 用 Read 工具读取 |

#### article.md 写入格式

无论来源类型，写入 `raw/<标题>/article.md` 时统一格式：

```markdown
---
title: 文章标题
source: 来源路径或 URL
type: wechat | web | github | local
repo: owner/repo              # 仅 type: github 时填写
files: 25                     # 仅 type: github 时填写，合并的文件数
ingested: YYYY-MM-DD
---

（正文 Markdown 内容）
```

#### 通用网页提取脚本

在浏览器中执行以下脚本提取正文（通过 `mcp__chrome-devtools__evaluate_script`）：

```javascript
(() => {
  const title = document.title || document.querySelector('h1')?.textContent || 'Untitled';

  // 尝试常见正文容器
  const selectors = [
    'article', '[role="main"]', 'main',
    '.post-content', '.article-content', '.entry-content',
    '.content', '.post-body', '.article-body',
    '#content', '#article', '#post-content'
  ];

  let contentEl = null;
  for (const sel of selectors) {
    const el = document.querySelector(sel);
    if (el && el.textContent.trim().length > 200) {
      contentEl = el;
      break;
    }
  }
  if (!contentEl) contentEl = document.body;

  // 递归提取为 Markdown
  function toMd(node, depth = 0) {
    if (node.nodeType === 3) return node.textContent;
    if (node.nodeType !== 1) return '';
    const tag = node.tagName.toLowerCase();

    // 跳过无关元素
    if (['script','style','nav','footer','header','aside','iframe','noscript'].includes(tag)) return '';
    if (node.getAttribute('role') === 'navigation') return '';

    const children = Array.from(node.childNodes).map(c => toMd(c, depth)).join('');

    switch(tag) {
      case 'h1': return `\n# ${children.trim()}\n`;
      case 'h2': return `\n## ${children.trim()}\n`;
      case 'h3': return `\n### ${children.trim()}\n`;
      case 'h4': return `\n#### ${children.trim()}\n`;
      case 'h5': return `\n##### ${children.trim()}\n`;
      case 'h6': return `\n###### ${children.trim()}\n`;
      case 'p': return `\n${children.trim()}\n`;
      case 'br': return '\n';
      case 'strong': case 'b': return `**${children.trim()}**`;
      case 'em': case 'i': return `*${children.trim()}*`;
      case 'code': return node.parentElement?.tagName.toLowerCase() === 'pre' ? children : `\`${children.trim()}\``;
      case 'pre': return `\n\`\`\`\n${children.trim()}\n\`\`\`\n`;
      case 'a': {
        const href = node.getAttribute('href');
        return href ? `[${children.trim()}](${href})` : children;
      }
      case 'img': {
        const src = node.getAttribute('src') || node.getAttribute('data-src') || '';
        const alt = node.getAttribute('alt') || '';
        return src ? `\n![${alt}](${src})\n` : '';
      }
      case 'li': return `- ${children.trim()}\n`;
      case 'blockquote': return `\n> ${children.trim().replace(/\n/g, '\n> ')}\n`;
      case 'hr': return '\n---\n';
      default: return children;
    }
  }

  const markdown = toMd(contentEl)
    .replace(/\n{3,}/g, '\n\n')
    .trim();

  return JSON.stringify({ title: title.trim(), markdown, url: location.href });
})()
```

#### 滚动脚本

```javascript
(async () => {
  const delay = ms => new Promise(r => setTimeout(r, ms));
  const step = Math.floor(window.innerHeight * 0.8);
  let y = 0;
  const max = document.body.scrollHeight;
  while (y < max) {
    y += step;
    window.scrollTo(0, y);
    await delay(300);
  }
  window.scrollTo(0, 0);
  await delay(500);
  return 'scrolled';
})()
```

#### 图片下载流程

对于在线 URL 抓取的文章，提取正文后需下载图片到本地：

1. **创建图片目录**：`mkdir -p raw/<标题>/images/`
2. **提取图片 URL**：从 article.md 中找到所有 `![...](http...)` 格式的图片引用
3. **下载图片**：使用 `curl` 带 `Referer` 头下载每张图片到 `images/` 目录

```bash
curl -sL -H "Referer: <文章URL>" -o "raw/<标题>/images/<序号>-<描述>.jpg" "<图片URL>"
```

4. **命名规则**：`<序号>-<简短描述>.<扩展名>`，如 `01-架构图.jpg`、`02-流程对比.png`
   - 序号两位数补零，按文中出现顺序编号
   - 描述从图片上下文推断（如所在章节标题）
   - 扩展名从 URL 或 Content-Type 推断（默认 `.jpg`）
5. **替换引用**：将 article.md 中的远程 URL 替换为本地相对路径 `images/<文件名>`
6. **CDN 注意事项**：
   - 知乎（`zhimg.com`）：需 `Referer` 为文章 URL
   - 其他 CDN：先尝试无 Referer 下载，403 则加 Referer 重试

**本地文件**摄入时：若 `.md` 或 `.html` 中引用了本地图片路径，将图片一并复制到 `raw/<标题>/images/` 并更新引用。

#### GitHub 仓库 Ingest 流程

仅适用于**文档型仓库**（书籍、教程、知识库等以 `.md` 文件为主体的仓库），不适用于常规代码仓库。全程使用 GitHub MCP 工具（`mcp__plugin_github_github__get_file_contents`），不依赖本地 git 命令。

**步骤：**

1. **URL 解析**：从 URL 中提取 `owner` 和 `repo`
   - 支持格式：`https://github.com/{owner}/{repo}`、`https://github.com/{owner}/{repo}.git`、`https://github.com/{owner}/{repo}/tree/{branch}`
   - 剥离 `.git` 后缀和 `/tree/...` 路径
   - 若 URL 中包含分支信息（`/tree/{branch}`），记录并传给 MCP 的 `ref` 参数；否则使用默认分支

2. **扫描仓库结构**（GitHub MCP）：
   - 调用 `get_file_contents(owner, repo, path="/")` 获取根目录
   - 对每个子目录并行调用 `get_file_contents(owner, repo, path="子目录/")` 获取内容列表
   - 递归扫描直到获得完整文件树（仅需扫描包含 `.md` 文件的目录）

3. **文档型仓库判断**：
   - ✅ 文档型：`.md` 文件占仓库文本文件总数 ≥ 50%，且不存在典型代码目录（`src/`、`lib/`、`pkg/`）或代码配置文件（`package.json`、`Cargo.toml`、`go.mod`、`pom.xml` 等）
   - ⚠️ 非文档型：不满足上述条件时，提示用户"这看起来是代码仓库，不适合整仓库摄入"，终止流程

4. **展示预览，等待用户确认**：
   ```
   📦 检测到 GitHub 仓库: lintsinghua/claude-code-book (main 分支)
     ✅ 文档型仓库（25 个 .md 文件，4 个分组目录）

     将摄入以下文件：
     ├─ 00-前言.md
     ├─ 第一部分-基础篇/（6 个文件）
     │  01-xxx.md, 02-xxx.md, ...
     ├─ 第二部分-核心系统篇/（8 个文件）
     ├─ ...

     已跳过：README.md, .gitignore, cover.png, assets/

     确认摄入？(Y/n)
   ```

   **自动过滤规则**（从预览列表中排除）：
   - 根目录的 `README.md`、`CONTRIBUTING.md`、`LICENSE*`、`CHANGELOG*`、`CODE_OF_CONDUCT*`
   - 配置文件：`.gitignore`、`.github/*`、`.vscode/*`、`node_modules/*`
   - 非文本文件：`*.png`、`*.jpg`、`*.gif`、`*.svg`、`*.ico`、`*.pdf`
   - 代码配置：`package.json`、`tsconfig.json`、`*.lock`
   - 内容过少的 `.md`（< 3 行，通常是占位符）
   - **子目录的 README.md 保留**（可能是章节导读，用户可在预览中手动排除）

5. **批量读取文件内容**（GitHub MCP）：
   - 对确认列表中的每个 `.md` 文件调用 `get_file_contents(owner, repo, path="文件路径")`
   - 尽量并行调用以提高速度
   - MCP 返回的文件内容为 base64 编码，需解码为 UTF-8 文本

6. **合并为 article.md**：
   - **排序规则**：按目录结构组织，目录内文件按数字前缀自然排序（natural sort：`2-xxx` 排在 `10-xxx` 前面），无数字前缀的文件按字母序排在末尾
   - **合并格式**：每个顶层目录作为一级标题，文件按原有标题层级嵌入，文件之间用 `---` 分隔
   - **标题推断**：优先用仓库的 `README.md` 中的第一个 `#` 标题，其次用仓库名
   - 写入 `raw/<仓库标题>/article.md`，frontmatter 如下：

   ```markdown
   ---
   title: 仓库标题（从 README 或仓库名推断）
   source: https://github.com/owner/repo
   type: github
   repo: owner/repo
   files: 25
   ingested: YYYY-MM-DD
   ---

   # 前言

   （00-前言.md 内容）

   ---

   # 第一部分 - 基础篇

   ## 01-xxx

   （文件内容）

   ## 02-xxx

   （文件内容）

   ---

   # 第二部分 - 核心系统篇
   ...
   ```

   若 `raw/<仓库标题>/` 已存在同名子目录，则跳过（与其他来源类型行为一致）。

7. **编译知识**：
   - 合并后文件 < 3000 行：一次性编译（复用 Ingest 步骤 3-5）
   - 合并后文件 ≥ 3000 行：按顶层目录分段编译，每段独立提炼知识页面，最后做一次整合 pass 补充跨章节的交叉引用和综合洞察

### Evolve（进化追踪）

对知识库分类进行覆盖度审计，识别进化向量，追踪知识成熟度变化。

**命令：**

| 用法 | 行为 |
|------|------|
| `wiki evolve <分类名>` | 审计指定分类（模糊匹配 index.md 中的分类名） |
| `wiki evolve` | 更新所有**已评估过**的分类（evolve/ 下已有文件的） |
| `wiki evolve --all` | 全量评估所有分类（含未评估的） |

**步骤：**
1. **定位 wiki-root**，读取 `wiki/index.md` 获取目标分类下所有页面列表
2. **扫描页面元信息**：读取每个 wiki 页面的标题、标签、来源数量（不读全文，控制 token）
3. **五维评估**：广度（覆盖范围）、深度（是否有源码/实战级内容）、实用性（是否可直接指导行动）、时效性（最近更新时间）、交叉引用（页面间互联程度）
4. **识别进化向量**：找出知识可生长的方向——缺什么主题、哪里可以更深、有没有过时内容
5. **写入 evolve 文件**：
   - 新分类 → 创建 `evolve/<分类名>.md`（含 frontmatter、覆盖度评估、进化向量、进化日志）
   - 已有分类 → 更新评分、向量状态（🔴→🟡→🟢）、追加进化日志
6. **更新 evolve/index.md**：更新对应行的评分、向量数、下一步建议
7. **追加 wiki/log.md**：`## [YYYY-MM-DD] evolve | <分类名> 覆盖度审计`

**进化向量格式**：

```markdown
### → 向量名称
**阶段**: 🔴 空白 → 🟡 有基础覆盖 → 🟢 成熟
**当前**: 🔴 空白
**价值**: 从 N → M（+X）
**线索**: [具体搜索方向或已知来源]
```

**进化日志格式**：

```markdown
| 日期 | 触发 | 事件 | 变化 |
|------|------|------|------|
| YYYY-MM-DD | evolve 主动 | 全面审计 | 建立基线 N 分 |
| YYYY-MM-DD | query 被动 | 查询"xxx"暴露盲区 | 新增 1 个向量 |
```

### Query（知识查询）

回答关于知识库内容的问题。

**步骤：**
1. 读取 `wiki/index.md`，定位相关页面（不遍历全部文件）
2. 读取相关 wiki 页面，综合给出结构化答案并注明来源页面
3. 根据问题选择合适输出格式：对比表、分析页、概念梳理、时间线等
4. 若回答有洞察价值（综合分析、对比结论、发现的连接），存为新页面：
   - 文件名：`query-<主题>.md`，如 `query-claude-vs-codex.md`
   - 追加 log.md：`## [YYYY-MM-DD] query | 问题摘要`
5. **被动进化评估（保守策略）**：回答完成后，判断是否出现以下**明确信号**：
   - 有重要子问题在知识库中**完全没有**对应页面（不是"不够深"，而是"完全没有"）
   - 回答时发现了页面间**未被记录的事实性矛盾**
   - 若命中任一信号：更新对应的 `evolve/<分类>.md`（新增进化向量或更新状态），并在回答末尾附一行提示：`💡 进化信号：本次查询发现「XXX」为知识盲区，已更新 evolve/<分类>.md`
   - 若未命中：静默，不触发，不消耗额外 token

### Lint（健康检查）

输出知识库健康报告，并追加 log.md：`## [YYYY-MM-DD] lint | 发现 N 个问题`

检查项：
- **矛盾**：不同页面关于同一事实的相互矛盾表述
- **过时内容**：被更新文章推翻的旧有结论（标记为 `[STALE]`）
- **孤儿页面**：未被 index.md 或任何页面引用的页面
- **概念缺页**：多处提到某概念但尚无独立 wiki 页面
- **缺失交叉引用**：页面中提到了其他 wiki 页面但未用 `[[]]` 链接
- **超大页面**：>500 行，建议拆分
- **数据空白**：建议补充调查的话题或来源

### Refresh（刷新 WIKI.md）

根据知识库当前实际状态，重新生成 `WIKI.md` 的统计信息和结构描述。适用于多次 ingest 后 WIKI.md 与实际内容脱节的情况。

**步骤：**
1. **定位 wiki-root**：按「Wiki-Root 定位规则」确定路径
2. **扫描实际状态**：
   - 统计 `raw/` 下的文章数量和标题列表
   - 统计 `wiki/` 下的知识页面数量、分类分布
   - 读取 `wiki/index.md` 获取当前分类结构
   - 读取 `wiki/log.md` 获取最近操作时间和操作次数统计
3. **重新生成 WIKI.md**：保留原有的工作流规则和页面格式约定不变，更新以下内容：
   - **统计概览**：文章总数、知识页面数、分类数、最近更新时间
   - **知识分类**：从 index.md 提取实际分类及每分类的页面数
   - **最近活动**：最近 5 次操作的精简摘要（从 log.md 提取）
   - **健康状态**：若最近有 lint 记录，附上最近一次 lint 的问题数
4. **若为 collection 类型**：同时更新父目录 WIKI.md 中该子知识库的描述行（含最新统计）
5. **追加 log.md**：`## [YYYY-MM-DD] refresh | 更新 WIKI.md 元信息`

### Log（聚合操作日志）

读取 `wiki/log.md`，输出精简的聚合摘要。去除冗余细节，每条操作压缩为一行，按日期分组。

**步骤：**
1. **定位 wiki-root**，读取 `wiki/log.md`
2. **解析所有操作条目**：每个 `## [日期] 操作类型 | 标题` 为一条
3. **聚合输出**，格式如下：

```
📊 知识库操作日志（共 N 条操作）

## 2026-04-14（1 条）
  ingest  踏马的 Agent → +2 页（Harness-Engineering, Context-Engineering）

## 2026-04-13（10 条）
  ingest  一文带你看懂Skills → +2 页, ~3 页
  query   CoT 横纵分析 → 存档 query-cot-横纵分析.md
  lint    发现 6 个问题，修复 4 处
  ingest  某博客文章 → +1 页, ~2 页（web 来源）
  ingest  本地笔记.md → +1 页（local 来源）

统计：ingest 15 次 | query 1 次 | lint 4 次 | refresh 0 次
```

**聚合规则：**
- 每条一行，格式：`操作类型  标题缩写 → 变更摘要`
- 标题超过 20 字截断加 `…`
- `+N 页` 表示新建页面数，`~N 页` 表示更新页面数
- 底部附操作类型统计汇总
- **不修改** log.md 原文，仅输出聚合视图

### List（精简知识概览）

输出知识库的精简概览，一屏可读，便于快速了解知识库全貌。

**步骤：**
1. **定位 wiki-root**，读取 `wiki/index.md`
2. **扫描补充信息**：统计 `raw/` 文章数、`wiki/` 页面数、最近更新时间
3. **输出格式**：

```
📚 知识库概览
   21 篇文章 → 23 个知识页面 | 3 个分类 | 最近更新: 2026-04-14

┌─ 知识管理（3 页）
│  LLM-Wiki · LLM-Wiki-核心思想 · RAG-vs-LLM-Wiki
│
├─ Agent 开发（14 页）
│  Harness-Engineering · Context-Engineering · 代码执行范式
│  Skill定义规范 · Skill实战踩坑指南 · Skill框架源码分析
│
└─ Prompt 工程（6 页）
   Prompt语言选择 · 横纵分析法 · Chain-of-Thought
```

**输出规则：**
- 分类从 index.md 提取，保持原有分组
- 每分类显示页面数 + 页面名列表（用 `·` 分隔，每行不超 70 字换行）
- 头部一行统计：文章数 → 页面数 | 分类数 | 最近更新
- 若为 collection 类型，先列出所有子知识库概览，再询问是否展开某个

### Graph（知识图谱可视化）

将知识库的交互式知识图谱 HTML 部署到 wiki 目录，用户可在浏览器中打开查看节点关系。

**步骤：**
1. **定位 wiki-root**：按「Wiki-Root 定位规则」确定路径
2. **部署 HTML**：读取本技能自带的 `references/knowledge-graph.html`，复制到 `<wiki-root>/knowledge-graph.html`（与 `raw/`、`wiki/`、`WIKI.md` 同级，若已存在则覆盖更新）
3. **输出确认**：告知用户文件路径，提示用浏览器打开后选择 `wiki/` 目录即可查看知识图谱

**说明：**
- HTML 文件使用浏览器 File System Access API，打开后需用户手动选择 `wiki/` 目录
- 自动解析 `index.md` 中的分类结构和所有 wiki 页面的 `[[wikilink]]` 关系
- 支持力导向图布局、分类着色、搜索过滤、节点详情、明暗主题切换
- `wiki init` 时会自动执行此步骤，无需单独调用；`wiki graph` 可随时手动更新

## 典型用法

```
# 指定知识库根目录（持久化，后续操作自动使用该路径）
wiki init ~/my-wiki
wiki init ~/Documents/个人知识库

# 摄入微信公众号文章
wiki ingest https://mp.weixin.qq.com/s/xxxxx

# 摄入非微信在线文章
wiki ingest https://blog.example.com/some-article

# 摄入 GitHub 文档型仓库（整仓库合并为一篇文章）
wiki ingest https://github.com/lintsinghua/claude-code-book

# 摄入 GitHub 仓库（指定分支）
wiki ingest https://github.com/owner/repo/tree/dev

# 摄入本地 Markdown 文件
wiki ingest ~/notes/my-research.md

# 摄入本地 HTML 文件
wiki ingest ./downloads/blog-post.html

# 批量摄入（逐篇处理，可混合来源类型）
wiki ingest https://mp.weixin.qq.com/s/aaa ~/notes/b.md https://blog.example.com/c

# 查询
wiki query "我写过哪些关于 AI 编程工具的文章？核心观点是什么？"
wiki query "Claude Code 和 Codex 有什么区别，各自适合什么场景？"

# 进化追踪
wiki evolve skills              # 审计 Skills 体系分类
wiki evolve                     # 更新所有已评估分类
wiki evolve --all               # 全量评估所有分类

# 部署/更新知识图谱可视化
wiki graph

# 健康检查
wiki lint

# 刷新 WIKI.md 元信息
wiki refresh

# 查看聚合操作日志
wiki log

# 查看精简知识概览
wiki list

# 指定知识库路径（路径放在来源前面）
# 不指定时自动探测：当前目录 WIKI.md → 子目录 */WIKI.md → 询问用户
wiki ingest ~/my-wiki https://mp.weixin.qq.com/s/xxx
wiki ingest ~/my-wiki ~/notes/research.md
wiki query ~/my-wiki "你的问题"
```

## 参考文件

- `references/wiki-schema-template.md`：单知识库 WIKI.md 模板（type: wiki），初始化时复制并定制
- `references/collection-schema-template.md`：多知识库集合 WIKI.md 模板（type: collection），自动注册子知识库
- `references/page-format.md`：wiki 页面格式规范与示例（含 index.md、log.md 格式）
- `references/knowledge-graph.html`：交互式知识图谱可视化模板，由 `wiki graph` 或 `wiki init` 部署到 `wiki/` 目录
