# Session IR（中间表示）规范

所有读端适配器必须把日志解析成下面的统一结构。下游（记录会话 / 整理知识库）只认 IR，不碰原始格式。这是「通用捕获层」成立的唯一关键约束——共同点只在 IR 层。

## 结构

Session 是一个对象：

| 字段 | 类型 | 说明 |
|---|---|---|
| `sessionId` | string | 会话唯一标识 |
| `tool` | string | 来源工具名（`"zcode"` / `"claude-code"` / …） |
| `sourceFile` | string | 原始日志文件路径（可追溯锚，对应草稿 `source.file`） |
| `lines` | `[int, int]` | 本次解析覆盖的行号区间 `[start, end]`（对应草稿 `source.lines`） |
| `startedAt` | ISO8601 | 会话开始时间 |
| `endedAt` | ISO8601 | 会话结束时间 |
| `events` | Event[] | 事件序列（时间序） |

Event 是一个对象：

| 字段 | 类型 | 说明 |
|---|---|---|
| `role` | enum | `user` / `assistant` / `system` / `tool` |
| `kind` | enum | `text` / `thinking` / `tool_use` / `tool_result` |
| `content` | string | 文本内容（tool_use / tool_result 时为序列化的输入/输出摘要） |
| `toolName` | string? | `kind=tool_use` / `tool_result` 时的工具名，否则省略 |
| `ts` | ISO8601 | 事件时间 |

## 映射约定

- **thinking**：一律映射为 `{role: assistant, kind: thinking}`，无论原格式叫什么（reasoning / 思维链 / thinking block / …）。
- **工具调用**：映射为 `{role: tool, kind: tool_use}`；工具结果映射为 `{role: tool, kind: tool_result}`，`toolName` 填工具名（Read/Bash/Edit/…）。
- **无法解析的字段**：记 `{kind: text, content: "[unparseable]"}`，**不丢弃事件、不崩**。记录会话时这类事件会被 capture-rules 自然过滤（属于"未导致结论的探索"）。
- **超长内容**：`content` 超过 ~2000 字符时，截断并追加 `…[truncated, N chars total]`。下游消化只关心语义，不需要全文。

## 示例

```json
{
  "sessionId": "sess_abc123",
  "tool": "zcode",
  "sourceFile": "~/.zcode/cli/rollout/model-io-sess_abc123.jsonl",
  "lines": [412, 1247],
  "startedAt": "2026-06-24T10:00:00+08:00",
  "endedAt": "2026-06-24T11:30:00+08:00",
  "events": [
    {"role": "user", "kind": "text", "content": "整理一下知识库", "ts": "2026-06-24T10:00:00+08:00"},
    {"role": "assistant", "kind": "thinking", "content": "用户要整理…需要先读 inbox", "ts": "2026-06-24T10:00:05+08:00"},
    {"role": "tool", "kind": "tool_use", "toolName": "Bash", "content": "ls 10.capture/inbox", "ts": "2026-06-24T10:00:10+08:00"}
  ]
}
```

## 为什么有 IR

不同工具的日志存储介质（JSONL / SQLite / Markdown / ZIP）和字段词汇天差地别，**没有统一解析器的可能**。但「值得捕获的内容」的语义是共通的——都是「用户说了什么、AI 想了什么、AI 用工具做了什么」。IR 把这个共通语义固定下来，让下游消化的规则（capture-rules / organize-rules）只写一遍，对所有工具生效。
