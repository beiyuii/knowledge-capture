---
name: knowledge-capture
description: Capture AI agent session logs (zcode/Claude Code/Codex…) into your knowledge base. Two modes — 记录会话 (save, fully automatic) and 整理知识库 (organize, kanban HTML report + conversational approval). Multi-tool read adapters + vault-aware organize.
---

# knowledge-capture

把 AI agent 工具的会话日志沉淀进知识库。

## 你是被动执行体

> **重要**：本技能不负责自己怎么被触发。触发方式是用户/工具侧的事（cron / 主动说 / `/kc save` `/kc organize`）。你只在被要求时按下面的模式执行。系统提示词里**没有**自动触发指令——这是刻意设计，避免长上下文里的指令衰减。

## 两个模式

用户说以下任一即进入对应模式：

| 用户说 | 模式 | 做什么 |
|---|---|---|
| 「记录会话」/「记录一下今天的会话」/ `/kc save` | **save** | 读增量日志 → 写 inbox 草稿 → 追加 now.md 指针 |
| 「整理知识库」/「总结这周到知识库」/ `/kc organize` | **organize** | 读 inbox → 看板报告 → 等对话 → OK 才落盘 |

---

## 模式 save（记录会话，全自动，零负担）

按以下顺序执行：

### 1. 读配置
读 `config/paths.json`（若不存在，提示用户 `cp config/paths.example.json config/paths.json` 并填路径）。取 `vault`、`tools.*.logDir`、`inboxDir`、`nowMd`、`timezone`。

### 2. 增量读取日志（水线去重）
- 读 `state/watermark.json`（不存在则视为空，全量读）。
- 对 `paths.json` 里每个工具，用 `adapters/<tool>.md` 的解析逻辑，读该工具 `logDir` 下每个日志文件的**水线以下新增行**，解析成 Session IR（见 `adapters/_ir-schema.md`）。
- **水线校验**：若记录的行数 > 文件实际行数（文件被截断/轮转），重置该文件水线为 0，全量重读（宁重复不漏读）。
- **错误处理**：某工具日志读失败 → 跳过该工具，记 `state/errors.log`，继续其他工具。绝不因一个工具挂了全停。

### 3. 消化成草稿
对每个 Session IR，按 `rules/capture-rules.md` 消化：
- 只捕获 decision/fix/architecture/method 四类（判断标准见规则）。
- 不捕获寒暄/纯调试/无结论探索/隐私。
- **主题级合并**：同会话同主题多轮 → 一张草稿。
- 每张草稿用 `templates/draft.md` 格式，frontmatter 必须含 `source`（tool/session/file/lines 可追溯）。

### 4. 写 inbox + now.md 指针
- 草稿写入 `<vault>/<inboxDir>/capture-<date>-<slug>.md`。
- **幂等**：now.md 已有同 `source.session` 的里程碑指针 → 跳过追加。否则用 `templates/now-pointer.md` 格式，在 now.md「近期里程碑」表最新行**上方**插入一行。
- 草稿写入失败 → 回滚本次草稿 + 不追加 now.md（原子一致）。

### 5. 更新水线 + 汇报
- 把每个文件的已读行数写回 `state/watermark.json`。
- 汇报：「记录了 N 个会话，产出 M 张草稿，now.md 追加了 K 个指针」。

---

## 模式 organize（整理知识库，半自动，要评审）

按以下顺序执行：

### 1. 读 inbox
读 `<vault>/<inboxDir>/` 全部草稿（`status: inbox`）。

### 2. 整理判断
对每张草稿，按 `rules/organize-rules.md` 判断：
- **合并判断**：多张指同主题 → 建议合并。
- **丢弃判断**：寒暄/无结论/重复 → 建议归档（附理由）。
- **去向判断**：按 `vaults/<vault>.md` 的**整理决策树**（如 Obsidian-KP 的 folder-boundaries 决策树）定目标房间。

### 3. 生成看板报告
用 `templates/review-report.html` 填充判断结果，写入 `<vault>/30.knowledge/10.capture/review-<week>.html`，**`open` 打开浏览器**让用户看。
- 报告是**静态只读**，不含任何可执行落盘动作。
- 卡片按去向房间分组，不露正文。

### 4. 等用户对话（三条铁律，见 organize-rules.md）
- **铁律 1**：OK 前绝不动任何文件。即使用户沉默也不算同意。
- **铁律 2**：用户提异议后，修订判断并**重新生成完整报告**。
- **铁律 3**：你只改判断、不辩解。用户说了算。

循环直到用户明确说「OK / 执行 / 没问题」。

### 5. 落盘（用户 OK 后）
**写入白名单检查**：任何目标路径不在 `vaults/<vault>.md` 的白名单内 → 拒绝并报错。
1. **staging**：所有目标文件先写进 `<vault>/<stagingDir>/`（补齐该房间 frontmatter 规范，保留 `capture_source`）。
2. **wikilink 检查**：移动前 grep 全库 `[[目标文件名]]`，被引用则标红警告 + 保留重定向。
3. **commit**：staging 全部成功后，原子移入正式房间。
4. **清理**：入库/合并草稿 → `status: promoted`，移 `<promotedDir>/`；归档草稿 → 移 `<discardedDir>/<YYYY-MM>/`，加 `status: discarded` + 理由。
5. **now.md**：追加一行「本周知识整理 N 条」里程碑指针。
6. **inbox** 只剩未评审草稿。
7. 汇报：「入库 X，合并 Y，归档 Z，now.md 已更新」。

### 落盘失败处理
staging 任一步失败 → staging 残留、正式区零污染。报错，等用户指示。

---

## 文件索引

| 文件 | 作用 |
|---|---|
| `adapters/_ir-schema.md` | Session IR 规范 |
| `adapters/<tool>.md` | 各工具解析规则（zcode/claude-code/…） |
| `vaults/<vault>.md` | 各知识库整理规则（obsidian-knowledge-palace/…） |
| `rules/capture-rules.md` | 捕获什么/不捕获什么 |
| `rules/organize-rules.md` | 整理判断 + 对话铁律 + 写入白名单 |
| `templates/draft.md` | 草稿模板 |
| `templates/now-pointer.md` | now.md 指针模板 |
| `templates/review-report.html` | 看板报告模板 |
| `config/paths.json` | 路径配置（用户填） |
| `state/watermark.json` | 增量去重水线（运行时） |

## 错误处理总览（按破坏性分级）

- **无破坏**（日志读失败/格式异常）：跳过/降级，记 errors.log，继续。
- **低破坏**（水线错乱/草稿写失败）：重置/回滚，报错。
- **中破坏**（落盘失败/断链风险）：staging 原子 + wikilink 保护。
- **高破坏**（AI 误判/撞轨道A）：报告评审兜底 + 写白名单硬护栏拒绝。

**核心原则**：所有破坏性动作可逆（归档不删、落盘走 staging、断链留重定向），所有不可逆动作都需用户 OK。
