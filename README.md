# knowledge-capture

> 把 AI agent 工具的会话日志沉淀进你的知识库。

一个**纯 skill**（markdown 指令 + AI 执行，零编译）。它把 zcode / Claude Code / Codex 等 agent 工具的会话日志自动整理进你的 Obsidian 知识库，并把 inbox 草稿通过**可视化看板报告 + 对话式拍板**半自动整理成正式知识。

- **读端通用**：按 agent 工具适配（每工具一个 markdown 适配器，统一吐 Session IR）。
- **写端通用**：按知识库适配（Obsidian Knowledge Palace / Logseq / Notion…），按知识库自己的规则整理。
- **独立的发行单元**：可脱离 skillpack 单独用；也是 skillpack 技能矩阵的一个组件。

## 快速开始

```bash
# 1. 装到 master 库（Phase 2 起：skillpack skill install knowledge-capture）
cp -r knowledge-capture ~/.agents/skills/knowledge-capture
cd ~/.agents/skills/knowledge-capture
cp config/paths.example.json config/paths.json
# 编辑 paths.json，填你的 vault 路径

# 2. 在 agent 里用
# 说「记录会话」→ 自动捕获日志进 inbox
# 说「整理知识库」→ 生成看板报告，你看了说 OK 才落盘
```

## 两个模式

| 触发 | 模式 | 自动化 | 你做什么 |
|---|---|---|---|
| 「记录会话」/ `/kc save` | save | 全自动 | 无 |
| 「整理知识库」/ `/kc organize` | organize | 半自动 | 看报告，说 OK 或提异议 |

详见 `SKILL.md`。

## 目录结构

```
SKILL.md              主入口
adapters/             读端：agent 工具适配器（zcode/claude-code…）
vaults/               写端：知识库适配器（obsidian-knowledge-palace…）
rules/                消化规则（捕获/整理）
templates/            产物模板（草稿/指针/看板报告）
config/               路径配置（用户填）
```

## 与 skillpack 的关系

knowledge-capture 是独立仓库，可单独用。它也是 [skillpack](https://github.com/beiyuii/skillpack) 技能矩阵的一个组件——skillpack 通过通用 `skill install` 命令把它（和 [personal-api-skill](https://github.com/beiyuii/personal-api-skill) 等）装入 master 库，形成矩阵。

- **skillpack**：本地技能软链农场管理器，技能发行/编排中心。
- **personal-api-skill**：把 Obsidian vault 变成 AI 身份层。
- **knowledge-capture**（本仓库）：把会话日志沉淀进知识库。

三者独立可用，组合成完整工作流。

## Phase 路线图

- **Phase 1（当前）**：zcode + Claude Code 适配器 + save/organize 两模式 + 看板报告 + Obsidian-KP 写端。
- **Phase 2**：Codex / Cline 适配器 + skillpack `skill install` + cron 定时接入。
- **Phase 3**：ChatGPT/Claude.ai 导出适配器 + 每周催办报告 + 其他 vault 适配器。

**不承诺支持**（已知无法稳定访问本地日志）：Cursor（SQLite+zlib 未文档化）、Windsurf（云端）、Gemini（云端）、ChatGPT 桌面版（加密）。

## 设计文档

- [Spec](../skillpack/docs/superpowers/specs/2026-06-24-knowledge-capture-design.md)
- [Phase 1 Plan](../skillpack/docs/superpowers/plans/2026-06-24-knowledge-capture-phase1.md)

## License

MIT
