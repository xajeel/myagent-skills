---
name: sdlc-init
description: "Set up AI-SDLC for a project — creates .sdlc/PROJECT.md (what it is, commands, git workflow, agent tiers), .sdlc/CRAFT.md (the code rulebook: pinned stack + idioms, structure, style, config, security), .sdlc/STATE.md (feature log), and rule stubs (AGENTS.md) so ANY coding tool follows the same rules. Run once per project, before any other SDLC skill. USE WHEN: 'set up sdlc', 'init project context', or any SDLC skill finds no .sdlc/ folder."
argument-hint: "Optionally: a PRD/description, a doc path, 'analyze the codebase', or --skip-<stage> / --unskip-<stage>"
---

# sdlc-init

Creates the files every other SDLC skill reads. PROJECT.md and CRAFT.md are loaded on every skill run, so every line must earn its place — facts and rules only, no prose.

CRAFT.md is the code rulebook: it is what makes ten build sessions — or ten different models — produce code that looks like one engineer wrote it. /plan and /spec design within its rules, /test and /build write by them, /qa checks the diff against them.

## Output contract

| File | Contains | Budget |
|---|---|---|
| `.sdlc/PROJECT.md` | What the project is, real commands, git workflow, agent tiers | ≤ 35 lines |
| `.sdlc/CRAFT.md` | Pinned stack + idiom rules, structure, style, config & secrets, security, boundaries | ≤ 80 lines |
| `.sdlc/STATE.md` | Feature status table + cross-feature notes | grows ~1 line per feature |
| `AGENTS.md` (repo root) | ≤ 8-line pointer so any coding tool obeys the same rules | append-only |

## Procedure

**0. Stage toggle?** If the argument is only skip/unskip flags (`--skip-ship`, `--unskip-qa`, or plain words like "skip ship") AND `.sdlc/PROJECT.md` already exists, this is a toggle, not a re-init: update the `Skip stages:` line, confirm in one line, stop. Skippable: `test`, `qa`, `ship` — `plan`, `spec`, and `build` are the irreducible core; refuse anything else with one line. (Skipping `test` is discouraged — say so in the confirmation — it removes the independent-tester bias guard.) On a fresh init the same flags pre-set the line. Skipped stages are bypassed by stage handoffs and /automate; invoking a skipped skill directly still runs it.

**1. Existing setup check.** If `.sdlc/PROJECT.md` or `.sdlc/CRAFT.md` exists and is non-empty: show a 3-line summary, ask — update, replace, or cancel. Wait for the answer.

**2. Gather facts — use the first source that applies:**
- **PRD or description provided** → read it. Extract: purpose, users, features, stack, constraints.
- **Codebase exists** → detect, don't interrogate:
  - Read manifests AND lockfiles (`pyproject.toml` + `uv.lock`/`poetry.lock`/`requirements*.txt`, `package.json` + its lockfile, `go.mod`, `Cargo.toml`, …), `docker-compose.yml`, `.env.example`, README.
  - Read the entry point, 2–3 representative source files (one per layer), and 1 test file.
  - Do NOT scan the whole codebase — ~12 file reads maximum.
- **Neither (greenfield, no PRD)** → question mode; steps 3–5 collect everything.

**3. Rules source — one MCQ, always asked.** "How should the code-craft rules be set?"
- A) Derive them from the codebase and show me `(Recommended when code exists)`
- B) Ask me questions and build them from my answers `(Recommended for a new project)`
- C) Propose best-practice rules yourself; I'll review before anything is written
- D) I'll provide my own rules — pasted here or a file path
Blank answer → the recommended option for the situation.

**4. Git workflow — one MCQ, always asked.** On an existing repo, look first (`git log --oneline -15`, branch names) and pre-select what the history shows. "How does this project manage branches and commits?"
- A) Feature branches off the default branch, PRs, Conventional Commits `(Recommended / default)`
- B) GitFlow — develop + feature/release/hotfix branches
- C) Trunk-based — short-lived branches, squash-merge to main
- D) My own — describe it (branch pattern, commit format, merge target)
Blank answer → A, marked `(assumed)`. The answer becomes PROJECT.md's `## Git workflow` section — /build branches by it, /ship commits and opens PRs by it, always.

**5. Build the draft rules.**
- **From code (A):** versions come from lockfiles/manifests — exact, never guessed. Structure comes from the real tree. Style rules only from patterns actually observed. Config rules from how the code reads settings today.
- **From answers (B):** MCQs for what steps 1–2 didn't answer: language + version, framework, DB, package manager, test runner, layout, auth approach, deploy target. Options A–D, one `(Recommended)`; blank = recommended. ≤ 3 questions → chat; > 3 → `.sdlc/questions.md` with `Answer:` lines, wait for **done**, read, delete. Hard cap: 7 questions.
- **From the user (D):** take their rules verbatim; MCQ only the gaps.
- **Best practice (C):** the boring industry-standard option for the project type.
- **Version pinning — greenfield, all paths:** pin the newest stable you can confirm — registry check (`pip index versions <pkg>`, `npm view <pkg> version`) or web search when tools allow; otherwise pin the newest you know, marked `(verify)` — the first build task must then confirm and correct CRAFT.md.
- **Idiom rules — all paths:** for every framework/ORM with a breaking major, one line naming the modern API and forbidding the legacy one — e.g. "SQLAlchemy 2.x — `DeclarativeBase`, `Mapped[]`, `mapped_column()`; 1.x style forbidden." This stops a model writing last-generation code against a current library.
- **Simplicity rules — all paths, non-negotiable:** the Style section always includes the four readability bullets pre-filled in the template below. They exist so any developer can trace the code — never drop them.

**6. Approval gate — never skip.** Show the draft in ≤ 15 lines: pinned stack, folder tree, git workflow, and the 5 rules that matter most. Ask: approve, or edit what? Apply edits and re-show until approved. Only then write files.

**7. Write the three .sdlc files** from the templates below.
- Style: max 12 bullets, each mechanically checkable.
- Commands must be real: run the test command once to confirm (skip on greenfield).
- No placeholders left in the final files.

**8. Rule stubs — make every coding tool obey the same rulebook.** Write this block to `AGENTS.md` at the repo root (create it, or append if the file exists — never overwrite existing content):

```markdown
## AI-SDLC rules
Before writing any code in this repo:
1. Read `.sdlc/CRAFT.md` and follow every rule — structure, style, config, security.
2. Read `.sdlc/PROJECT.md` — use its commands and its Git workflow for branches/commits.
3. Feature work lives in `.sdlc/specs/<feature>/` — spec.md is the task authority.
Keep code boring and readable: minimal comments, no long docstrings, no clever one-liners.
```

If `CLAUDE.md` or `.cursor/rules/` already exist, append/add the same pointer there so Claude Code, Cursor, Codex, Windsurf — any tool — lands on the same rules.

**9. Report in ≤ 6 lines:** files created, pinned stack, git workflow, skipped stages (if any), and the next step — `/roadmap` for a whole project, `/plan <ticket>` to diagnose a bug or design a feature, or `/automate <feature>` to run the whole pipeline hands-free.

## PROJECT.md template

```markdown
# <project name>
<one line: what it is and who uses it>

## Stack
<one line — full pinned versions live in CRAFT.md>

## Commands
- test: <exact command>
- lint: <exact command>
- run: <exact command>
- migrate: <exact command or "n/a">

## Git workflow
- Branches: <pattern, e.g. "feat/<feature>, fix/<feature> off main">
- Commits: <format, e.g. "Conventional Commits">
- Merge: <target + method, e.g. "PR to main, squash">

> Repos: <single | list each path + purpose if multi-repo>
> Skip stages: none   <!-- test, qa, ship — handoffs and /automate bypass these -->
> Agents: plan=strong · spec=strong · test=strong · build=mid · qa=strong
>   <!-- tiers, not vendor names: map strong/mid to your tool's best/faster models.
>        Every stage runs as its own FRESH agent — that separation is the bias guard. -->
```

## CRAFT.md template

```markdown
# Code Craft — <project>
Source: <derived from codebase | agreed with user> · <YYYY-MM-DD>

## Stack — pinned
- <language + version> · <package manager>
- <framework + version> — <modern idiom rule>
- <DB + ORM + version> — <idiom rule: modern API named, legacy API forbidden>
- <test runner + version>
> New dependency → latest stable, exact version recorded here in the same task.

## Structure
- <dir>/ → <one-line responsibility, e.g. "models/ → SQLAlchemy models only">
- <dir>/ → <…>
- <test location + naming, e.g. "tests/<area>/test_*.py">
- One concern per file: models, schemas, routes, services never share a file.
- New files go in the directory matching their concern — never at repo root.

## Style
- Comments only for what the code cannot say — no narration, no restating the line below.
- Docstrings one line, public API only. No essay docstrings.
- Boring beats clever: no dense one-liners, no unneeded abstractions, no premature patterns.
- Functions do one job and are short enough to read whole; a mid-level dev can trace any file top to bottom.
- <up to 8 more project-specific bullets, each mechanically checkable>

## Config & secrets
- All config read from environment variables in ONE settings module: <path>
- Every new key → `.env.example` with a placeholder, in the same task that introduces it
- Never hardcode secrets, URLs, ports, or keys in source

## Security baseline
- Passwords hashed with <bcrypt cost 12 | argon2id> — never logged, never returned
- DB access only through the ORM — never build SQL strings from input
- Every request body validated at the boundary (<validation tool>)
- Every non-public route checks auth before touching data
- Error responses never leak stack traces, queries, or internals

## Boundaries
- Never edit: <e.g. migrations/versions/*, generated files, lockfiles by hand>
- Never commit: .env, secrets, build artifacts — .env.example (placeholders only) IS committed
```

## STATE.md template

```markdown
# State

| Feature | Spec | Status | Shipped |
|---|---|---|---|

## Notes
<!-- cross-feature facts an implementer must know; one line each -->
```

## Edge cases

- Existing code contradicts best practice → the codebase wins: record what IS, mark the divergence `(legacy)` in Style. Rules the code doesn't follow just teach /build to drift.
- Multiple languages in the workspace → list all in Stack, ask which is primary (1 MCQ).
- Monorepo / multi-repo → one PROJECT.md + CRAFT.md at the workspace root; Structure lists each repo path + purpose.
- User says "skip questions" → recommended defaults, marked `(assumed)` in CRAFT.md — but the step-6 approval gate still runs.
