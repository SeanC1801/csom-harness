# Project Doc — Technical Proposal Generator

Turns a CSOM-populated project folder into a full technical proposal: adds missing technical depth to the architecture, synthesizes a consolidated risk view, and assembles one self-contained connecting document. Read this after `csom.md` has finalized a project's files in `PROJECTS/<slug>/`, or when the user directly asks for a project proposal document.

Format modeled on the Metropolis backend proposal style: Executive Summary, Objectives & Scope, Systems Overview, Architecture, Technical Appendix, Risks & Open Questions, Recommendation & Next Steps.

**This skill does not draft `00_CORE/overview.md`, `01_SPECS/specs.md`, `01_SPECS/tasks.md`, or the scored table in `02_CONTEXT/architecture.md` from scratch, CSOM already wrote those. Read them; do not overwrite them unless the user explicitly asks for a change.**

## Non-negotiables

- Require that `PROJECTS/<slug>/` already exists with CSOM's output before running. If it doesn't, tell the user to run CSOM first rather than drafting scope/objectives yourself.
- Every claim in the risk synthesis must trace back to something CSOM actually captured (the architecture table's Risk/Bottleneck column, `blocked.md`, `tech-debt.md`, or `learnings.md`), not invented filler.
- Do not overwrite an existing `02_CONTEXT/architecture.md`'s scored table; only append additional technical detail underneath it.
- Do not overwrite an existing `03_TRACKING/risks.md` or `PROPOSAL.md` without asking first.
- `PROPOSAL.md` shows only the approved decision, never rejected options. The full comparison (including what was rejected and why) stays in `02_CONTEXT/architecture.md` as the vault's internal record.
- Contracts (step 3) are not a suggestion the user can silently skip past. If they haven't explicitly agreed to a contract, it isn't locked, and this skill must say so rather than treat silence as agreement.

## What this skill adds to the folder

```
PROJECTS/<slug>/
  00_CORE/overview.md          — (already exists, from CSOM — read only)
  01_SPECS/specs.md             — (already exists, from CSOM — read only)
  01_SPECS/tasks.md              — (already exists, from CSOM — read only)
  02_CONTEXT/architecture.md      — (scored table exists from CSOM — this skill APPENDS technical depth)
  02_CONTEXT/business-logic.md     — (may exist from CSOM — read only if present)
  03_TRACKING/blocked.md            — (may exist from CSOM — read only if present)
  03_TRACKING/learnings.md           — (may exist from CSOM — read only if present)
  03_TRACKING/tech-debt.md            — (may exist from CSOM — read only if present)
  03_TRACKING/risks.md                 — NEW, synthesized by this skill
  04_LOGS/                              — left empty, execution hasn't started
  PROPOSAL.md                            — NEW, the self-contained connecting document
```

## Flow

### 1. Confirm the project and read what CSOM already wrote

Confirm the project slug with the user. Read every file that exists under `PROJECTS/<slug>/`. If `00_CORE/overview.md` or `01_SPECS/specs.md` is missing, stop, tell the user CSOM needs to run first, do not draft substitutes.

### 2. Expand architecture with technical depth

Append to `02_CONTEXT/architecture.md` (below CSOM's existing scored table), one subsection per approved system: how it's actually built, its interface with the other systems, and, where they exist, schema/API contract/code examples. This is genuinely new content this skill contributes, CSOM's table says *what* was chosen and *why*; this section says *how* it gets built. Do not touch the scored table itself.

If `02_CONTEXT/business-logic.md` exists, fold its rules into the relevant system's subsection rather than leaving it disconnected.

### 3. Establish contracts between systems — ask first, don't assume

This step exists to prevent integration problems between people building different parts *in parallel*. That risk doesn't exist for a solo builder working through systems one at a time, there's no other person to misalign with. Before drafting anything, ask the user directly: **"Is this a team effort with people building parts in parallel against a deadline, or a small-scale project you're building yourself?"**

- **Solo or small-scale**: skip this step entirely. No `## Contracts` heading, no interface locking, nothing added to `PROPOSAL.md`'s Architecture section beyond what step 2 already wrote. Say explicitly that this step was skipped and why, don't just silently omit it.
- **Team effort with a deadline**: proceed below, and also make sure `00_CORE/overview.md`'s Recommendation & Next Steps reflects a sprint framework (contract first, then parallel independent building, then a sync/integration phase), not a simple sequential list, since a team building in parallel needs that structure, not a single ordered checklist meant for one builder.

For each boundary, using whatever notation fits the project's actual stack (Python type hints, TypeScript interfaces, or a plain shape sketch, whichever the systems already use):

- **Function or event name**: what crosses this boundary is called.
- **Input shape**: what the receiving system needs, at minimum.
- **Output shape**: what the sending system produces.

Do not invent field-level detail nobody discussed, name the boundary, its input, and its output, only as specifically as the conversation actually supports. If a boundary is genuinely still undecided, say so explicitly rather than filling it in.

Present the drafted contracts to the user and get explicit agreement before treating them as locked, the same "can everyone agree to this" check the source guide requires. Append the agreed contracts to `02_CONTEXT/architecture.md`, under a `## Contracts` heading, below the per-system technical depth from step 2.

### 4. Synthesize risks

Write `03_TRACKING/risks.md`, combining:

- Every Risk/Bottleneck score and its `Why` from `02_CONTEXT/architecture.md`'s table, restated as a plain bulleted risk.
- The full contents of `03_TRACKING/blocked.md`, if it exists.
- The full contents of `03_TRACKING/tech-debt.md`, if it exists, each compromise is also a risk.
- Anything from `03_TRACKING/learnings.md` that reads as a genuine open question, not just a settled observation.

Each item needs one sentence on why it matters. Do not pad; if a category contributed nothing, it contributes nothing.

### 5. Write the connecting document

The user's actual workflow is to drag `PROPOSAL.md` into a separate, fresh Claude chat that will do the building. That chat has no access to this vault, so this document is the entire context that chat will ever get. Write `PROPOSAL.md` at `PROJECTS/<slug>/PROPOSAL.md`:

```markdown
# <Project Name> — Technical Proposal

Prepared for: <user's name, if known>
Document type: Technical Proposal

## Executive Summary
<Full content from 00_CORE/overview.md's Executive Summary.>

## Objectives & Scope
<Full content from 01_SPECS/specs.md — objectives, in scope, and non-goal, in full.>

## Systems Overview
<The complete systems table from 01_SPECS/specs.md.>

## Architecture
<Only the approved component(s) from 02_CONTEXT/architecture.md — the chosen choice, its Why, and the technical-depth subsection from step 2. Do not include rejected options; this is the final decision, not the working comparison.>

## Contracts
<Only if step 3 determined this is a team effort. The agreed contracts, one per system boundary: name, input shape, output shape. If a boundary was left undecided, say so explicitly rather than omitting it silently. Omit this entire heading for a solo/small-scale project, do not include an empty or "not applicable" section.>

## Risks & Open Questions
<The full content of 03_TRACKING/risks.md from step 4.>

## Recommendation & Next Steps
<Full content from 00_CORE/overview.md's Recommendation & Next Steps, plus the task list from 01_SPECS/tasks.md.>
```

**This document must be fully self-contained**, except for rejected options, which are deliberately left out. Nothing in it may say "see X for detail" or assume the reader can reach another file. Inline everything else, the split files exist for this vault's own future harness routing, not as a shortcut to keep the connecting document short. `PROPOSAL.md` is the clean final decision; `02_CONTEXT/architecture.md` (with the full comparison, rejected options included) stays behind in the vault as the internal record of how that decision was reached.

### 6. Confirm with the user

Report the full file tree, including which files were read from CSOM versus newly written by this skill. Do not consider the task complete until `PROPOSAL.md` is confirmed to exist and has been checked against the self-containment rule above.
