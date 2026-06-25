# 读端适配器（adapters/）

读端适配器负责把某个 AI agent 工具的本地会话日志，解析成统一的 [Session IR](_ir-schema.md)。下游（记录会话 / 整理知识库）只认 IR，不认原始格式。

## 职责边界

- ✅ 适配器**只做解析**：读日志 → 吐 IR。
- ❌ 适配器**不做**：去重（水线机制做）、消化（capture-rules 做）、整理（vaults 适配器做）。
- ✅ 适配器要**容错**：格式异常不崩，记 `[unparseable]`。

## 三族分类

经调研（13 个主流工具），日志格式分三族 + 一批无法稳定访问的工具：

| 族 | 工具 | 共性 | Phase |
|---|---|---|---|
| **A. 类型化事件 JSONL** | zcode、Claude Code、Codex CLI | 每行一事件，`type` 区分角色，可流式追加 | Phase 1（zcode + claude）/ Phase 2（codex） |
| **B. 单文件 JSON 会话** | Cline/Roo Code、Zed、Cody 导出 | 一文件一会话，消息数组 + 角色 | Phase 2 |
| **C. 树形/列表 JSON 导出** | ChatGPT 导出、Claude.ai 导出 | `conversations.json`，树形分支 | Phase 3 |
| ❌ 无法访问 | Cursor（SQLite+zlib 未文档化）、Windsurf（云端）、Gemini（云端）、ChatGPT 桌面版（加密） | — | 不承诺 |

**核心判断**：存储介质跨 SQLite/JSONL/Markdown/ZIP，必须多适配器；但同族可共享解析骨架。

## 共享骨架

- **JsonlEventFamily**（A 族）：zcode / Claude Code / Codex 共享「逐行读 JSONL → 按 type 路由 → 映射成 Event」的骨架，各适配器只差一张 `type → IR.kind` 的映射表。本骨架的解析逻辑写在本文件，各适配器只声明自己的映射表。

### JsonlEventFamily 解析逻辑（A 族适配器共用）

1. 逐行读 `.jsonl`（每行一个 JSON 对象）。
2. 取水线以下的新增行（增量）。
3. 对每行，按该工具的映射表（见各适配器）解析出 0~N 个 Event。
4. 按时间序拼成 Session IR。

## 如何新增一个适配器

1. 确认它属于哪一族（参考三族表）。
2. 若是 A 族：在 `adapters/<tool>.md` 里写「日志路径 + 格式 + type→IR 映射表」，复用 JsonlEventFamily 骨架。
3. 若是新族：在 `adapters/<tool>.md` 里写完整解析逻辑（无骨架可复用）。
4. 附一份脱敏样本到 `adapters/fixtures/<tool>-sample.jsonl`。
5. 在 `tests/test-adapter-parse.md` 加该工具的解析断言。

## 通用性边界（重要）

本技能的通用性体现在「能读谁的日志」。但**「能不能被 cron 定时触发」是另一回事**，与适配器无关：
- CLI 类工具（zcode/Claude Code/Codex/Aider）有命令行入口，可被 cron 调。
- GUI 类工具（Cursor/Windsurf/VSCode 扩展）无干净命令行入口，只能手动触发。

各适配器末尾附「如何定时触发本工具」说明（CLI 给 cron 模板，GUI 标注需手动）。这是部署侧的事，不影响适配器本身的解析职责。
