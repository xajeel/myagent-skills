<p align="center">
  <img src="https://raw.githubusercontent.com/xajeel/AI-SDLC/main/assets/ai-sdlc-kit.png" alt="ai-sdlc-kit" width="320">
</p>

<h1 align="center">ai-sdlc-kit</h1>
<p align="center">Spec-driven SDLC skills that make AI coding agents work like a real engineering team — and leave the developer understanding every change.</p>

<p align="center">
  <a href="https://pypi.org/project/ai-sdlc-kit/"><img src="https://img.shields.io/pypi/v/ai-sdlc-kit.svg" alt="PyPI version"></a>
  <a href="https://www.npmjs.com/package/ai-sdlc-kit"><img src="https://img.shields.io/npm/v/ai-sdlc-kit.svg" alt="npm version"></a>
  <a href="https://github.com/xajeel/AI-SDLC/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
</p>

---

## What is it?

`ai-sdlc-kit` installs a set of skills for Claude Code, Cursor, Windsurf, and Gemini CLI that turn one coding agent into a small engineering team with separated roles:

**diagnose & teach → spec in verifiable tasks → write the tests first → build to the tests → independent QA gate → ship**

Two ideas drive the design:

1. **The developer must understand everything.** `/plan` walks you through the problem in plain language — *"when the user does X, this function runs, it should return Y but returns Z, and here's the line that causes it"* — plus the solution options and why one wins. Every feature keeps a human-readable `walkthrough.md` next to its machine-readable `spec.md`. You finish able to trace the code and defend the decisions.
2. **No agent grades its own homework.** Each stage runs as its own fresh agent: the tester writes tests from the spec before the code exists, the builder can't touch those tests, and QA never sees the builder's reasoning. The final diff is written to survive review by an outside frontier model.

| Skill | Role | Does |
|---|---|---|
| `sdlc-init` | — | One-time setup: project facts, git workflow, agent tiers, the code-craft rulebook, rule stubs for every coding tool |
| `roadmap` | Planner | Splits a PRD into ordered, shippable features |
| `plan` | Planner (strong) | Diagnoses a ticket from the real code, teaches you the flow, proposes production-grade solutions with pros/cons — never writes code |
| `spec` | Planner (strong) | Turns the agreed plan into machine-facing tasks with contracts + verify commands |
| `test` | Tester (strong) | Writes the tests from the spec BEFORE implementation; for a bug, a failing reproduction test first |
| `build` | Developer (mid) | Implements task-by-task until the tester's tests go green — it may never edit them |
| `qa` | QA (strong) | Independent gate: verifies, regression-checks, scans the diff for PR-readiness |
| `ship` | — | Branch guard, staged commit per your git workflow; `--pr` opens a self-explanatory pull request |
| `automate` | Orchestrator | The whole pipeline hands-free — adopts any plan you already agreed on |
| `architecture-diagram` | — | Renders a self-contained HTML/SVG architecture diagram |
| `mentor` | — | Teaches you a topic — or a whole codebase — one tracked lesson at a time |

## Install

```bash
# Python
pip install ai-sdlc-kit        # or: uv tool install ai-sdlc-kit

# Node
npm install -g ai-sdlc-kit     # or one-off: npx ai-sdlc-kit install --agent claude
```

Both give you the same `ai-sdlc` command and the same skills — pick whichever ecosystem you already have.

## Use

```bash
ai-sdlc install --agent claude   # or cursor / windsurf / gemini / all
```

This copies the skills into your agent's skills directory. They then appear as slash commands: `/sdlc-init`, `/roadmap`, `/plan`, `/spec`, `/test`, `/build`, `/qa`, `/ship`, `/automate`, `/mentor`.

| Flag | Effect |
|---|---|
| `--agent all` | install for every supported agent at once |
| `--target PATH` | install into a specific project directory (default: `.`) |
| `--global` | install into your home directory instead of a project |
| `--force` | overwrite existing skill folders |

```bash
ai-sdlc list         # see bundled skills
ai-sdlc --version    # print the installed version
```

> **Monorepo, or no Python project at the root?** Not a problem — `ai-sdlc install` only copies skill files into `.claude/skills/` (or your agent's folder); it never reads `pyproject.toml` or anything else at the target. Install the CLI once with `pipx install ai-sdlc-kit` or `uv tool install ai-sdlc-kit`, then run it at your repo root — or from anywhere with `--target /path/to/root`.

## Getting started

Every project starts with `/sdlc-init` — run it once, in your agent, inside your project folder. It writes:

- `.sdlc/PROJECT.md` — project facts, real commands, **your git workflow** (it asks how you branch/commit; answer or take the default), and the **agent tier map** (which stages run on a strong vs mid model)
- `.sdlc/CRAFT.md` — the code rulebook: pinned stack versions with modern-idiom rules, folder structure, style (including the always-on simplicity rules: minimal comments, no essay docstrings, boring beats clever), config and security rules
- `.sdlc/STATE.md` — the feature tracker
- `AGENTS.md` at the repo root — a short pointer so **any** coding tool (Claude Code, Cursor, Codex, Windsurf…) lands on the same rules before writing code

It works two ways: give it a PRD (`/sdlc-init <path>`) for a new project, or run it bare on an existing codebase and it detects your stack from lockfiles and a handful of source files. Either way it shows you the rules for approval before writing anything.

## The pipeline

Every feature or bug lives in `.sdlc/specs/<name>/` as exactly two files:

- **`walkthrough.md`** — for humans: the problem, the traced flow, the chosen solution, and every decision with its "why", in plain language.
- **`spec.md`** — for machines: tasks, file whitelists, contracts, verify commands. No prose.

The stages, each in its own fresh agent:

1. **`/plan <ticket>`** — the conversation. It reads the real code and explains: what the ticket means, what actually happens (with `file:line` traces), why, and the production-grade ways to fix or build it — with pros/cons when options genuinely compete. You push back and refine; when you agree, it writes `walkthrough.md`. It never writes code.
2. **`/spec <feature>`** — turns the agreed direction into ≤ 8 small, independently verifiable tasks.
3. **`/test <feature>`** — a separate tester writes the test suite from the spec's contracts, before any implementation exists. For a bug, the first test *reproduces* it and fails — proof the cause was understood. The developer model never writes the tests that will judge its code.
4. **`/build <feature>`** — implements task-by-task until the tester's tests flip red → green. It may never edit those tests; if one looks wrong it files a challenge and `/test` adjudicates against the spec.
5. **`/qa <feature>`** — a fresh agent re-runs everything, checks for regressions, and scans the diff for PR-readiness: no debug leftovers, no dead code, no hardcoded config, code a mid-level dev can trace. Failures become fix-tasks `/build` executes.
6. **`/ship <feature>`** — commits per your git workflow; `/ship <feature> --pr` pushes and opens a PR whose description (why / what changed / how to test) stands alone for any outside reviewer — human or another company's frontier model.

Pick the path that matches what you're doing:

| Situation | Commands |
|---|---|
| A bug ticket | `/plan <ticket>` → it traces and explains the bug, you agree on the fix → `/automate <name>` (or step through `/spec` → `/test` → `/build` → `/qa`) |
| One feature | `/plan <feature>` → agree on the design → `/automate <name>` or the stages one by one |
| A whole project from a PRD | `/sdlc-init <PRD>` → `/roadmap <PRD>` → run the pipeline per feature, in order |
| "Just do it, don't ask me" | `/automate <ticket>` with no prior plan — it makes every call itself and logs each one `(auto)` |
| Learning a codebase or a technology | `/mentor` — see [Learning a codebase](#learning-a-codebase) below |

## Hands-free mode

`/automate <x>` runs the whole pipeline back-to-back with **zero stops** except real blockers (a verify failing after 3 attempts, an unfixable regression). Its one branch point is at the start:

- **You already planned** — a `walkthrough.md` you agreed in `/plan` exists → automate adopts *your* plan and continues from whatever stage is next. It never re-litigates an agreed decision.
- **Nothing exists** — it plans itself, always taking the recommended production-standard option, and records every choice in the walkthrough marked `(auto)` so you can audit the whole run afterwards.

Every stage still runs as its own fresh agent on its configured tier — automation never collapses the roles back into one context.

Other controls:

- **Skip a stage** — `/sdlc-init --skip-ship` (or `--skip-qa`, `--skip-test`) drops a stage from handoffs and `/automate`; toggle any time with `--unskip-<stage>`. Invoking a skipped skill directly still runs it.
- **Bug memory** — `/qa` keeps `.sdlc/BUGS.md`, a plain-English log: what went wrong, why, how to avoid it. `/plan`, `/spec`, and `/build` read it on every run so the same bug never ships twice.
- **Agent tiers** — the `Agents:` line in PROJECT.md maps each stage to `strong` or `mid`. Tiers, not vendor names: map them to whatever models your tool offers. Thinking stages (plan, spec, test, qa) default to strong; implementation runs on mid.

## Learning a codebase

The other skills build software; `/mentor` teaches it. Point it at a topic ("teach me Kafka") or at the repo you're sitting in ("teach me this codebase") and it builds a curriculum, then teaches **one lesson per session** — never dumping the whole course at once — quizzing you at the end of each and only advancing once you pass.

For a codebase it reads the repo first, then runs a short course of one or two modules:

- **Map & Run** — what the project does, a guided tour of the directories, getting it running locally, and one core flow traced end-to-end through the real files.
- **Work On It** — the subsystems that change most often (picked from `git log`), the repo's conventions, and how a change actually ships here: branch, tests, CI, PR.

Every lesson cites real `file:line` locations and the exercises happen inside the repo — run it, trace it, write a failing test. The final project is a real change with tests passing, so you finish able to contribute rather than just able to describe the code.

State lives in `curriculum/<topic>/` (tracker, lessons, quizzes, projects), so progress survives across days and machines. Say `next` to continue, `quiz me` for a cumulative check, or `status` to see where you are.

## Changelog

- [x] `0.2.0` — pipeline redesign: new `/plan` (diagnose & teach a ticket in plain language, no code) and `/test` (independent tester writes the tests from the spec before build) skills; specs split into human `walkthrough.md` + machine `spec.md` under `.sdlc/specs/<feature>/`; every stage runs as its own fresh agent with configurable model tiers; `/automate` adopts an agreed plan and runs with zero stops; git-workflow rules and `AGENTS.md` stubs from `/sdlc-init`; `/ship --pr` opens a self-explanatory pull request
- [x] `0.1.8` — PyPI and npm realigned on one version number, so `pip` and `npm` always ship the same kit
- [x] `0.1.7` — new `/mentor` skill: a long-term, file-tracked teacher for any topic, with a codebase mode that onboards you to a repo well enough to contribute to it
- [x] `0.1.6` — same kit on npm: `npm install -g ai-sdlc-kit` (zero-dependency Node CLI); releases now publish to PyPI and npm together
- [x] `0.1.5` — `/automate` hands-free flow, stage skipping (`--skip-qa` / `--skip-ship`), plain-English bug log (`.sdlc/BUGS.md`) that feeds future specs, and specs now open with a mental-model section (what / why / how) presented in chat
- [x] `0.1.4` — code-craft rulebook: `.sdlc/CRAFT.md` pins stack versions + modern idioms, enforces folder structure (one concern per file), env-based config with `.env.example`, and a security baseline across `/spec`, `/build`, `/qa`
- [x] `0.1.3` — added a Getting started section: how to actually invoke the skills for a new project, an existing codebase, or a single feature
- [x] `0.1.2` — automated PyPI releases via GitHub Actions trusted publishing
- [x] `0.1.1` — cleaner README, professional package presentation
- [x] `0.1.0` — initial release: 6 core skills + architecture-diagram, pip-installable CLI

---

Related prior art: [github/spec-kit](https://github.com/github/spec-kit).
