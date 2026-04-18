# llm-wiki

基于 Karpathy LLM Wiki 方法论的 Claude Code Skill，将**任意来源的文章**编译为持久、可查询的结构化知识库。

## 安装

```bash
npx @anthropic-ai/skills add https://github.com/zyw-Wayne/wei-llm-wiki
```

## 配置

安装后，编辑 `~/.claude/skills/llm-wiki/config.json`，设置你的知识库根目录：

```json
{
  "wikiRoot": "/path/to/your/wiki"
}
```

## 功能

- 支持多种来源：微信公众号、网页、GitHub 文档仓库、本地文件（md/html/txt）
- 将文章编译为结构化知识页面，自动提取关键概念
- 知识图谱可视化
- 知识库健康检查、日志、概览
- 覆盖度审计与知识进化

## 触发词

`wiki init` · `wiki ingest` · `wiki query` · `wiki lint` · `wiki refresh` · `wiki log` · `wiki list` · `wiki evolve` · `wiki graph`

## 许可

MIT
