# Kanban Workflow — Multi-Agent Task Board Protocol

## Board Columns
- `Inbox`
- `Backlog`
- `Ready`
- `In Progress`
- `Parallel Running`
- `Blocked`
- `Review`
- `Done`
- `Archived`

## Task Properties
- `task_id` — unique identifier
- `title` — short human title
- `description` — concrete deliverable description
- `assigned_agent` — profile name, e.g. `soren-code-builder`
- `status` — one of the columns above
- `priority` — `critical`, `high`, `medium`, `low`
- `dependencies` — list of task IDs that must complete first
- `files_affected` — absolute or project-relative paths
- `acceptance_criteria` — checkable conditions
- `result_summary` — what was delivered
- `review_status` — `pending`, `approved`, `needs_revision`

## Workflow Rules

### Creating Cards
1. Orchestrator (`soren-orchestrator`) creates cards for every meaningful subtask.
2. Title is actionable: "Build hero section", not "Design".
3. Description states concrete output: "Deliver a TypeScript React component implementing XYZ with ABC behavior."
4. Acceptance criteria are checkable: exact behaviors, file paths, or acceptance gates.

### Assigning Cards
1. Use `kanban assign` to assign the card to the correct profile.
2. Coincident or parallel-safe cards are marked `Parallel Running`; otherwise use `In Progress`.
3. Cards must not leave `Backlog` without an assignee and files_affected.

### Updating Status
1. Worker moves its assigned card to `In Progress` or `Parallel Running` when starting.
2. Worker sets `files_affected` before making changes.
3. Worker moves to `Review` on completion with `result_summary` filled.
4. QA moves to `Done` on approval, or back to `In Progress` with a punch list on rejection.

### Blocked Cards
1. Mark as `Blocked` and add a comment explaining exactly what is missing.
2. Blocked cards must include an unblock condition: "Waiting for API key X" or "Waiting for task Y".
3. Orchestrator reviews `Blocked` cards on each turn.

### Archived Cards
1. Move tasks that are no longer relevant to `Archived`.
2. Never delete cards unless explicitly requested.

## Agent Execution Contract
1. Always claim the task before editing files.
2. On completion, update the card with:
   - `result_summary`
   - `files_affected`
   - `review_status`
3. On block or failure, update the card with exact reason and proposed remediation.

## Parallel Safety
- Tasks touching the same file must be sequential.
- Dependency map must be explicit.
- When in doubt, prefer sequential over parallel.
