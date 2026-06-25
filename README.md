# knowledge-capture

> **Turn your AI agent chats into permanent knowledge.** Auto-capture session logs from zcode / Claude Code / Codex into your Obsidian vault, then review and promote valuable drafts into permanent notes — through a visual kanban report.

中文说明：[README.zh-CN.md](./README.zh-CN.md)

[![version](https://img.shields.io/badge/version-0.1.0-blue)](./SKILL.md)
[![license](https://img.shields.io/badge/license-MIT-green)](./LICENSE)
[![category](https://img.shields.io/badge/category-capture%20%26%20review-purple)](#)
[![platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20WSL-lightgrey)](#)
[![agents](https://img.shields.io/badge/agents-zcode%20%7C%20Claude%20Code%20%7C%20Codex-orange)](#)

---

## Why

You work with AI agents every day — designing features, fixing bugs, making decisions. Each session is full of hard-won insight. But two things always go wrong:

**1. The work never makes it into your knowledge base on its own.**

You finish a session, the window closes, and that's it. The decision you debated, the bug you finally cracked, the method you figured out — none of it becomes a note unless you sit down and write it yourself. And nobody does that consistently. So your knowledge base stays empty while real insight leaks out through chat history every single day.

**2. There's no easy, recurring moment to decide what your knowledge base should actually hold.**

Even when you have a capture buffer, looking at it is unpleasant — a pile of raw scraps you have to read one by one, figure out what each is, decide where it goes. So you put it off. The buffer grows. The knowledge never lands. Your "second brain" becomes a graveyard you're afraid to open.

knowledge-capture fixes both:

- **save happens automatically, in the background.** It reads your agent session logs, pulls out the parts genuinely worth remembering, and drops them into your knowledge base inbox as drafts. You don't lift a finger — the work starts writing itself into your knowledge base.
- **organize gives you a periodic, low-effort moment to decide.** It turns the pile of drafts into a **visual kanban report** — what's worth keeping, where each piece should go, what to drop. You skim it (not the raw drafts), nod or nudge, and it commits. No more dread; just a clean weekly review where the heavy lifting is already done.

The capture happens without you. The judgment stays with you.

---

## Quick Start

```bash
# 1. Clone into your agent's skill library
git clone https://github.com/beiyuii/knowledge-capture ~/.agents/skills/knowledge-capture

# 2. Configure your vault path
cd ~/.agents/skills/knowledge-capture
cp config/paths.example.json config/paths.json
# edit paths.json — set your Obsidian vault path

# 3. For Claude Code, also link into its skill dir
ln -s ~/.agents/skills/knowledge-capture ~/.claude/skills/knowledge-capture
```

Then in any agent session:

```
# say 「记录会话」(record session)  → auto-captures into inbox
# say 「整理知识库」(organize)      → kanban report, approve to commit
```

---

## What You Get

| Path | Role |
|---|---|
| `SKILL.md` | Entry point — defines the save and organize flows |
| `adapters/` | READ side — per-tool parsers that turn raw logs into a unified Session IR |
| `adapters/zcode.md` | zcode rollout parser (Phase 1) |
| `adapters/claude-code.md` | Claude Code session parser (Phase 1) |
| `vaults/` | WRITE side — per-knowledge-base organize rules |
| `vaults/obsidian-knowledge-palace.md` | Obsidian KP folder-boundaries decision tree (Phase 1) |
| `rules/capture-rules.md` | What to capture / not capture / merge granularity |
| `rules/organize-rules.md` | Merge & discard judgment + 3 ironclad guardrails + write-allowlist |
| `templates/draft.md` | inbox draft frontmatter template |
| `templates/review-report.html` | kanban report template (light, polished, card-wall driven) |
| `templates/now-pointer.md` | now.md milestone pointer template |
| `state/watermark.json` | incremental dedup watermark (not in git) |
| `config/paths.json` | your vault & log paths (not in git) |

---

## Architecture

knowledge-capture uses a **dual-adapter** model — symmetric read and write sides, joined by a unified Session IR:

```
session logs (zcode / Claude Code / Codex / …)
        │
        ▼  read adapters (one .md per tool)
   ┌─────────────────────┐
   │     Session IR       │  ← the single shared contract
   │  {events:[{role,     │     (downstream only knows IR,
   │   kind, content}]}   │      never raw formats)
   └─────────────────────┘
        │
        ▼  capture-rules (digest → drafts)
   inbox drafts  ──────►  now.md milestone pointer  (read-back loop ✓)
        │
        ▼  organize (on demand)
   kanban HTML report  ──►  you say "OK" or nudge  ──►  write adapters promote to permanent notes
```

### Two modes, strictly separated

| Trigger | Mode | Automation | What you do |
|---|---|---|---|
| 「记录会话」/ `/kc save` | **save** | fully automatic | nothing |
| 「整理知识库」/ `/kc organize` | **organize** | semi-automatic | skim the report, say OK or push back |

**Why separate them**: if capture and review were one step, you'd be ambushed by a review every time a session ends. Splitting them keeps capture silent and zero-burden; review becomes periodic, batched, and on your terms.

### Session IR — the key to multi-tool support

Every agent tool's logs use a different storage format (JSONL, SQLite, Markdown, ZIP exports) and different field names. There is **no universal parser**. But the *semantics* of "what's worth capturing" are shared — what the user said, what the AI thought, what tools it used.

All read adapters parse their logs into one **Session IR**. Downstream rules (capture / organize) are written once and work for every tool. Adding a tool = adding one `.md` adapter. No compilation.

Tools cluster into three families that share parsing skeletons:

| Family | Tools | Shared by |
|---|---|---|
| Typed-event JSONL | zcode, Claude Code, Codex | `JsonlEventFamily` skeleton |
| Single-file JSON session | Cline/Roo, Zed, Cody | (Phase 2) |
| Tree/list JSON export | ChatGPT export, Claude.ai export | (Phase 3) |

---

## Kanban Report

When you run organize, the skill generates a **static, polished HTML report** and opens it in your browser:

- **Flow bar** at top — one colored strip showing how drafts split (promote / merge / archive) at a glance.
- **Card wall grouped by destination room** — each card shows only a type pill + title + source. **No prose.** Click for a one-line summary and the reason for its suggested destination.
- **Merge lane** — `draft A + draft B → target` chip flow.
- **Archive lane** — single-line items with reasons, so you can catch false-discards.

You review visually, then in the terminal say "OK" to commit or describe what to change. The skill regenerates the full report until you're satisfied. **Nothing is written until you approve.**

---

## Safety

| Concern | Guarantee |
|---|---|
| Re-processing the same logs | Watermark incremental dedup — never re-captures |
| Wrong destination / AI misjudgment | Report review is the backstop; all suggestions, nothing committed until OK |
| Accidental deletion | Archive ≠ delete; discarded drafts move to `90.archive/discarded/`, reversible |
| Corrupting your formal notes | Atomic staging commits — all target files land in `.staging/` first, move to formal rooms only if all succeed |
| Broken wikilinks | Pre-commit scan; if a target is referenced, keeps a redirect, never silently breaks |
| Touching your identity layer | Write-allowlist hard guard — skill can only write inside `30.knowledge/`, never `ME.md` / identity / skills |

---

## Agent Compatibility

The skill is plain markdown, so any AI agent that can read files and follow instructions can use it.

- **zcode** — reads from `~/.zcode/cli/rollout/*.jsonl` (Phase 1 ✅)
- **Claude Code** — reads from `~/.claude/projects/<dir>/*.jsonl` (Phase 1 ✅)
- **Codex CLI** — reads from `~/.codex/sessions/` (Phase 2, same JSONL family)
- **Cline / Roo Code, Zed, Cody** — single-file JSON sessions (Phase 2)

**Triggering**: CLI agents (zcode, Claude Code, Codex, Aider) can be scheduled via cron. GUI agents (Cursor, Windsurf) have no clean CLI entry — use manual trigger. The skill itself is a passive executor; it doesn't care how it's invoked.

> **Not supported** (no stable local logs): Cursor (SQLite+zlib, undocumented schema), Windsurf & Gemini (cloud-only), ChatGPT desktop (encrypted).

---

## Ecosystem

knowledge-capture is an independent repo that also fits into the [**skillpack**](https://github.com/beiyuii/skillpack) matrix. Three independent pieces, composable into a full personal-AI workflow:

| Repo | Role | Direction |
|---|---|---|
| **knowledge-capture** (this) | session logs → knowledge base | capture & promote |
| [personal-api-skill](https://github.com/beiyuii/personal-api-skill) | Obsidian vault → AI identity layer | identity & navigation |
| [skillpack](https://github.com/beiyuii/skillpack) | local skill farm + matrix orchestrator | install & manage |

**How they connect**: `personal-api-skill` scaffolds your vault (ME.md, Knowledge Palace v2 structure) and makes any AI agent understand who you are. `knowledge-capture` then feeds that vault with captured insights — it promotes drafts into the very `30.knowledge/` rooms that `personal-api-skill` set up, following the same Knowledge Palace v2 rules. `skillpack` installs and orchestrates both.

```
personal-api-skill ──scaffolds──► vault (ME.md + 30.knowledge/)
                                        ▲
knowledge-capture ────promotes into──────┘   (follows KP v2 rules)
        ▲
skillpack ──install──► both skills
```

---

## Phase Roadmap

- ✅ **Phase 1 (current, verified end-to-end on real vault)** — zcode + Claude Code adapters, save & organize both modes, kanban report, Obsidian-KP vault adapter.
- ⏳ **Phase 2** — Codex / Cline-Roo adapters · `skillpack skill install` command · cron scheduling via hermes.
- 🔜 **Phase 3** — ChatGPT/Claude.ai export adapters · weekly reminder reports · Logseq/Notion vault adapters.

---

## Design Docs

- [Spec](https://github.com/beiyuii/skillpack/blob/main/docs/superpowers/specs/2026-06-24-knowledge-capture-design.md) — full design (5-section brainstorming)
- [Phase 1 Plan](https://github.com/beiyuii/skillpack/blob/main/docs/superpowers/plans/2026-06-24-knowledge-capture-phase1.md) — implementation plan (Plan-reviewed)
- [Validation](./docs/phase1-validation.md) — end-to-end real-vault verification record

License: MIT.
