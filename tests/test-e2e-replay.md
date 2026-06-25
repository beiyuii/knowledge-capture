# 测试：端到端回放

验证 save → organize 全链路在黄金会话样本上闭环。这是 Phase 1 的回归基线——以后改任何东西，跑这条确保没破坏闭环。

> **最终人工把关**：本测试的 organize 阶段（生成报告、用户对话、落盘）以 Plan Task 12 的**真实数据验证**为准。本文件用样本做结构验证，真实验证见 Task 12。

## 输入（黄金会话样本）

`adapters/fixtures/zcode-sample.jsonl` + `adapters/fixtures/claude-sample.jsonl`（合并视为一次"本周"的捕获）。

## 执行

1. 清空 state（删 `watermark.json`）。
2. **save**：对两个样本跑记录会话。
3. **organize**：对 inbox 跑整理，生成报告。

## 期望断言

### save 阶段

- [ ] 两个样本都产出 Session IR（无 unparseable）
- [ ] inbox 出现草稿（数量 >= 1；样本含明确的"整理知识库"决策，应被捕获为 decision 类）
- [ ] 草稿 frontmatter 含 `source`（tool/session/file/lines 四字段齐全）
- [ ] 草稿 `capture_type` 是 decision/fix/architecture/method 之一
- [ ] now.md 追加了里程碑指针（含 wikilink 到草稿）

### organize 阶段

- [ ] 生成了 `review-*.html` 报告文件
- [ ] 报告含流向条（入库/合并/归档三段，flex 值 = 各自数量）
- [ ] 报告含按房间分组的卡片（不露正文）
- [ ] 去向判断符合 Obsidian-KP 决策树（如 decision 类草稿 → 40.notes/permanent 或相关房间，不应乱放）
- [ ] 报告格式与 `templates/review-report.html` 一致（含 style/lane/card/merge/discard 结构）

### 对话循环（铁律）验证

- [ ] 生成报告后，**没有**自动落盘（inbox 文件 status 仍为 inbox）
- [ ] 模拟用户提异议 → 能修订并重新生成完整报告
- [ ] 模拟用户说「OK」→ 才执行落盘

### 落盘后（模拟 OK）

- [ ] 草稿进入正确房间（路径符合决策树）
- [ ] 入库草稿 status 变为 promoted，移到 promotedDir
- [ ] inbox 只剩未评审草稿
- [ ] now.md 追加了"本周整理 N 条"指针
- [ ] **未碰轨道 A**（ME.md/identity/skills/memory-stream/maps/context 未被修改）

## 失败处理
任一断言为假 → 报告失败 + 实际值 + 哪个阶段。这是回归基线，失败必须修复才能继续。
