# 测试：适配器解析

验证 `adapters/<tool>.md` 能把脱敏样本正确解析成 Session IR。

## 输入

- `adapters/fixtures/zcode-sample.jsonl`（2 行）
- `adapters/fixtures/claude-sample.jsonl`（5 行）

## 执行

对每个样本，按对应适配器的解析逻辑解析，输出 Session IR。

## 期望断言（精确，逐条核对）

### zcode-sample.jsonl

- [ ] 产出 1 个 Session IR（一个文件 = 一个会话）
- [ ] `IR.tool == "zcode"`
- [ ] `IR.sessionId == "sess_demo001"`
- [ ] `IR.sourceFile` 含 `model-io-sess_demo001.jsonl`
- [ ] `IR.lines == [0, 2]`（从 0 读到第 2 行）
- [ ] `IR.events` 数量 == 3（L1: 1 assistant text + 1 tool_use；L2: 1 assistant text）
  - 断言细节：
    - [ ] 存在 `{role: assistant, kind: text, content: "好的，我来帮你整理知识库..."}`（L1 response.text）
    - [ ] 存在 `{role: tool, kind: tool_use, toolName: "Bash", content 含 "ls <vault>/30.knowledge/10.capture/inbox/"}`（L1 toolCalls）
    - [ ] 存在 `{role: assistant, kind: text, content 含 "inbox 里有 3 张草稿"}`（L2 response.text）
- [ ] 无 `[unparseable]` 事件（样本格式正常）

### claude-sample.jsonl

- [ ] 产出 1 个 Session IR
- [ ] `IR.tool == "claude-code"`
- [ ] `IR.events` 数量 == 5（user text / thinking / tool_use / tool_result / assistant text 各 1）
  - 断言细节：
    - [ ] 存在 `{role: user, kind: text, content: "帮我整理一下知识库"}`（type:user + content text）
    - [ ] 存在 `{role: assistant, kind: thinking, content 含 "用户要整理 inbox"}`（type:assistant + content thinking）
    - [ ] 存在 `{role: tool, kind: tool_use, toolName: "Bash"}`（type:assistant + content tool_use）
    - [ ] 存在 `{role: tool, kind: tool_result, content 含 "draft-a.md"}`（type:user + content tool_result）
    - [ ] 存在 `{role: assistant, kind: text, content 含 "生成看板报告"}`（type:assistant + content text）
- [ ] `queue-operation` 行被跳过（样本里没有，若有应跳过）

## 失败处理

任一断言为假 → 报告该断言失败 + 实际值，不继续后续测试。
