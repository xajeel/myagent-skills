---
name: plan
description: "Diagnose a ticket — bug or feature — and teach the developer what is going on BEFORE anything is built. Reads the real code, traces the flow in plain language (what happens, what should happen, why the gap), then proposes production-grade solutions with pros and cons. Pure discussion: it never writes or edits code. On agreement it writes .sdlc/specs/<feature>/walkthrough.md, which /spec and /automate adopt. USE WHEN: 'plan X', a ticket or bug report is pasted, 'why is this happening', 'how should we build X' — before /spec on anything non-trivial."
argument-hint: "The ticket, bug report, or feature ask (text, doc path, or roadmap item)"
---

# plan

The thinking stage. One job: by the end of the conversation the developer can explain the problem AND the chosen solution to anyone — before a line of code exists. Run this stage on the strong tier, in its own fresh agent (PROJECT.md `Agents:` line).

## Procedure

**1. Load** `.sdlc/PROJECT.md`, `.sdlc/CRAFT.md`, `.sdlc/STATE.md`, and `.sdlc/BUGS.md` if present. PROJECT.md or CRAFT.md missing → run sdlc-init first, then return here.

**2. Classify the ticket** — say which in one line:
- **bug** — something behaves wrong today
- **feature** — a new capability or a change to one
- **whole app** — a PRD or multi-feature ask → hand to /roadmap; plan runs later, per feature

Pick the feature's kebab-case name now — reuse the existing spec folder if one exists.

**3. Investigate — read code, stay lean.** Trace the real flow: grep first, then read only what matters — ≤ 10 file reads.
- **Bug:** find the exact path from the user's action to the wrong behavior, and the root-cause line(s). Reproduce with a command when PROJECT.md makes that cheap. Never guess: if the cause isn't provable from the code, say exactly what would confirm it.
- **Feature:** find where the new flow attaches — the routes, services, models it touches — and the existing pattern new code should mirror.

**4. Teach it in chat — this message is the product.** Sections in order:

For a **bug**:
- **What the ticket says** — 2–3 plain sentences.
- **What actually happens** — the trace: "when the user does X, `a()` [path:line] runs → calls `b()` [path:line] → it should return Y but returns Z". ASCII flow ≤ 8 lines when the path has 3+ hops.
- **Why it happens** — the root cause in everyday words, citing the exact line(s).
- **The fix** — the recommended solution. When 2–3 real options exist: a short pros/cons table and ONE clear recommendation.

For a **feature**:
- **What we are building** — 2–3 plain sentences.
- **How it will flow** — the journey from the user's action to the result, naming each moving part and where it will live; what exists today vs what gets added.
- **Solution options** — the recommended production approach; alternatives with pros/cons only when they genuinely compete.
- **What it touches** — existing files and behavior at risk.

**5. Refine.** The developer pushes back, asks, changes direction — keep updating the sections in chat until they agree. A question you can't answer from the code → say so honestly and offer how to find out.

**6. On agreement** — write `.sdlc/specs/<feature>/walkthrough.md` from the template. Then: "Next: `/spec <feature>` to break this into tasks — or `/automate <feature>` to run the whole pipeline on this plan."

**Under /automate:** skip steps 4–5 — investigate, take the recommended production solution, write walkthrough.md with every Decision marked `(auto)`, continue.

## Writing rules — non-negotiable

- Written for the developer, not for the model: define every technical term in brackets at first use, short sentences, no acronym soup.
- Every claim about the code cites `path:line`.
- The test: the developer could re-explain the problem and the plan to a teammate without opening the code.

## Hard rules

- NEVER write, edit, or scaffold code — not even "just a quick fix". The output is understanding plus walkthrough.md.
- No task lists, file whitelists, or schemas — that is /spec's job.
- Solutions must be production-grade: correct, secure, scalable — the option a senior engineer would defend. A quick hack is never presented as an equal option. Check BUGS.md: never propose an approach it records as having failed.

## walkthrough.md template

```markdown
# <feature> — walkthrough
Type: bug | feature · Agreed: <YYYY-MM-DD>

## The ticket
<2–3 plain sentences — what was reported or asked>

## What happens now
<the traced flow with path:line cites; for a bug, where and how it breaks>
<optional ASCII trace, ≤ 8 lines>

## Why                      <!-- bug only: root cause in everyday words -->
<the cause + exact line(s)>

## The solution
<what we chose; the flow after the change, plain words>

## Why this solution
- **<Decision>** — chose <X> because <plain reason>. Rejected <Y> — <why not>.

## Open risks
<existing behavior that could break — /spec turns these into QA targets>
```
