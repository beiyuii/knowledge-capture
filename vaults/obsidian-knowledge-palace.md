# 写端适配器：Obsidian + Knowledge Palace v2

把草稿整理进 Obsidian 知识库（采用 Knowledge Palace v2 组织方法）。所有整理决策、frontmatter、约束都严格对齐该 vault 的 `30.knowledge/00.system/` 既有规则——本文件不臆造规则，只把 vault 已有的规则转成 AI 可执行的指令。

## 适用 vault

顶层目录含 Johnny.Decimal 编号房间的 Obsidian vault：

```
00.context/  10.identity/  20.skills/  30.knowledge/  40.memory-stream/  50.maps/
30.knowledge/
  00.system/  10.capture/  20.intelligence/  30.research/  40.notes/
  50.frameworks/  60.projects/  70.outputs/  90.archive/
```

## 一、整理决策树（对齐 folder-boundaries.md 的快速决策树）

整理每张草稿时，**按此决策树逐条判断**（不是按草稿类型查表——同一类型可能去不同房间）：

```
第 1 步：是否绑定具体项目？
  （草稿提到/属于某个进行中的项目，如 rencai / drift / skillpack / personal-api…）
  ├─ 是 → 60.projects/<项目名>/  （项目室：项目知识资产、复盘、决策记录）
  └─ 否 → 第 2 步

第 2 步：按内容性质分发（用 folder-boundaries 决策树）
  ├─ 短期行业动态 / AI 信号 / 产品动态 / 趋势 → 20.intelligence/{ai,business}/  （情报室）
  ├─ 长期专题 / 问题链 / 研究索引             → 30.research/                     （研究室）
  ├─ 个人原子化认知、可长期复用                → 40.notes/permanent/              （卡片柜·永久卡）
  ├─ 对外部材料的结构化笔记                    → 40.notes/literature/             （卡片柜·文献）
  ├─ 可复用方法 / SOP / 判断模型 / 模板        → 50.frameworks/                   （工具墙）
  └─ 准备对外发布或已发布的内容                → 70.outputs/                      （发布厅）

第 3 步：无价值
  └─ 寒暄 / 未导致结论的探索 / 操作指令 / 个人信息
     → 90.archive/discarded/YYYY-MM/  （归档，加 #status/raw，不真删）
```

**判断依据**：folder-boundaries.md 的「目录边界」表（每房间有「放这里 / 不放这里」）。拿不准时翻那张表。

## 二、frontmatter 规范（对齐 folder-boundaries.md 的标签建议）

落盘到任何房间，文件 frontmatter 必须含：

```yaml
---
aliases: [<稳定短名>]          # 方便 wikilink
updated: YYYY-MM-DD
layer: 2                       # 30.knowledge 全是 layer 2
status: <status 标签>          # 见下表
type: <type 标签>              # 见下表
source_captured: YYYY-MM-DD    # 来自哪次捕获（可追溯）
---
```

**status 标签**（folder-boundaries.md 定义）：

| 标签 | 用途 |
|---|---|
| `#status/raw` | 未处理原始材料 |
| `#status/compiled` | 已整理文献笔记 |
| `#status/permanent` | 已沉淀永久认知 |
| `#status/published` | 已对外输出 |

**type 标签**：`#type/intelligence` / `#type/research` / `#type/framework` / `#type/output`（按房间对应）。

**草稿原 frontmatter 的保留**：草稿里的 `source`（tool/session/file/lines）必须**搬到成品 frontmatter 的 `capture_source` 字段**，保证可追溯（KP v2「不能为整齐牺牲可追溯」）。

## 三、硬约束（对齐 methodology.md「绝对不要做的事」）

### 写入白名单（AI 只能写这些路径前缀）

```
✅ 允许写入（30.knowledge/ 内）：
   30.knowledge/10.capture/inbox/
   30.knowledge/10.capture/promoted/
   30.knowledge/20.intelligence/
   30.knowledge/30.research/
   30.knowledge/40.notes/
   30.knowledge/50.frameworks/
   30.knowledge/60.projects/
   30.knowledge/70.outputs/
   30.knowledge/90.archive/
   30.knowledge/.staging/        （落盘暂存区）

❌ 绝不允许写入（轨道 A，100% 人工策展）：
   ME.md
   10.identity/
   20.skills/
   40.memory-stream/
   50.maps/
   00.context/                   （含 now.md！now.md 指针是"追加一行"，且需用户确认语境）
```

> **now.md 例外**：记录会话（save）模式追加 now.md 指针是允许的，因为它是「追加里程碑行」而非改写身份内容。但整理（organize）模式不碰 now.md 的身份部分。

### 落盘原子性

所有目标文件**先写进 `30.knowledge/.staging/`**，全部成功后才原子移入正式房间。中途失败 = staging 残留、正式区零污染。

### wikilink 保护

移动文件前**先扫全库 wikilinks**（grep `[[目标文件名]]`）：
- 若目标被引用 → 报告里标红警告，落盘时保留旧文件软链/重定向，**绝不静默断链**。
- 这条是 methodology.md「批量移动前必须先检查 wikilinks」的硬要求。
