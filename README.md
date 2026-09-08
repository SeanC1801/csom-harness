# Harness System

A personal system for planning hackathon projects inside this Obsidian vault, using Claude directly (via Claude Code/Desktop) instead of a metered API. It turns a rough idea into a documented, scored decision, then into a full technical proposal ready to hand to a fresh Claude chat that does the actual building.

## Philosophy

**Fully human-documented, AI-assisted only.** This system does not build projects autonomously. It automates the *documentation* process, recording decisions into organized files, so nothing is lost and nothing has to be re-explained from scratch. Every meaningful step requires explicit human confirmation before anything is written:

- The participant makes every final decision. The AI suggests; it never decides.
- Nothing gets written to disk until the user explicitly confirms.
- Every score or claim must be justified, not asserted, and roughly half of each justification must come from the user's own words, not invented after the fact.
- Rejected options and skipped steps are recorded honestly, not hidden.

## Folder structure

```
00_CORE/       — routing and (currently dormant) execution control
01_SPECS/      — active specs and task tracking for this harness system itself
02_CONTEXT/    — architecture, business rules, and the skills that do the work
03_TRACKING/   — tech debt, learnings, blockers, audit findings
04_LOGS/       — execution history (empty until something actually executes)
```

`00_CORE/agents.md` is the authoritative routing table, read it first to find any file. `Folder Branches.md` (vault root) has a visual tree of the same structure.

**Important**: files here (`01_SPECS/specs.md`, `02_CONTEXT/architecture.md`, etc.) describe *this harness system's own* state. Each individual project gets its own, separately-scoped copies of these same filenames under `PROJECTS/<slug>/`, produced by the skills below. They are never the same file.

## The two working skills

Both live in `02_CONTEXT/skills/`, and are the only part of this system currently doing real work, everything else here is routing and reserved structure.

1. **`csom.md`** (Context Skill for Obsidian Markdown) — run this first. Takes a rough idea through objectives, scope, reflective questioning, and a scored comparison of candidate approaches, then writes a per-project mini-harness to `PROJECTS/<slug>/`.
2. **`project-doc.md`** — run this second, after CSOM. Reads what CSOM wrote, adds technical depth, optionally establishes interface contracts (only for team projects with parallel builders, explicitly skipped for solo work), synthesizes a risk view, and assembles one self-contained `PROPOSAL.md`.

**The actual workflow**: run CSOM, then project-doc, then drag the resulting `PROPOSAL.md` into a separate, fresh Claude chat. See "Using it, step by step" below for the full walkthrough.

## Status

- ✅ `csom.md` and `project-doc.md`, both tested end to end.
- 🟡 `00_CORE/prompt.md` (the "Ralph Loop") is an intentionally dormant stub, reserved for a possible future autonomous execution mode, not currently needed or used. Everything above runs by manually invoking each skill in a Claude Code conversation.
- 🟡 `02_CONTEXT/rules/` and `04_LOGS/execution-logs/` exist but are empty, nothing has needed them yet.

## Using it, step by step

Everything happens inside a Claude Code conversation started in this vault.

**1. Run CSOM.** Mention it by name (or reference `HARNESS SYSTEM/02_CONTEXT/skills/csom.md`) and describe your idea. CSOM walks you through, in order:

- Confirming a project slug and checking `PROJECTS/<slug>/` doesn't already exist.
- Getting your idea, verbatim, no rephrasing.
- Searching the vault for anything already there worth reusing.
- Objectives, in-scope items, and one explicit non-goal.
- Reflective questions, each one tied to a specific file it's gathering material for, not generic requirements-gathering.
- Proposing candidate approaches, scored across five axes (Compatibility, Efficiency, Difficulty, Risk/Bottlenecks, Overall), with justifications drawn half from your own words, half from its own read.
- You approve, edit, or reject each option.
- A confirmed build order.
- One final preview of every file it's about to write, and an explicit yes/no before anything touches disk.

Result: a mini-harness at `PROJECTS/<slug>/` (`00_CORE/overview.md`, `01_SPECS/specs.md` and `tasks.md`, `02_CONTEXT/architecture.md`, plus `business-logic.md`/`blocked.md`/`learnings.md`/`tech-debt.md` wherever something real actually came up).

**2. Run project-doc.** Once CSOM has finished for that project, invoke `project-doc.md` on the same slug. It:

- Reads everything CSOM wrote, it does not redraft objectives, scope, or the scored table.
- Appends real technical depth to `architecture.md`, how each approved piece actually gets built.
- Asks whether this is a team effort with parallel builders on a deadline, or solo/small-scale. Only for a team does it draft interface contracts between systems (locked shapes and function names at each boundary, so parallel builders can't drift apart); solo projects skip this step entirely.
- Synthesizes a consolidated `risks.md` from whatever's scattered across the tracking files.
- Writes `PROPOSAL.md`, one self-contained document with everything inlined, no "see other file" pointers, and rejected options deliberately left out.

**3. Take `PROPOSAL.md` to a fresh Claude chat.** Drag the file in. That chat has no access to this vault, `PROPOSAL.md` is the entire context it will ever get, which is exactly why steps 1 and 2 insist on it being complete and self-contained. Build from there.

A full worked example of all of the above is at `PROJECTS/test-dry-run/`, kept intentionally as a reference for what correct output actually looks like.
