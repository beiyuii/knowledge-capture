# 适配器：Claude Code

把 Claude Code 的会话日志解析成 [Session IR](_ir-schema.md)。

## 日志位置

```
~/.claude/projects/<encoded-dir>/*.jsonl
```

`<encoded-dir>` 是工作目录路径的编码（`/` → `-`），例如 `-Users-beiyuii-Desktop-skillpack`。每个 `.jsonl` 文件 = 一个会话。

## 格式

**族：A（类型化事件 JSONL）**，复用 [JsonlEventFamily](README.md#jsoneventfamily-解析逻辑a-族适配器共用) 骨架。

每行是一个事件对象，顶层 `type` 字段区分事件类型：

| `type` | 含义 | 关键字段 |
|---|---|---|
| `user` | 用户消息或工具结果 | `message.role` / `message.content[]`（content 可能是 `text` 或 `tool_result`） |
| `assistant` | 助手消息 | `message.content[]`，每项可能是 `text` / `thinking` / `tool_use` |
| `queue-operation` | 排队操作 | `operation` / `content`（用户输入文本） |
| `attachment` | 附件（hook 等） | — |
| `mode` | 模式标记 | — |

每事件含 `timestamp`、`sessionId`、`uuid`、`parentUuid`（用于重建对话树，本适配器走线性时序，不用 parentUuid）。

## type → IR 映射表

遍历每行，按 `type` 和 `message.content[]` 的子类型映射：

| 源 | IR Event |
|---|---|
| `type:user` + `content[].type:text` | `{role: user, kind: text, content, ts: timestamp}` |
| `type:user` + `content[].type:tool_result` | `{role: tool, kind: tool_result, content, ts: timestamp}` |
| `type:assistant` + `content[].type:text` | `{role: assistant, kind: text, content, ts}` |
| `type:assistant` + `content[].type:thinking` | `{role: assistant, kind: thinking, content: thinking字段, ts}` |
| `type:assistant` + `content[].type:tool_use` | `{role: tool, kind: tool_use, toolName: name, content: JSON.stringify(input), ts}` |
| `type:queue-operation` + `operation:enqueue` | `{role: user, kind: text, content: content字段, ts}`（用户输入） |
| 其他（attachment/mode/last-prompt） | 跳过 |

> Claude Code **完整保留 thinking**（`content[].type:thinking`），这是它相比 zcode rollout 的优势——thinking 是高价值的"AI 怎么想的"信号。

## 解析逻辑（JsonlEventFamily）

1. 逐行读 `~/.claude/projects/<encoded-dir>/<sid>.jsonl`，取水线以下新增行。
2. 每行按上表映射出 0~N 个 Event（一行 user/assistant 可能含多个 content 子项）。
3. `ts` 取 `timestamp`。
4. 拼成 Session IR，`tool: "claude-code"`，`sourceFile` = 该 jsonl 路径，`lines` = [水线旧值, 当前行数]。

## 如何定时触发 Claude Code

Claude Code 是 CLI agent，可被 cron 调用：

```
# 每天记录一次当天会话
claude -p "记录会话"
```

> 非交互模式 `-p` 以当前版本为准。模板在 Phase 2 接入 hermes 时细化。
