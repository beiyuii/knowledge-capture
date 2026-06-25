# 适配器：zcode

把 zcode 的会话日志解析成 [Session IR](_ir-schema.md)。

## 日志位置

```
~/.zcode/cli/rollout/*.jsonl
```

每个文件名形如 `model-io-sess_<sessionId>.jsonl`，一个文件 = 一个会话的 model IO 记录。

## 格式

**族：A（类型化事件 JSONL）**，复用 [JsonlEventFamily](README.md#jsoneventfamily-解析逻辑a-族适配器共用) 骨架。

每行是一个 JSON 对象，`type: "model_io"`，含一次完整的 model 请求/响应：

| 字段 | 含义 |
|---|---|
| `sessionId` | 会话 ID → IR `sessionId` |
| `turnId` | 轮次 ID |
| `request.body.system[]` | system prompt（数组，每项 `{type:"text", text}`） |
| `response.text` | assistant 的文本输出 |
| `response.toolCalls[]` | 工具调用数组，每项 `{id, name, input}` |
| `response.usage` | token 用量 |
| `completedAt` | 完成时间 → 事件 `ts` |
| `model.modelId` | 模型名 |

> **注意**：zcode rollout **只记录 model 单次 IO**，不含完整对话历史。用户消息（user role）的历史走 zcode 的会话数据库（`~/.zcode/cli/db/db.sqlite`），不在 rollout 里。本适配器**只解析 rollout**，从 `response.text` / `response.toolCalls` 提取 assistant 的产出；用户意图若需补全，可后续扩展为同时读 db（Phase 2 评估）。

## type → IR 映射表

| rollout 字段 | IR Event |
|---|---|
| `response.text`（非空） | `{role: assistant, kind: text, content: <text>, ts: completedAt}` |
| `response.toolCalls[].name/input` | 每个 → `{role: tool, kind: tool_use, toolName: <name>, content: JSON.stringify(input), ts: completedAt}` |
| `request.body.system[].text` | 每个 → `{role: system, kind: text, content: <text>}`（通常过滤掉，system prompt 无捕获价值） |

> zcode rollout 当前**不含独立的 thinking 字段**（thinking 在 model 内部，rollout 不单独落盘）。如未来 zcode 暴露 thinking，在此表加一行映射为 `{role: assistant, kind: thinking}`。

## 解析逻辑（JsonlEventFamily）

1. 逐行读 `~/.zcode/cli/rollout/model-io-sess_<sid>.jsonl`，取水线以下新增行。
2. 每行解析：取 `response.text`（若非空）→ 1 个 assistant text Event；遍历 `response.toolCalls[]` → 每个 1 个 tool_use Event。
3. `ts` 统一取 `completedAt`。
4. 拼成 Session IR，`tool: "zcode"`，`sourceFile` = 该 jsonl 路径，`lines` = [水线旧行数, 当前行数]。

## 如何定时触发 zcode

zcode 是 CLI agent，可被 cron 调用。模板（LaunchAgent / hermes job）：

```
# 每 2 小时记录一次会话
zcode -p "记录会话"
```

> 具体命令行参数以 zcode 当前版本为准（`zcode --help` 查非交互模式）。模板在 Phase 2 接入 hermes 时细化。
