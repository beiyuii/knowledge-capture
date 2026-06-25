# knowledge-capture

> **把你的 AI 对话变成永久知识。** 自动从 zcode / Claude Code / Codex 捕获会话日志进你的 Obsidian 知识库，再通过可视化看板报告，把值得留的草稿评审、晋升为正式笔记。

English: [README.md](./README.md)

[![version](https://img.shields.io/badge/version-0.1.0-blue)](./SKILL.md)
[![license](https://img.shields.io/badge/license-MIT-green)](./LICENSE)
[![category](https://img.shields.io/badge/category-capture%20%26%20review-purple)](#)
[![platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20WSL-lightgrey)](#)
[![agents](https://img.shields.io/badge/agents-zcode%20%7C%20Claude%20Code%20%7C%20Codex-orange)](#)

---

## 为什么需要它

你每天都在用 AI agent 干活——设计方案、修 bug、做决策。每一次会话都装满了来之不易的洞察。但有两件事总是出错：

**1. 这些工作不会自己进入你的知识库。**

你做完一次会话，窗口一关，就没了。你辩论过的决策、终于搞定的 bug、想出来的方法——除非你自己坐下来写，否则不会变成笔记。而没人能坚持这么做。于是你的知识库一直空着，真正的洞察却每天从聊天记录里漏掉。

**2. 你没有一个轻松的、定期的时刻，去决定知识库到底该放什么。**

就算你有个收件箱，看它也是件烦事——一堆原始碎屑，得一条条读、判断每条是什么、决定它该去哪。于是你一直拖。缓冲区越堆越多，知识永远落不了地。你的"第二大脑"变成了一个你不敢打开的坟墓。

knowledge-capture 同时解决这两个问题：

- **记录会话（save）在后台自动发生。** 它读你的 agent 会话日志，把真正值得记住的部分拉出来，丢进你知识库的收件箱变成草稿。你什么都不用做——工作自己开始写进你的知识库。
- **整理知识库（organize）给你一个定期、低负担的决策时刻。** 它把一堆草稿变成一份**可视化看板报告**——什么值得留、每一块该去哪、什么该扔。你扫一眼（不是读原始草稿）、点头或调整、它就提交。不再有压力，只是一次干净的周期性回顾，重活已经替你干完了。

捕获不需要你参与。判断永远留给你。

---

## 快速开始

```bash
# 1. 克隆到你的 agent 技能库
git clone https://github.com/beiyuii/knowledge-capture ~/.agents/skills/knowledge-capture

# 2. 配置你的 vault 路径
cd ~/.agents/skills/knowledge-capture
cp config/paths.example.json config/paths.json
# 编辑 paths.json，填入你的 Obsidian vault 路径

# 3. Claude Code 用户，额外链接到它的技能目录
ln -s ~/.agents/skills/knowledge-capture ~/.claude/skills/knowledge-capture
```

然后在任意 agent 会话里：

```
# 说「记录会话」  → 自动捕获进收件箱
# 说「整理知识库」→ 生成看板报告，你确认后才提交
```

---

## 你会得到什么

| 路径 | 作用 |
|---|---|
| `SKILL.md` | 入口——定义 save 和 organize 两个流程 |
| `adapters/` | 读端——每个工具的解析器，把原始日志变成统一的 Session IR |
| `adapters/zcode.md` | zcode rollout 解析器（Phase 1） |
| `adapters/claude-code.md` | Claude Code 会话解析器（Phase 1） |
| `vaults/` | 写端——每个知识库的整理规则 |
| `vaults/obsidian-knowledge-palace.md` | Obsidian KP folder-boundaries 决策树（Phase 1） |
| `rules/capture-rules.md` | 捕获什么 / 不捕获什么 / 合并粒度 |
| `rules/organize-rules.md` | 合并与丢弃判断 + 三条铁律 + 写入白名单 |
| `templates/draft.md` | 收件箱草稿 frontmatter 模板 |
| `templates/review-report.html` | 看板报告模板（浅色精致、看板墙驱动） |
| `templates/now-pointer.md` | now.md 里程碑指针模板 |
| `state/watermark.json` | 增量去重水线（不入 git） |
| `config/paths.json` | 你的 vault 与日志路径（不入 git） |

---

## 架构

knowledge-capture 采用**双适配器**模型——读端和写端对称，中间用统一的 Session IR 衔接：

```
会话日志 (zcode / Claude Code / Codex / …)
        │
        ▼  读端适配器（每工具一个 .md）
   ┌─────────────────────┐
   │     Session IR       │  ← 唯一的公共契约
   │  {events:[{role,     │    （下游只认 IR，
   │   kind, content}]}   │     不碰原始格式）
   └─────────────────────┘
        │
        ▼  capture-rules（消化 → 草稿）
   收件箱草稿  ──────►  now.md 里程碑指针  （读回路闭环 ✓）
        │
        ▼  organize（按需）
   看板 HTML 报告  ──►  你说"OK"或调整  ──►  写端适配器晋升为正式笔记
```

### 两个模式，严格分离

| 触发 | 模式 | 自动化 | 你做什么 |
|---|---|---|---|
| 「记录会话」/ `/kc save` | **save** | 全自动 | 无 |
| 「整理知识库」/ `/kc organize` | **organize** | 半自动 | 扫一眼报告，说 OK 或调整 |

**为什么分开**：如果捕获和评审是一步，每次会话结束你都会被评审骚扰。分开后，捕获保持静默零负担；评审变成周期性、攒一批、由你决定时机。

### Session IR —— 多工具支持的关键

每个 agent 工具的日志用不同的存储格式（JSONL、SQLite、Markdown、ZIP 导出）和不同的字段名。**没有万能解析器**。但「值得捕获的内容」的语义是共通的——用户说了什么、AI 想了什么、用了什么工具。

所有读端适配器把日志解析成同一种 **Session IR**。下游规则（capture / organize）只写一遍，对所有工具生效。新增一个工具 = 加一个 `.md` 适配器，零编译。

工具分三族，可共享解析骨架：

| 族 | 工具 | 共享骨架 |
|---|---|---|
| 类型化事件 JSONL | zcode、Claude Code、Codex | `JsonlEventFamily` |
| 单文件 JSON 会话 | Cline/Roo、Zed、Cody | （Phase 2） |
| 树形/列表 JSON 导出 | ChatGPT 导出、Claude.ai 导出 | （Phase 3） |

---

## 看板报告

运行 organize 时，技能生成一份**静态、精致的 HTML 报告**，并在浏览器打开：

- **顶部流向条** —— 一条彩色横条，一眼看出草稿怎么分（入库 / 合并 / 归档）。
- **按去向房间分组的卡片墙** —— 每张卡只显示类型标签 + 标题 + 来源。**没有正文。** 点击看一句话摘要和建议去向的理由。
- **合并区** —— `草稿 A + 草稿 B → 目标` 的 chip 流。
- **归档区** —— 单行条目带理由，让你能抓住误判的丢弃。

你用眼睛评审，然后在终端说"OK"提交，或描述要改什么。技能重新生成完整报告，直到你满意。**你确认前，什么都不写。**

---

## 安全

| 担忧 | 保证 |
|---|---|
| 重复处理同一段日志 | 水线增量去重——绝不重复捕获 |
| 去向错误 / AI 误判 | 报告评审兜底；全是建议，确认前不提交 |
| 误删 | 归档 ≠ 删除；丢弃的草稿移到 `90.archive/discarded/`，可逆 |
| 损坏你的正式笔记 | 原子暂存提交——所有目标文件先进 `.staging/`，全部成功才移入正式房间 |
| 断裂 wikilink | 提交前扫描；若目标被引用，保留重定向，绝不静默断链 |
| 碰你的身份层 | 写入白名单硬护栏——技能只能写 `30.knowledge/` 内，绝不碰 `ME.md` / 身份 / 技能 |

---

## Agent 兼容性

技能是纯 markdown，任何能读文件、能按指令执行的 AI agent 都能用。

- **zcode** —— 读 `~/.zcode/cli/rollout/*.jsonl`（Phase 1 ✅）
- **Claude Code** —— 读 `~/.claude/projects/<dir>/*.jsonl`（Phase 1 ✅）
- **Codex CLI** —— 读 `~/.codex/sessions/`（Phase 2，同 JSONL 族）
- **Cline / Roo Code、Zed、Cody** —— 单文件 JSON 会话（Phase 2）

**触发**：CLI 类 agent（zcode、Claude Code、Codex、Aider）可被 cron 定时调度。GUI 类（Cursor、Windsurf）无干净命令行入口——用手动触发。技能本身是被动执行体，不关心怎么被调用。

> **不支持**（无稳定的本地日志）：Cursor（SQLite+zlib，schema 未文档化）、Windsurf 与 Gemini（纯云端）、ChatGPT 桌面版（加密）。

---

## 生态

knowledge-capture 是独立仓库，也接入 [**skillpack**](https://github.com/beiyuii/skillpack) 矩阵。三个独立组件，组合成完整的个人 AI 工作流：

| 仓库 | 角色 | 方向 |
|---|---|---|
| **knowledge-capture**（本仓库） | 会话日志 → 知识库 | 捕获与晋升 |
| [personal-api-skill](https://github.com/beiyuii/personal-api-skill) | Obsidian vault → AI 身份层 | 身份与导航 |
| [skillpack](https://github.com/beiyuii/skillpack) | 本地技能农场 + 矩阵编排器 | 安装与管理 |

**它们怎么连接**：`personal-api-skill` 搭建你的 vault（ME.md、Knowledge Palace v2 结构），让任何 AI agent 理解你是谁。`knowledge-capture` 然后往这个 vault 里喂捕获来的洞察——它把草稿晋升进 `personal-api-skill` 搭好的 `30.knowledge/` 房间，**遵循同一套 Knowledge Palace v2 规则**。`skillpack` 安装并编排这两个技能。

```
personal-api-skill ──搭建──► vault (ME.md + 30.knowledge/)
                                    ▲
knowledge-capture ───晋升进─────────┘   （遵循 KP v2 规则）
        ▲
skillpack ──安装──► 两个技能
```

---

## 阶段路线图

- ✅ **Phase 1（当前，已在真实 vault 端到端验证）** —— zcode + Claude Code 适配器、save 与 organize 双模式、看板报告、Obsidian-KP 写端适配器。
- ⏳ **Phase 2** —— Codex / Cline-Roo 适配器 · `skillpack skill install` 命令 · 通过 hermes 的 cron 定时。
- 🔜 **Phase 3** —— ChatGPT/Claude.ai 导出适配器 · 每周提醒报告 · Logseq/Notion vault 适配器。

---

## 设计文档

- [Spec](https://github.com/beiyuii/skillpack/blob/main/docs/superpowers/specs/2026-06-24-knowledge-capture-design.md) —— 完整设计（5 段 brainstorming）
- [Phase 1 Plan](https://github.com/beiyuii/skillpack/blob/main/docs/superpowers/plans/2026-06-24-knowledge-capture-phase1.md) —— 实现计划（经 Plan Review）
- [验证记录](./docs/phase1-validation.md) —— 端到端真机验证

License: MIT.
