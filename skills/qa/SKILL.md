---
name: qa
description: "Mechanical QA gate for a built feature — re-runs every task Verify, the tester's suite, Acceptance checks, full-suite regression, and a diff scan that includes PR-readiness (the diff must survive review by an outside frontier model). Failures become fix-tasks /build can execute. USE WHEN: 'qa X', 'test the feature', 'verify the implementation', after /build finishes."
argument-hint: "The feature to QA (must have a spec in .sdlc/specs/)"
---

# qa

Verifies a feature against its own spec. Runs on the strong tier (PROJECT.md `Agents:`) in a FRESH agent that never saw the implementation reasoning — QA judges the code, not the story behind it. Mechanical by design: the spec already defines every check — QA executes them, it does not wander the codebase. QA reports; /build fixes.

## Procedure

**1. Load** `.sdlc/PROJECT.md` + `.sdlc/CRAFT.md` + `.sdlc/specs/<feature>/spec.md`. (Legacy layout: a single `.sdlc/specs/<feature>.md` → treat it as spec.md.) Spec name unclear → list `.sdlc/specs/` and ask.

**2. Task verifies.** Re-run the Verify command of every `[x]` task. Record pass/fail per task.

**3. Tester suite.** Run every test file in the spec's `Tests` section. All green required — these ARE the feature's contract. Any red → FAIL, and check `git log`/diff that none was edited by build (an edited tester test is itself a CRITICAL failure).

**4. Acceptance checks.** Run each command under Acceptance checks; compare actual vs expected. If a check needs a live app, start it with PROJECT.md's `run:` command and stop it afterwards.

**5. Regression.** Run the full `test:` suite. Any failure outside this feature's own tests — especially in areas named under `Touches → Risk` — is a regression: always CRITICAL, always blocks PASS.

**6. Coverage gap.** An acceptance check not covered by any test → write ONE minimal test for it (following the project's test pattern), run it, keep it, and note it in the Tests section.

**7. Diff scan.** `git diff --name-only` (plus `git status --short` for untracked). Six checks:
- Every changed file appears in some task's `Files:` list or the Tests section — unlisted changes get flagged.
- Each changed file respects CRAFT.md Structure, Style bullets, and Boundaries — walk them mechanically.
- Scan changed source for hardcoded config/secrets — literal keys, passwords, tokens, URLs, ports that belong in the settings module + `.env.example`.
- Scan changed source for legacy-version APIs forbidden by CRAFT.md's Stack idiom lines.
- **PR-readiness** — this diff will be reviewed by an outside frontier model: no debug prints or leftover logging, no commented-out code, no dead code, no TODO/FIXME, no narration comments or bloated docstrings.
- Complexity smell — a function a mid-level developer couldn't trace (deep nesting, clever one-liners doing 3 things) gets flagged as a note; a structural mess (concerns mixed in one file) blocks.

**8. Verdict.**
- **PASS** — all task verifies, the tester suite, acceptance checks, and the full suite are green; no unlisted changes; no hardcoded secrets, structure, security, or idiom violations; PR-ready. Style nits alone don't block — list them as notes.
- **FAIL** — anything else. For each failure, append a fix-task to the spec's Tasks section — `### [ ] F1 — fix: <what>` with Files / Do / Verify filled in — so `/build <feature>` can execute the fixes directly.

**9. Bug log — every failure teaches.** For each failure found this run, append an entry to `.sdlc/BUGS.md` (create with a `# Bugs` heading if missing). Plain words, no stack traces, ≤ 6 lines:

```markdown
## B<n> — <one-line title> · <feature> · <YYYY-MM-DD> · open
- What went wrong: <1–2 sentences — what was done and what happened instead>
- Why it happened: <the root cause in everyday words>
- How to avoid it: <the rule or fix, one line, written to be reusable>
```

Same root cause as an existing entry → don't duplicate; add this feature to that entry's title line. When a rerun confirms a fix, flip `open` to `fixed`. /plan, /spec, and /build read this file so the same bug never ships twice.

**10. Append to the spec's QA log** (≤ 20 lines per run):

```markdown
### QA <date> — PASS | FAIL
Task verifies: 6/6 · Tester suite: 14/14 · Acceptance: 3/3 · Suite: 42 passed, 0 failed · Regressions: 0
Unlisted changes: none · Craft scan: clean · PR-ready: yes
Failures: <one line per F-task, or "none">
Notes: <convention nits, or "none">
Try it:
1. <command or click-path with concrete sample data>
2. <expected result>
```

The **Try it** block is 2–5 steps any human can follow to see the feature working: concrete sample data, exact expected outcome.

**11. Report in ≤ 8 lines:** verdict, the counts line, failures if any, next step — `/build <feature>` to execute fixes, or `/ship <feature>` on PASS. If ship is on `Skip stages:`, do ship's bookkeeping here (spec Status → `done`, STATE.md row → `done`, ROADMAP status if listed) and report the feature complete.

## Rules

- Never PASS with a failing acceptance check, a red tester test, or a regression. No "pass with warnings" for those.
- Never fix code here — failures become F-tasks for /build. (Sole exception: the minimal coverage test in step 6.)
- Never edit a tester-owned test — a wrong test goes through /test's challenge mode.
- Scope is this feature's diff plus the test suite — do not re-review the whole codebase.
- Environment blocks a check (no DB, no network)? Record it as SKIPPED with the reason — never silently count it as passing.
- Invoked directly while qa is on the skip list → run anyway (a direct call outranks the list) and note it in the report.
