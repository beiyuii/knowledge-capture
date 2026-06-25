# 测试（tests/）

> **纯 skill 测试张力声明**（必读）
>
> 本技能无编译、无测试框架。测试以「**黄金样本 + 精确断言清单**」形式存在，而非 CI 级自动化。
>
> - 断言写成**机器可查的精确条件**（如「产出草稿数 == 0」「watermark[文件] == 1247」），而非主观判断。
> - 执行时 agent 逐条核对待命令断言，全部为真才算通过。
> - 端到端测试（test-e2e-replay）以真实数据验证（Plan Task 12）为最终人工把关。
>
> 这是纯 skill 范式与 TDD 之间的根本张力。Phase 1 接受这个折中：**黄金样本 + 精确断言 + 人工把关**。若后续需要更强保证，可给关键解析逻辑配轻量校验脚本（Phase 2+ 评估）。

## 如何跑测试

1. 手动触发：在 agent 里说「跑 knowledge-capture 的测试」。
2. agent 按各 `test-*.md` 的「输入 → 执行 → 期望断言」执行。
3. 逐条核对待命令断言，报告哪些通过/失败。

## 三个测试

| 文件 | 测什么 | 输入 |
|---|---|---|
| `test-adapter-parse.md` | 适配器把日志解析成正确 IR | `adapters/fixtures/*-sample.jsonl` |
| `test-watermark-idempotent.md` | 水线增量去重幂等 | 同一份样本跑两次 |
| `test-e2e-replay.md` | save → organize 全链路 | 黄金会话样本 |
