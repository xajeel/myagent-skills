---
name: automate
description: "Hands-free pipeline for ONE ticket: plan → spec → test → build → qa, back-to-back, zero stops except real blockers. Adopts any stage already done (a walkthrough agreed in /plan, an existing spec) and continues from there; anything missing it does itself, taking the recommended production option and logging it (auto). Each stage runs as its own fresh agent on its PROJECT.md tier. USE WHEN: 'automate <x>', 'fix this ticket end to end', 'build X without asking me'."
argument-hint: "The feature/bug to run end-to-end (e.g. 'user-auth', 'roadmap feature 2', 'payments --ship')"
---

# automate

Runs the whole pipeline for one ticket without waiting for the user. Each stage follows its own SKILL.md to the letter — automate changes only WHO answers questions (the decision policy below) and WHO runs each stage (a fresh agent per stage, never the context that produced the previous stage's output).

## Stage separation — the bias guard

Fresh context per stage is what keeps the pipeline honest: the tester never sees the developer's code, QA never sees the developer's reasoning. Tiers come from PROJECT.md's `Agents:` line.
- Host supports subagents (Claude Code, Cursor, Windsurf, …): spawn each stage as a subagent, give it ONLY the files its SKILL.md lists, on the tier's model when the host allows choosing one.
- No subagents: run stages sequentially, but start each stage clean — discard prior reasoning, re-read only that stage's files.
- Token discipline: no stage re-reads the whole codebase; every SKILL.md caps its own reads. Never re-derive what an .sdlc file already records.

## Setup

1. Read `.sdlc/PROJECT.md`, `.sdlc/CRAFT.md`, `.sdlc/STATE.md`, and `.sdlc/BUGS.md` if present. PROJECT.md or CRAFT.md missing → stop: run /sdlc-init first (init is never automated — its approval gate needs a human).
2. Resolve the ticket to a feature name: from the argument, or by name/number from a `todo` roadmap entry. Already `done` in STATE.md → stop and say so.
3. **Adopt what exists — never redo an agreed stage.** Check `.sdlc/specs/<feature>/` and start at the first stage whose artifact is missing:

| Found | Start at |
|---|---|
| nothing | plan |
| walkthrough.md only | spec |
| spec.md without a `Tests` section | test |
| `Tests` section present, unchecked tasks remain | build |
| all tasks `[x]`, no QA PASS in the log | qa |

A walkthrough you agreed in /plan is honored as-is — automate builds YOUR plan, not its own.
4. Read PROJECT.md's `Skip stages:` line — skipped stages are bypassed.
5. Announce in ≤ 2 lines what will run — e.g. `automate user-auth → adopting your walkthrough → spec → test → build → qa — decisions auto-picked and logged` — then go. Do not wait for a reply.

## Decision policy — replaces every question

- A stage would ask an MCQ → take its `(Recommended)` option; none marked → the boring production standard for this stack — the choice a senior engineer would defend without a meeting. Record EVERY auto-made choice in walkthrough.md's "Why this solution" section, marked `(auto)`.
- /plan under automate: no discussion — investigate, take the recommended production solution, write walkthrough.md with decisions marked `(auto)`, continue.
- /build files a test challenge → run /test's challenge mode in a fresh agent; its verdict is final. The spec itself ambiguous → smallest `(auto-amended)` fix in CRAFT.md's favor, one line in chat, continue.
- Never auto-answer: anything destructive, anything touching credentials, payments, or data deletion, or a change to CRAFT.md itself. Those stop the run.

## Stages — in order, each per its own SKILL.md, each in a fresh agent

1. **plan** (strong) — auto mode: investigate, decide, write walkthrough.md. No chat discussion.
2. **spec** (strong) — full procedure; MCQs go through the decision policy.
3. **test** (strong) — tests from the spec, red before build; bug repro test first.
4. **build** (mid) — full task loop, every verify gate and checkpoint. The 3-attempt limit and the regression rule stay hard stops. Tester tests stay untouchable.
5. **qa** (strong) — full run; failures become F-tasks and BUGS.md entries. On FAIL → **build** the F-tasks, then **qa** again. Max 3 QA rounds; still failing → stop and report.
6. **ship** — NOT run by default; automation ends before the commit. Run only when ship is not skipped AND the user asked (`--ship` or "including ship") — ship's confirm step is then auto-approved, but its own guards (branch guard, staging by name, never push without `--pr`) still apply in full.

## Hard stops — automation never bulldozes these

- A verify or the suite still fails after 3 fix attempts → stop, report exactly like /build would.
- A regression it cannot fix → stop.
- A spec-vs-CRAFT.md conflict no amendment can fix → stop and ask.
- On any stop, report: the stage, what completed, what is blocking, and the exact command to resume — every stage is resumable from its artifacts.

## Final report — ≤ 15 lines

- 3 lines from walkthrough.md: the problem, the chosen solution, why
- Stages run and results: tasks done, tester tests red → green counts, QA verdict, suite totals
- Every `(auto)` and `(auto-amended)` decision, one line each — the audit trail
- Next step: `/ship <feature>` — or "feature complete" if ship was skipped or already run
