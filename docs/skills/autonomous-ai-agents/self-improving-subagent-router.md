---
name: self-improving-subagent-router
description: "Use before any task to classify, route to subagents, and after completion to reflect and improve skills. The self-improvement loop for Hermes subagent orchestration."
version: 1.0.0
author: soren
license: MIT
metadata:
  hermes:
    tags: [router, self-improvement, subagents, orchestration, reflection, skill-management]
    related_skills:
      - skill-router
      - hermes-subagent-orchestrator
      - self-improvement-memory
---

# Self-Improving Subagent Router

## Overview

This skill combines three layers:
1. **Pre-task classification** — classify the task and decide the best route
2. **During-task orchestration** — dispatch focused subagents only when useful
3. **Post-task reflection** — improve skills, memory, and routing for next time

It references three sibling skills rather than duplicating them:
- `skill-router` — automatic skill dispatch (loaded before every task)
- `hermes-subagent-orchestrator` — subagent roles, workflows, orchestration rules
- `self-improvement-memory` — post-task reflection and durable memory

## When to Use

- **Before every non-trivial task** — classify and route
- **After every non-trivial task** — reflect and improve
- When the user says "Route this task" or "use subagents"
- When the user says "improve your skills" or "audit your router"
- When the user provides a complex multi-step request

**Do NOT use for:**
- Trivial yes/no questions
- One-line lookups (time, weather, simple facts)
- Purely conversational messages

## Procedure

### Phase 1: Before Task — Classify & Route

When a task arrives:

1. **Classify the task type** into one of:
   - `direct` — simple answer, no subagents needed
   - `research` — fact-finding, docs, API discovery
   - `architecture` — system design, component structure
   - `implementation` — code writing, file edits
   - `review` — code review, bug hunting, quality check
   - `motion-design` — UI, animation, 3D, WebGL
   - `multi-agent` — complex task needing 2+ subagent types
   - `improvement` — skill audit, router improvement, reflection

2. **Decide if subagents are useful** using these criteria:
   - Does the task span multiple disciplines? → multi-agent
   - Does the task require both discovery AND implementation? → multi-agent
   - Is the task >3 files or cross-cutting? → multi-agent
   - Is the answer obvious with <3 tool calls? → direct
   - Otherwise → use the matching single-agent route

3. **Route to the appropriate handler:**
   - `direct` → answer immediately. No subagents, no reflection needed.
   - `research` → load `hermes-subagent-orchestrator`, use Research subagent
   - `architecture` → load `hermes-subagent-orchestrator`, use Architecture subagent
   - `implementation` → proceed with normal tools (terminal, write_file, patch)
   - `review` → load `hermes-subagent-orchestrator`, use Review subagent
   - `motion-design` → load `hermes-subagent-orchestrator`, use Motion/Design subagent
   - `multi-agent` → load `hermes-subagent-orchestrator`, dispatch parallel/sequential subagents
   - `improvement` → enter Phase 3 immediately (this is a meta-task)

### Phase 2: During Task — Execute with Focus

Follow the matching subagent's mission, input, output format, and boundaries (defined in `hermes-subagent-orchestrator`).

**Keep rules:**
- Each subagent gets a clear mission, input, output format, and boundaries
- Prefer parallel work for independent sub-tasks (research + review)
- Prefer sequential work for dependent chains (architecture → implementation → review)
- Never implement before architecture is reviewed
- Require structured output from every subagent
- Synthesize all subagent results into one coherent final answer
- If a subagent fails, continue with the rest and clearly report the failure

### Phase 3: After Task — Reflect & Improve

Run this for every task that took 3+ tool calls OR involved subagents OR revealed something new.

**Ask four questions:**

| # | Question | If yes → |
|---|----------|----------|
| A | Did this task reveal a reusable pattern? | Save to memory as a compact fact. If it's a 3+ step procedure, create a new skill (ask user first). |
| B | Did Hermes miss a tool, checklist, or instruction? | Patch the relevant existing skill. Load it with `skill_view`, identify the gap, use `skill_manage(action='patch')`. |
| C | Would a future version of Hermes do better with a new skill? | Propose the new skill to the user with a specific name, trigger description, and what it would contain. Create only after user confirms. |
| D | Should the routing rules be updated? | Update this skill's classification or routing table with `skill_manage(action='patch')`. Log the change. |

**Rules for improvements:**
- Do not create new skills for trivial tasks (<5 tool calls, no reusable pattern)
- Do not store sensitive personal information unless explicitly asked
- Do not overwrite existing skills without backup (config backup exists at `config.backup-*.yaml`)
- Prefer improving existing skills over creating duplicates
- Every improvement must be small, specific, and reusable
- Every improvement must include a reason and a test prompt
- Log every improvement to `hermes-skill-improvement-log.md`

### Phase 4: Log the Improvement

Append to `C:\Users\soren\AppData\Local\hermes\hermes-skill-improvement-log.md` using this format:

```
## YYYY-MM-DD HH:MM — <Task Type>

**Issue discovered:** <what was missing, wrong, or improvable>

**Improvement made:** <exact change>

**Files changed:**
- `<path>` — <reason>

**Test prompt:**
> <one-liner to trigger this improvement next time>

**Rollback note:** <how to undo this if needed>

**Reflection:** <1-2 sentences on what worked, what didn't>
```

## Reusable Command Prompts

The following prompt patterns are stored in the companion file `references/subagent-router-commands.md`. When the user matches one of these, load and execute:

| User says | Action |
|-----------|--------|
| "Route this task through the best Hermes subagents" | Classify → route to multi-agent orchestration |
| "Use research, architecture, implementation, and review subagents" | Full feature workflow via `hermes-subagent-orchestrator` |
| "Use motion/design subagent for this UI" | Route to Motion/Design subagent only |
| "After completing this task, improve your skill router if useful" | After task, enter Phase 3 reflect-and-improve |
| "Audit your current subagent skills and suggest improvements" | Enter Phase 3 immediately, evaluate all subagent-related skills |
| "Route this task" | Same as "Route this task through the best Hermes subagents" |

## Example: Full Session Flow

```
User: "Build a premium landing page with animations"

Phase 1: Classify → multi-agent (architecture + implementation + motion-design)
Phase 2: Dispatch Architecture → Motion/Design → Implementation → Review → Synthesize
         (via hermes-subagent-orchestrator)
Phase 3: Reflect → "The scroll-trigger timing was tricky, but the GSAP skill handled it.
          Should I save this pattern?" → Log improvement
Phase 4: Append to hermes-skill-improvement-log.md
```

## Common Pitfalls

1. **Skipping Phase 3 after a complex task.** The reflection loop is what makes this self-improving. Set a reminder if needed.
2. **Creating a new skill when an existing one should be patched.** Always check `skills_list` and `skill_view` first.
3. **Logging too much detail.** Keep improvement log entries focused on what changed and why, not every step of the task.
4. **Router bypass.** Do not skip Phase 1 classification. Even when the task seems obvious, the 2-second classification ensures consistency.
5. **Over-improvement.** If a skill was used correctly and nothing went wrong, do not invent improvements. "No improvement needed" is a valid outcome.
6. **Forgetting to reference existing skills.** Before creating a new skill or rule, check if `hermes-subagent-orchestrator`, `self-improvement-memory`, or `skill-router` already covers it.

## Verification Checklist

- [ ] Task was classified before execution
- [ ] Subagents were used only when beneficial (or direct answer chosen appropriately)
- [ ] For complex tasks: Phase 3 reflection was completed
- [ ] For applicable improvements: logged to `hermes-skill-improvement-log.md`
- [ ] Existing skills were checked before creating new ones
- [ ] No new skills were created without user confirmation
- [ ] Backups exist before any skill was modified
