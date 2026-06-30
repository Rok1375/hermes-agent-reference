---
name: hermes-subagent-orchestrator
description: "Use when a task is complex and benefits from splitting into focused subagents: multi-step work, large coding tasks, debugging, repo analysis, design/motion systems, or research-heavy workflows."
version: 1.0.0
author: soren
license: MIT
metadata:
  hermes:
    tags: [subagents, delegation, orchestration, research, review, architecture, implementation]
    related_skills: [plan, systematic-debugging, github-code-review, codebase-inspection, self-improving-subagent-router]
---

# Hermes Subagent Orchestrator

## Overview

For complex tasks, split work into focused subagents and synthesize the final answer. This avoids context-window overload, enables parallel workstreams, and produces higher-quality results with cleaner separation of concerns.

## When to Use

- Large coding tasks (3+ files, cross-cutting changes)
- Debugging sessions (root cause, fix plan, implementation)
- Repo analysis (architecture, security, code quality)
- Design systems / motion systems (UI, animation, 3D)
- Research-heavy tasks (discovery → synthesis)
- Multi-step workflows where each step is a distinct discipline
- Code review (architecture + bugs + quality + fix plan)

**Do NOT use for:**
- Simple questions or quick answers — answer directly
- Tasks with only 1-2 files and no cross-cutting concerns
- Mechanical single-step operations (move a file, rename a variable)

## Subagent Roles

### A. Research Subagent
- **Mission:** Find current facts, docs, APIs, errors, package versions, and relevant references.
- **Input:** Question or topic with search/browse context.
- **Output:** Concise findings with source/path references.
- **Boundaries:** Does not modify files. Does not design solutions. Returns raw findings only.

### B. Architecture Subagent
- **Mission:** Design the system/component structure.
- **Input:** Requirements + research findings.
- **Output:** File structure, data flow, state management, routing, integration points, recommended patterns.
- **Boundaries:** Does not modify files unless explicitly allowed. Produces a plan, not code.

### C. Implementation Subagent
- **Mission:** Write or edit code after the architecture plan is clear.
- **Input:** Architecture plan + requirements.
- **Output:** Changed files with the reason for each change.
- **Boundaries:** Keeps changes scoped. Avoids unrelated refactors. Does not redesign the architecture.

### D. Review Subagent
- **Mission:** Review code for bugs, missing imports, type errors, security issues, performance issues, accessibility issues, and edge cases.
- **Input:** Code files or diff to review.
- **Output:** Pass/fail findings per category with specific fix recommendations.
- **Boundaries:** Does not modify files unless explicitly requested.

### E. Motion/Design Subagent
- **Mission:** Handle UI, animation, 3D, WebGL, GSAP, Framer Motion, Tailwind, React, Next.js, Vite, Three.js, or Spline tasks.
- **Input:** Plain-language animation/design idea + tech stack context.
- **Output:** Professional motion terms, timing/easing definitions, stagger schedules, scroll behavior, responsive rules, reduced-motion behavior, and acceptance criteria.
- **Boundaries:** Translates creative intent into technical specs. Does not implement unless assigned.

## Orchestration Rules

1. **Use subagents only when the task genuinely benefits.** For small tasks, answer directly.

2. **Never spawn vague subagents.** Every subagent must have a clear mission, input, output format, and boundaries.

3. **Prefer parallel work** for independent research/review tasks (e.g., Research + Review can run simultaneously).

4. **Prefer sequential work** for architecture → implementation → review (each depends on the previous).

5. **Always synthesize** subagent results into one final answer. Integrate findings, resolve conflicts, present a coherent result.

6. **Keep final answers concise but complete.** Do not expose raw logs, stack traces, or internal event dumps unless asked.

7. **If a subagent fails**, continue with the remaining subagents and report the failure clearly in the synthesis.

8. **Never erase or rewrite unrelated user files.** Scope changes to what the task requires.

9. **Always prioritize safety, correctness, and reversible changes.**

10. **Subagents should not make file changes unless explicitly assigned implementation work.**

11. **Prefer sequential for architecture → implementation → review** — never implement before architecture is reviewed.

## Recommended Workflows

### Workflow: Full Feature (New)
```
Research → Architecture → [Architecture Review] → Implementation → Review → Synthesize
```
Run Architecture Review as a subagent between Architecture and Implementation for complex features.

### Workflow: Bug Fix
```
Research (find root cause) → Implementation (fix) → Review → Synthesize
```

### Workflow: Code Review
```
Architecture Inspector → Bug Checker → UI/Motion Quality Check → Fix Plan → Synthesize
```
Run all inspectors in parallel, then create the fix plan from their findings.

### Workflow: Design / Motion
```
Research (inspiration/patterns) → Motion/Design (spec) → Implementation → Review → Synthesize
```

## Preferred Technology Defaults

When applicable, use these defaults:

### Tech Stack
- React / Next.js / Vite
- Tailwind CSS
- GSAP / ScrollTrigger
- Framer Motion
- Three.js / React Three Fiber
- Spline (when useful)

### Design Direction
- Premium, dark, futuristic, cinematic, luxury
- High-end typography
- Glass panels (frosted backdrop)
- 3D depth
- Smooth motion
- Instant wow impact

### Coding Output Style
- Production-grade
- Copy-paste-ready
- File-by-file structure
- Exact implementation steps
- Responsive rules
- Performance requirements
- Accessibility requirements
- What not to do (common pitfalls)
- Acceptance criteria

## Common Pitfalls

1. **Spawning a subagent without clear output format.** Always specify: "Return a list of findings with file paths" not "investigate the codebase."
2. **Forgetting to synthesize.** Subagent outputs are intermediate — always combine them into one final answer.
3. **Parallel when sequential is needed.** Do not run Implementation and Review in parallel — Review must see Implementation output.
4. **Running too many subagents.** 3-5 is the sweet spot. More than 6 usually fragments the task.
5. **Subagent scope creep.** Architecture subagent starts writing code, Review subagent starts refactoring. Define boundaries upfront.
6. **Ignoring kanban.** For multi-agent flows, write cards before work. Work must not start without a claimed card.
7. **Parallel file collisions.** Two agents editing the same file or boundary-block in parallel breaks the result. Maintain a dependency map and only run parallel when safe.

## Verification Checklist

- [ ] Each subagent has a clear mission, input, output format, and boundaries
- [ ] Parallel work used for independent tasks; sequential for dependent chains
- [ ] Final answer synthesizes all subagent outputs
- [ ] No raw logs or internal dumps in the final answer
- [ ] All modified files are listed with reasons
- [ ] Failures are reported clearly
- [ ] Unrelated files are untouched
