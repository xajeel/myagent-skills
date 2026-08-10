---
name: test
description: "Write the feature's tests from the spec BEFORE implementation — a separate tester agent, so the developer model never writes the tests that judge its own code. Tests come from spec contracts + acceptance checks only; for a bug the first test reproduces it and must fail. /build may never edit these tests — it can challenge one, and this skill adjudicates. USE WHEN: after /spec and before /build; 'write the tests for X'; or /build reported a suspect test."
argument-hint: "The feature (must have .sdlc/specs/<feature>/spec.md) — or a test challenge from /build"
---

# test

The tester. Fresh agent, strong tier (PROJECT.md `Agents:`), and deliberately blind to the implementation: it reads the spec and walkthrough, never code the developer is about to write. The tests encode what SHOULD happen — the contract — so /build implements toward them, not the other way around.

## Write mode — after /spec

**1. Load** `.sdlc/PROJECT.md`, `.sdlc/CRAFT.md`, `.sdlc/specs/<feature>/spec.md` + `walkthrough.md`. Spec missing → run /spec first.

**2. Derive the cases — from the spec only:**
- every Acceptance check → at least one test
- every task contract: signatures, request/response shapes, edge cases named in Do bullets
- the standard edges the spec implies: empty/invalid input, unauthorized, not-found
- **bug feature** → the FIRST test reproduces the bug exactly as walkthrough.md describes it, and must fail on the current code — proof the cause was understood.

**3. Write** the tests where CRAFT.md's Structure puts them, mirroring the project's existing test pattern. Plain, readable tests: one behavior per test, real assertions, no loops or logic in test bodies, no testing private internals the spec doesn't name.

**4. Run them.**
- Every NEW test must be RED — the feature doesn't exist yet. A new test that passes on current code means the behavior already exists or the test asserts nothing: investigate, then fix or delete it.
- The EXISTING suite must stay green. If the tester broke it, fix your own files — nothing else.

**5. Record** — append to spec.md:

```markdown
## Tests — owned by /test · read-only for /build
- <test file path> — <n> tests · red (expected until built)
Covers: <acceptance checks + task contracts + "bug repro" if applicable>
```

**6. Report ≤ 5 lines:** files written, test counts, red/green status, next: `/build <feature>`.

## Challenge mode — /build says a test is wrong

Input: the test file + test name, build's reason, and the spec line it cites. Decide from the spec, never from the implementation:
- The test contradicts the spec → the test is wrong: fix it, rerun, add one line under the spec's Tests section (`fixed after challenge: <what>`).
- The test matches the spec → confirm in one line; /build must make the code satisfy it.
- The SPEC itself is ambiguous or self-contradictory → say so: the fix is a spec amendment. The user decides; under /automate, apply the smallest amendment consistent with CRAFT.md, marked `(auto-amended)`.

## Hard rules

- Touch ONLY test files. Never create or edit source files — not even scaffolding or fixtures that live outside the test layout.
- Never weaken a test to make it pass; never delete a red test because it is inconvenient.
- Tests import the exact paths and names the spec declares. The spec doesn't name them → that's a spec gap: raise it, don't invent.
- Keep the suite fast: no sleeps, no network unless the spec demands it (then mark that test clearly).
