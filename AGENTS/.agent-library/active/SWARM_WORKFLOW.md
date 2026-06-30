# Swarm Workflow — Multi-Agent Execution Protocol

## Trigger
Use this workflow for any request that benefits from parallel specialist work, structured deliverables, multi-stage review, or production-grade output.

## Phases

### 1. Intake
- Understand the request.
- Identify project type and constraints.
- Infer missing but useful requirements from context.
- Define success criteria before decomposition.

### 2. Task Breakdown
- Create kanban cards via `kanban create`.
- Assign specialists based on task type:
  - Creative vision, branding, layout, premium feel → `soren-creative-director`
  - Architecture, components, routing, state, folder structure → `soren-frontend-architect`
  - Animation choreography, scroll behavior, microinteractions → `soren-motion-systems`
  - 3D scenes, shaders, particles, WebGL logic → `soren-webgl-3d`
  - Code implementation, file creation, clean production code → `soren-code-builder`
  - QA, visuals, responsiveness, performance, accessibility → `soren-qa-reviewer`
  - Strong prompts for coding agents → `soren-prompt-engineer`
  - Local setup, model config, CLI tools, Docker, endpoints → `soren-devops-tools`
  - Docs, setup guides, command references → `soren-documentation`
- Mark dependencies. Group parallel-safe tasks.
- Run coordinating agents first when they unblock other work.

### 3. Specialist Execution
- Claim cards: specialist updates status to `In Progress` or `Parallel Running`.
- Agents output role-scoped, copy-paste-ready deliverables.
- Report exact files affected and decisions made.

### 4. Review
- QA reviews concrete outputs and the combined result.
- Orchestrator resolves conflicts and merges.
- Weak outputs return to the responsible agent with explicit issues.
- Blocked tasks explain exactly what is missing.

### 5. Final Synthesis
- Orchestrator produces one unified final answer.
- Include: files changed, commands run, decisions made, blockers, and next steps.

## Parallel Rules
Safe to parallelize:
- Creative and architecture planning
- Motion spec and content structure
- Independent component builds
- Documentation and QA checklist drafting
- Research/planning that does not touch the same files

Unsafe to parallelize:
- Editing the same file concurrently
- Architecture changes during implementation
- Refactoring while another agent builds the same component boundary
- Running destructive commands without approval

File-lock pattern:
- Before editing files, update the card status to `In Progress` or `Parallel Running`.
- After finishing, report changed files in the card.

## Kanban Columns
- `Inbox` — captured but not assessed
- `Backlog` — assessed, not ready
- `Ready` — ready to assign
- `In Progress` — single-owner active work
- `Parallel Running` — parallel-safe active work
- `Blocked` — blocked with explicit reason and unblock condition
- `Review` — awaiting QA/orchestrator approval
- `Done` — accepted
- `Archived` — retained for reference

## Task Card Content
Every card must include:
- Task ID
- Title
- Description
- Assigned agent
- Status
- Priority
- Dependencies
- Files affected
- Acceptance criteria
- Result summary
- Review status

## Escalation Rules
- If requirements are genuinely unclear, escalate to the user before blocking.
- If an agent fails twice, block the task and report the exact failure mode.
- If there are merge conflicts between parallel agents, orchestrator resolves before asking the user.

## Output Format
Final synthesis should follow:
- What changed
- Files affected
- Commands run
- Decisions made
- Blockers or risks
- Next steps
