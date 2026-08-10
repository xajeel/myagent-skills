---
name: spec
description: "Turn an agreed plan into an executable spec — .sdlc/specs/<feature>/spec.md: machine-facing tasks with contracts and verify commands, nothing else. Adopts the walkthrough from /plan (creates one itself if none exists); every human explanation and decision lives in walkthrough.md, never in the spec. USE WHEN: 'spec X', after /plan agreement, 'plan roadmap feature 3' when you want tasks directly. Produces the file /test and /build execute."
argument-hint: "The feature to spec (e.g. 'user-auth', 'roadmap feature 3')"
---

# spec

Produces the machine-facing plan: tasks small enough that a mid-tier model implements each one without judgment calls. Human understanding lives in `walkthrough.md` (from /plan); `spec.md` is contracts and instructions only — no prose, no decisions, no teaching. Run on the strong tier (PROJECT.md `Agents:`), in its own fresh agent.

## Procedure

**1. Load — these files and nothing else:**
- `.sdlc/PROJECT.md`, `.sdlc/CRAFT.md` — missing → run sdlc-init first, then return.
- `.sdlc/STATE.md` — what already shipped; never re-plan existing work.
- `.sdlc/BUGS.md` — if it exists: no task may reintroduce a recorded bug; where one touches this feature's area, add a guarding Do bullet or Verify.
- `.sdlc/ROADMAP.md` — only if the user referenced a roadmap feature.

**2. Adopt or create the walkthrough.**
- `.sdlc/specs/<feature>/walkthrough.md` exists (from /plan) → it IS the agreed direction: design within it. Never re-litigate its decisions; if research below contradicts one, raise that in one line before proceeding.
- Missing → run the plan skill's investigate step yourself (lean: ≤ 10 file reads), take the recommended production solution, write walkthrough.md from plan's template. Decisions and every "why" go THERE — never into spec.md.

**3. Research the unknowns (no code yet).** For each component the solution needs that this codebase hasn't solved: find the standard production approach for the versions CRAFT.md pins — official docs / web search when tools allow, otherwise known best practice. Record each choice as a Decision line in walkthrough.md (plain English, jargon defined in brackets at first use).

**4. Integration scan.** Read ONLY the existing files this feature touches or extends. Record in `Touches`: files to be edited and what changes, pattern files new code must mirror, and existing behavior this could break (→ QA regression targets). Fold in walkthrough.md's Open risks.

**5. Gaps → MCQ.** Only unknowns that change the design. Max 5; options A–D with one `(Recommended)`. ≤ 3 → ask in chat; > 3 → write `.sdlc/questions.md`, wait for **done**, read answers, delete the file.

**6. Write `.sdlc/specs/<feature>/spec.md`** from the template. Task rules — /test and /build depend on these:
- **≤ 8 tasks.** Need more → split the feature; tell the user.
- Order by dependency. Each task independently verifiable.
- Each task ≤ 20 lines with exactly these fields:
  - `Files:` explicit CREATE / EDIT list — /build treats this as a whitelist
  - `Pattern:` existing file to mirror (omit if none)
  - `Do:` 3–7 imperative bullets — exact names, signatures, routes, data shapes, edge cases
  - `Verify:` one runnable command + expected outcome
- Code appears ONLY as: function/route signatures, request/response/schema shapes, and ≤ 5-line pseudocode for genuinely tricky logic. NEVER full file bodies.
- Every Do bullet must be decidable without judgment: "hash with bcrypt, cost 12" — not "hash securely".
- `Files:` must follow CRAFT.md Structure — one concern per file, each file in its concern's directory.
- A task that introduces a config value gets a Do bullet: read it via the settings module and add the key to `.env.example`. Never plan a hardcoded value.
- A task that adds a dependency names the exact package + version, consistent with CRAFT.md's pins.
- Do NOT write test tasks — /test owns the test suite. Name every public contract (signature, route, shape) precisely enough that /test can write tests from the spec alone.

**7. Present in chat, briefly** — /plan already taught the problem, so no re-teaching:
- each Decision from walkthrough.md as one line: `**<Component>** — chose <X> because <Y>`
- the task list, one line per task: `T1 — <name>`
- then: "Review the spec, then `/test <feature>` writes the tests and `/build <feature>` implements — or `/automate <feature>` runs the rest hands-free."

## Spec template

```markdown
# <feature>
Status: draft            <!-- draft → building → qa → done · owned by build/qa/ship -->
Goal: <one line>
Done when:
- <verifiable criterion>
- <verifiable criterion>

## Touches
- Edits: <existing files + what changes in each>
- Mirrors: <pattern file(s) new code must match>
- Risk: <existing behavior that could break — QA checks these>

## Tasks

### [ ] T1 — <name>
Files: CREATE src/…, EDIT src/…
Pattern: src/…
Do:
- <exact imperative instruction>
- <signature / shape where needed>
Verify: `<command>` → <expected>

### [ ] T2 — <name>
…

## Acceptance checks
- [ ] `<command>` → <expected observable result>

## Tests
<!-- appended by /test — read-only for /build -->

## QA log
<!-- appended by /qa -->
```

## Quality bar — check before saving

- [ ] A model with ONLY spec.md + PROJECT.md + CRAFT.md could implement every task
- [ ] /test could write the full suite from spec.md alone — every public contract is named
- [ ] Every task has a runnable Verify; every "Done when" maps to an Acceptance check
- [ ] No full code bodies; no vague bullets ("handle errors properly")
- [ ] spec.md contains ZERO explanation prose — every "why" lives in walkthrough.md
- [ ] Every `Files:` list lands each concern in its CRAFT.md directory — no multi-concern files
- [ ] No config value hardcoded anywhere in the plan — settings module + `.env.example` only
- [ ] Touches lists every existing file that will change
- [ ] No task reintroduces a bug recorded in `.sdlc/BUGS.md`
