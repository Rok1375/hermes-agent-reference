---
name: codex-goal-prompt-authoring
description: Explain and write effective instructions for persistent self-checking agent loops (Codex /goal, or analogous long-running autonomous task loops with auto-continue). Use when the user mentions a long-running autonomous agent run, a goal loop, wants to kick off an overnight job, or asks how to write a goal prompt.
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [codex, /goal, autonomous, long-running, agent-loop, prompt-engineering]
---

# Codex /goal — Prompt Authoring

A persistent self-checking agent loop is a long-running autonomous task that auto-continues across turns until a stop condition is met, the user pauses, or the token budget runs out. Codex's `/goal` (CLI v0.128.0+) is the canonical example; the 4-part contract generalizes to any equivalent loop.

**Key difference from a normal prompt:** when a turn ends but the goal isn't met, the agent auto-continues instead of waiting for input.

**Lifecycle states (typical):** `pursuing`, `paused`, `achieved`, `unmet`, `budget-limited`.

**Not:** a budget command, a safety boundary, "run forever", or a replacement for `/plan`. It's a contract enforcer with a verification loop.

## When to use it

Use only when **all three** are true:
1. Task is >30 min of mechanical work.
2. There's a **verifiable stop condition** (tests pass, coverage hit, eval ≥ X, build green).
3. Repo is agent-ready (working build, decent tests, `AGENTS.md` present).

**Fits:** migrations, coverage lifts, TDD feature builds, refactors with contract tests, prompt/eval optimization, deploy retry loops, bug-repro-then-fix.

**Bad fits:** exploratory work, vague "improve this", anything without a "done" definition, prod credentials, destructive shared-infra ops.

## The 4-part contract (every goal needs this)

1. **Objective** — one sentence, one concrete outcome.
2. **Constraints** — what must NOT change (public API, files, libs, conventions).
3. **Validation command** — the exact shell command that proves progress (`pytest -q`, `pnpm test`, etc.).
4. **Stop condition** — verifiable: "Stop when X passes" OR "when further changes need human/product input."

Plus: tell the agent what to read first, ask it to work in checkpoints with a short progress log.

## The one-paragraph goal (the core deliverable)

Pack the 4-part contract into a single dense paragraph. Template:

> `/goal <one-sentence objective>. Read first: <files/PLAN.md/issue>. Constraints: <what not to change, libs, conventions>. Validate after each change with: <exact command>. Work in checkpoints and log progress briefly. Stop when: <verifiable condition>, OR when further changes require human/product input.`

### Example (migration)

> `/goal Migrate this project from Pydantic v1 to v2. Read first: pyproject.toml, src/, tests/. Constraints: no public API changes; keep imports backwards-compatible via shims if needed; no new dependencies. Validate after each change with: pytest -q. Work in checkpoints; log progress briefly. Stop when: full suite passes with zero deprecation warnings, OR when a change requires architecture decisions.`

### Example (coverage lift)

> `/goal Raise coverage in src/auth/ from ~38% to ≥75%. Read first: src/auth/, tests/auth/, AGENTS.md. Constraints: no new deps; mirror existing test style; do not modify production code unless strictly required for testability. Validate with: pytest --cov=src/auth --cov-report=term-missing. Work in checkpoints; log coverage delta each one. Stop when: coverage ≥75% AND all tests pass, OR when uncovered code needs design changes.`

## Writing rules

- **One objective, one stop condition.** Not a backlog.
- Use **literal strings** for paths, commands, issue numbers — exact.
- Forbid scope creep explicitly: "Do not refactor unrelated code. Do not add dependencies."
- Tell the agent when to pause: "If <condition>, pause and ask before proceeding."
- Short, vague goals burn tokens for no extra value vs. a normal prompt.

## Meta-prompting trick (highest-leverage)

Hand-written goals under-specify. Ask a second AI session (a fresh agent with the codebase loaded, or a separate thread in the same dir) to:
1. Inspect the codebase,
2. Surface hidden assumptions / constraints / edge cases,
3. Emit a dense `/goal` paragraph using the 4-part contract.

Order-of-magnitude better runs.

## Operational tips

- Always launch on a branch or worktree. Never on `main`.
- Inspect status periodically with bare `/goal`.
- **Always review the diff** before merging — long autonomy means more code to validate, not less.
- Keep approvals / sandboxing tight; default permissions are correct.
- First run: pick a 30-min scoped task so you learn how the loop actually stops before trusting it overnight.

## When a goal drifts

- **Minor drift:** type a correction in the composer (auto-pauses, folds it in, resumes).
- **Loose objective:** pause, read status, then `<tighter version>` — replaces the contract. Don't pile instructions on a vague goal.
- **Bad mess:** clear, `git reset --hard HEAD` or `git stash`, rewrite with the meta-prompting trick, restart.

Don't let a drifting goal keep running "to see where it goes." Tokens burn, diffs compound.

## Mental model

A persistent self-checking loop is a **contract enforcer with a verification loop**, not a "run forever" button. The shift: stop writing prompts, start writing **specifications with stop conditions**. Spend the time upfront defining "done"; the run takes care of itself.
