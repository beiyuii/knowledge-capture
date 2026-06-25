# 写端适配器（vaults/）

写端适配器负责「按某个知识库自己的规则，把草稿整理进去」。与读端（adapters/）**完全对称**：

| | 读端 adapters/ | 写端 vaults/ |
|---|---|---|
| 适配对象 | AI agent 工具（zcode/Claude Code/…） | 知识库（Obsidian-KP/Logseq/Notion/…） |
| 产出 | Session IR | 正式知识卡片（进 vault 对应房间） |
| 一个文件 | 一个工具的解析规则 | 一个知识库的整理规则 |

## 职责边界

- ✅ 写端**只做整理决策 + 落盘格式**：判断草稿去哪个房间、按该知识库的 frontmatter 规范写。
- ❌ 写端**不做**：日志解析（读端做）、捕获什么（capture-rules 做）、对话循环（organize-rules 做）。
- ✅ 写端**必须含硬约束**：写入白名单、绝不碰该知识库的人工策展区。

## 当前适配器

| 文件 | 知识库 | Phase |
|---|---|---|
| `obsidian-knowledge-palace.md` | Obsidian + Knowledge Palace v2 | Phase 1 |

## 通用性

本技能的通用性体现在「能写哪种知识库」。当用户用别的知识库时：
- 加一个 `vaults/<name>.md`，描述它的房间结构 + frontmatter 规范 + 整理决策树。
- 记录会话 / 整理知识库流程不变（它们只认通用契约）。

## 如何新增一个 vault 适配器

1. 研究该知识库的组织规则（目录结构、frontmatter 约定、人工策展区在哪）。
2. 在 `vaults/<name>.md` 写：整理决策树（草稿→房间）、frontmatter 规范、写入白名单。
3. 与读端一样，纯 markdown 指令，AI 执行。
