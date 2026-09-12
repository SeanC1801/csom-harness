# Harness System

A personal system for planning hackathon projects inside this Obsidian vault, using Claude directly (via Claude Code/Desktop) instead of a metered API. It turns a rough idea into a documented, scored decision, then into a full technical proposal ready to hand to a fresh Claude chat that does the actual building.

> This README file exists to get you oriented. Once you understand how the system works, feel free to delete your local copy of it.

```
idea → CSOM → project-doc → PROPOSAL.md → drag into a fresh Claude chat → build
```

## Contents

- [Philosophy](#philosophy)
- [Folder structure](#folder-structure)
- [The two working skills](#the-two-working-skills)
- [Status](#status)
- [Using it, step by step](#using-it-step-by-step)
  - [1. Run CSOM](#1-run-csom)
  - [2. Run project-doc](#2-run-project-doc)
  - [3. Take PROPOSAL.md to a fresh Claude chat](#3-take-proposalmd-to-a-fresh-claude-chat)
- [Future ideas](#future-ideas)
- [License](#license)

---

## Philosophy

**Fully human-documented, AI-assisted only.** This system does not build projects autonomously. It automates the *documentation* process, recording decisions into organized files, so nothing is lost and nothing has to be re-explained from scratch. Every meaningful step requires explicit human confirmation before anything is written:

- The participant makes every final decision. The AI suggests; it never decides.
- Nothing gets written to disk until the user explicitly confirms.
- Every score or claim must be justified, not asserted, and roughly half of each justification must come from the user's own words, not invented after the fact.
- Rejected options and skipped steps are recorded honestly, not hidden.

Delivery model partly inspired by Metropolis (a separate project): using an already-paid-for Claude subscription instead of a metered API, so cost stays fixed regardless of usage. Metropolis's own architecture (local BM25 ranking, no LLM call at all) doesn't apply here, its job is retrieval, this system's job is generation, but the underlying delivery philosophy, don't own a metered API call if the user already has a subscription that can do the work, carried over directly.

---

## Folder structure

```
00_CORE/       routing and (currently dormant) execution control
01_SPECS/      active specs and task tracking for this harness system itself
02_CONTEXT/    architecture, business rules, and the skills that do the work
03_TRACKING/   tech debt, learnings, blockers, audit findings
04_LOGS/       execution history (empty until something actually executes)
```

`00_CORE/agents.md` is the authoritative routing table, read it first to find any file. `Folder Branches.md` (vault root) has a visual tree of the same structure.

> **Note:** files here (`01_SPECS/specs.md`, `02_CONTEXT/architecture.md`, etc.) describe *this harness system's own* state. Each individual project gets its own, separately-scoped copies of these same filenames under `PROJECTS/<slug>/`, produced by the skills below. They are never the same file.

---

## The two working skills

Both live in `02_CONTEXT/skills/`, and are the only part of this system currently doing real work, everything else here is routing and reserved structure.

| Skill | Runs | Does |
|---|---|---|
| `csom.md` | first | Takes a rough idea through objectives, scope, reflective questioning, and a scored comparison of candidate approaches, then writes a per-project mini-harness to `PROJECTS/<slug>/`. |
| `project-doc.md` | second, after CSOM | Reads what CSOM wrote, adds technical depth, optionally establishes interface contracts, synthesizes a risk view, and assembles one self-contained `PROPOSAL.md`. |

Full walkthrough of both: [Using it, step by step](#using-it-step-by-step).

---

## Status

| | |
|---|---|
| ✅ | `csom.md` and `project-doc.md`, both tested end to end |
| 🟡 | `00_CORE/prompt.md` (the "Ralph Loop") is an intentionally dormant stub, reserved for a possible future autonomous execution mode, not currently needed or used. Everything runs by manually invoking each skill in a Claude Code conversation |
| 🟡 | `02_CONTEXT/rules/` and `04_LOGS/execution-logs/` exist but are empty, nothing has needed them yet |

---

## Using it, step by step

Everything happens inside a Claude Code conversation started in this vault.

### 1. Run CSOM

Mention it by name (or reference `HARNESS SYSTEM/02_CONTEXT/skills/csom.md`) and describe your idea. CSOM walks you through, in order:

1. Confirming a project slug and checking `PROJECTS/<slug>/` doesn't already exist.
2. Getting your idea, verbatim, no rephrasing.
3. Searching the vault for anything already there worth reusing.
4. Objectives, in-scope items, and one explicit non-goal.
5. Reflective questions, each one tied to a specific file it's gathering material for, not generic requirements-gathering.
6. Proposing candidate approaches, scored across five axes (Compatibility, Efficiency, Difficulty, Risk/Bottlenecks, Overall), with justifications drawn half from your own words, half from its own read.
7. You approve, edit, or reject each option.
8. A confirmed build order.
9. One final preview of every file it's about to write, and an explicit yes/no before anything touches disk.

**Result:** a mini-harness at `PROJECTS/<slug>/`:

```
PROJECTS/<slug>/
  00_CORE/overview.md
  01_SPECS/specs.md
  01_SPECS/tasks.md
  02_CONTEXT/architecture.md
  02_CONTEXT/business-logic.md      (only if something real came up)
  03_TRACKING/blocked.md            (only if something real came up)
  03_TRACKING/learnings.md          (only if something real came up)
  03_TRACKING/tech-debt.md          (only if something real came up)
```

### 2. Run project-doc

Once CSOM has finished for that project, invoke `project-doc.md` on the same slug. It:

- Reads everything CSOM wrote, it does not redraft objectives, scope, or the scored table.
- Appends real technical depth to `architecture.md`, how each approved piece actually gets built.
- Asks whether this is a team effort with parallel builders on a deadline, or solo/small-scale. Only for a team does it draft interface contracts between systems (locked shapes and function names at each boundary, so parallel builders can't drift apart); solo projects skip this step entirely.
- Synthesizes a consolidated `risks.md` from whatever's scattered across the tracking files.
- Writes `PROPOSAL.md`, one self-contained document with everything inlined, no "see other file" pointers, and rejected options deliberately left out.

### 3. Take PROPOSAL.md to a fresh Claude chat

Drag the file in. That chat has no access to this vault, `PROPOSAL.md` is the entire context it will ever get, which is exactly why steps 1 and 2 insist on it being complete and self-contained. Build from there.

> A full worked example of all three steps is at `PROJECTS/test-dry-run/`, kept intentionally as a reference for what correct output actually looks like.

---

## Future ideas

- **A free-tier path for people without Claude Pro.** This system currently assumes a flat-cost Claude subscription, that's the whole point (fixed cost instead of a metered API you can burn through). A collaborator is currently working on a free-tier alternative so people without a Pro subscription can still use CSOM. Not yet merged; tracked here so the intent is visible even before the implementation lands.
- Session resuming and a lighter-weight CLI, the same ideas CHOM's own README tracks under its own Future ideas section, likely worth mirroring here once the core workflow has more real usage behind it.

---

## License

MIT, see [LICENSE](LICENSE).
