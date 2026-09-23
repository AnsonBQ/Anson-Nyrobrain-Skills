---
name: flowchart
description: Use when someone asks how a part of this system actually works end-to-end — "画 flowchart", "整个系统/pipeline 怎么跑", "big picture", "X 和 Y 怎么交互、什么触发什么", "这个文件是干嘛的、没有它会怎样", or after a new feature/pipeline has landed and the operator has lost the picture (专有名词太多,看不到全局). Produces file-anchored system maps — flows, trigger scenarios, failure points, file purposes — under docs/system-map/, with a freshness fingerprint so a map can prove it is not stale.
---

# /flowchart — system maps a CEO can act on

**What changes after this exists:** when any subsystem's workings become fog (too many
in-house terms — Sprint KB, FRONTIER, MCP shim…), running `/flowchart <target>` produces a
map of how it goes from 0 to 100 — every trigger, step, file and failure point anchored to
`file:line` — so the operator can point at a component and decide, instead of asking about
terms one by one.

## Invocation

```
/flowchart                  draw or refresh the skeleton overview (docs/system-map/index.md)
/flowchart <target>         draw or refresh one subsystem map (docs/system-map/<target>.md)
/flowchart <target> --check freshness report only: which input files changed since the map was drawn
```

`<target>` is a short subsystem id (`sprint-loop`, `kb`, `mining`, `portfolio`, …). Known
targets and their read sets are indexed in `docs/system-map/index.md`; a new target is
allowed — name it and define its read set on first draw.

## Contract

- **reads**: code (`brain/scripts/**`, `oms/**`, anything the subsystem executes), docs
  (`docs/**`, `AGENTS.md`, folder READMEs), run artifacts (`brain/sprint/INDEX.md`, recent
  run dirs), live state when it matters (LaunchAgent plists, a run's status/state JSON files).
- **writes**: only `docs/system-map/**` and `brain/scripts/system_map_check.py`'s subjects.
- **never**:
  - never edits code, config, KB content, or anything outside `docs/system-map/`;
  - never asserts a behavior without a `file:line` anchor read this run — an unanchored
    claim does not go on the map;
  - never trusts docs over code: when they disagree, **code wins** and the doc is named as
    stale (the 2026-09-08 lesson: measure, then judge);
  - never blocks on or disturbs a live run — everything here is read-only;
  - never syncs maps into the intake Obsidian vault — intake is one-way (vault → brain/),
    writing back creates a loop. To see Mermaid rendered, open the repo as an Obsidian
    vault instead;
  - never lists a file without saying why it exists and what breaks without it.

## Procedure

1. **Frame the target.** One paragraph: what this subsystem is for, where it starts (human
   command? cron? LaunchAgent? another pipeline?) and where it ends. If you cannot write
   this yet, you have not read enough — keep reading.
2. **Assemble the read set.** Entry points from the AGENTS.md router rows and folder
   READMEs, then the code those entry points execute, then one recent real run's artifacts.
   Record every file read — it becomes the fingerprint.
3. **Extract the machine, code-first.** Trace each trigger scenario to its end: step by
   step, what each step reads/writes (files, dirs, sqlite, services, terminals), where
   state lives between steps, and what can stop it (quotas, locks, gates, exhausted
   inputs, provider windows).
4. **Write the map** using the mandatory section layout below. ASCII tree first, Mermaid
   from the SAME numbered step list (the two must say the same thing; Mermaid stays simple
   `flowchart TD`, no exotic syntax — ASCII is the source of truth). Form follows content:
   block-and-arrow composition (the 一屏全局 style) for how parts connect; nested step-tree
   for sequential traces. A map may use both.
   **Alignment is correctness, not cosmetics** — a box whose borders drift is a wrong map.
   CJK glyphs are 2 columns: pad by display width, never by character count, and never rely
   on trailing spaces. Run `uv run brain/scripts/system_map_check.py <map> --align` and fix
   every flagged row before finishing.
5. **Verify anchors.** Every cited file exists; spot-check that cited lines say what the
   map claims. Run the check script once to record a valid fingerprint:
   `uv run brain/scripts/system_map_check.py docs/system-map/<target>.md --write`.
6. **Report to the operator** in plain language: the 3–5 things on this map that most
   constrain the subsystem (where it can deadlock, idle, or silently lie), not a retelling
   of the whole map.

## Mandatory map sections

```markdown
# <target> — 系统运作图（drawn YYYY-MM-DD by /flowchart）

## 0. 这张图是干嘛的        一段话:子系统目的、从哪触发、到哪结束
## 1. 名词表               专有名詞 | 人话解释 | owner file:line — 每个自造词都收
## 2. 触发场景             事件 | 入口 | 读什么/写什么 | 会卡在什么(quota、锁、窗)
## 3. 主流程 0→100         编号步骤,每步带 file:line;先 ASCII 分层树,后 Mermaid(同一步骤表)
## 4. 文件名册             文件 | 为什么存在 | 没有它会怎样 — 只收 load-bearing(删掉它,
                          CEO 会注意到行为变化,才配占一行)
## 5. Edge cases 与故障点   症状 | 根因 file:line | 真实事故(带日期,来自 sprint 报告/尸检)
                          — 没有真实事故的"可能会坏"不写(AGENTS.md §0:defence 要有场合)
## 6. 指纹与保鲜            inputs 清单 + sha1(短) + 绘制日期;--check 用法
```

Fingerprint block (machine-readable, at the end of section 6, kept in sync by the check
script's `--write`):

    ```system-map-inputs
    brain/scripts/sprint_runtime.py  sha1:1f92550f  2026-09-17
    ```

## Boundaries

- Not `domain-modeling`: that owns vocabulary/CONTEXT.md/ADRs; this owns **operational**
  flow — triggers, steps, files, failures. The 名词表 here is for orientation, not the
  domain model.
- Not `research`: research answers a question; this produces a standing, freshness-checked
  artifact. A research finding can be an input to a map.
- Maps are derived artifacts: refreshing overwrites that target's map in place (git keeps
  history). The workspace no-delete rule covers knowledge content, never derived maps.

## Anti-debt bar

A map that is shallow, stale, or unanchored is worse than none. Budget: skeleton overview
fits one screen; a subsystem map earns every line by changing a decision the operator could
make. If a section has nothing anchored to say, cut the section, say why in one line.
