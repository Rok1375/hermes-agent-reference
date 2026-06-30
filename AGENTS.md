# Hermes Agent - AGENTS.md & Protocol Reference

> Reference for the **20 instruction files** that shape how Hermes Agent boots, routes work, and behaves across the local `.agent-library/` and `.agents/` directories.

**Source locations covered:**

- `C:/Users/soren/.agent-library/active/` - 9 global protocol files loaded at agent boot
- `C:/Users/soren/.agent-library/agents/` - 4 agent-specific AGENTS.md (one per agent variant)
- `C:/Users/soren/.agents/skills/` - 5 third-party skill packages with AGENTS.md (plus 1 nested `loki-mode/references/AGENTS.md`)
- `C:/Users/soren/AGENTS.md` - 1 home-level instruction file
-
**Total:** 4 + 9 + 5 + 1 + 1 = **20 documents**

---

## Table of contents

### Active global protocol files (.agent-library/active/)

- [`.agent-library/active/FABLE_MODE.md`](./.agent-library/active/FABLE_MODE.md) - NOT a model identity. Workflow discipline only.
- [`.agent-library/active/GLOBAL_AGENT_BOOT.md`](./.agent-library/active/GLOBAL_AGENT_BOOT.md) - This is the universal preflight boot file. All agents must read and follow this before starting work.
- [`.agent-library/active/GLOBAL_SWARM_SETUP.md`](./.agent-library/active/GLOBAL_SWARM_SETUP.md) - You are Hermes Desktop acting as a local AI systems architect, agent orchestrator, and automation engineer. Your task is
- [`.agent-library/active/HANDOFF_PROTOCOL.md`](./.agent-library/active/HANDOFF_PROTOCOL.md) - Required output:
- [`.agent-library/active/KANBAN_WORKFLOW.md`](./.agent-library/active/KANBAN_WORKFLOW.md) - - `Inbox`
- [`.agent-library/active/SKILL_ROUTER.md`](./.agent-library/active/SKILL_ROUTER.md) - Routes by task type:
- [`.agent-library/active/SWARM_WORKFLOW.md`](./.agent-library/active/SWARM_WORKFLOW.md) - Use this workflow for any request that benefits from parallel specialist work, structured deliverables, multi-stage revi
- [`.agent-library/active/TASK_PREFLIGHT.md`](./.agent-library/active/TASK_PREFLIGHT.md) - Before task:
- [`.agent-library/active/VERIFICATION_PROTOCOL.md`](./.agent-library/active/VERIFICATION_PROTOCOL.md) - - Tests pass

### Agent-specific instructions (.agent-library/agents/)

- [`.agent-library/agents/generic/AGENTS.md`](./.agent-library/agents/generic/AGENTS.md) - Before task: read GLOBAL_AGENT_BOOT.md, nearest AGENTS.md, run skill router.
- [`.agent-library/agents/hermes/AGENTS.md`](./.agent-library/agents/hermes/AGENTS.md) - Before any task: read GLOBAL_AGENT_BOOT.md, run skill router, check Fable mode.
- [`.agent-library/agents/kilo/AGENTS.md`](./.agent-library/agents/kilo/AGENTS.md) - Default role: reviewer/verifier (unless assigned implementation).
- [`.agent-library/agents/opencode/AGENTS.md`](./.agent-library/agents/opencode/AGENTS.md) - Before any task: read project context, apply relevant skills, check Fable mode.

### External skill packages (.agents/skills/)

- [`.agents/skills/dbos-golang/AGENTS.md`](./.agents/skills/dbos-golang/AGENTS.md) - > **Note:** `CLAUDE.md` is a symlink to this file.
- [`.agents/skills/dbos-python/AGENTS.md`](./.agents/skills/dbos-python/AGENTS.md) - > **Note:** `CLAUDE.md` is a symlink to this file.
- [`.agents/skills/dbos-typescript/AGENTS.md`](./.agents/skills/dbos-typescript/AGENTS.md) - > **Note:** `CLAUDE.md` is a symlink to this file.
- [`.agents/skills/loki-mode/references/AGENTS.md`](./.agents/skills/loki-mode/references/AGENTS.md) - Complete specifications for all 37 specialized agent types in the Loki Mode multi-agent system.
- [`.agents/skills/postgres-best-practices/AGENTS.md`](./.agents/skills/postgres-best-practices/AGENTS.md) - **Version 1.0.0**
- [`.agents/skills/react-best-practices/AGENTS.md`](./.agents/skills/react-best-practices/AGENTS.md) - **Version 0.1.0**

### Home-level instructions

- [`AGENTS.md`](./AGENTS.md) - Before starting any task in this repository, read:

---

## Documents by group (with stats)

### Active global protocol files (.agent-library/active/) (9 files)

#### `.agent-library/active/FABLE_MODE.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\FABLE_MODE.md`
- **Size:** 674 chars, 22 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/FABLE_MODE.md`](./.agent-library/active/FABLE_MODE.md)

#### `.agent-library/active/GLOBAL_AGENT_BOOT.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\GLOBAL_AGENT_BOOT.md`
- **Size:** 888 chars, 28 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/GLOBAL_AGENT_BOOT.md`](./.agent-library/active/GLOBAL_AGENT_BOOT.md)

#### `.agent-library/active/GLOBAL_SWARM_SETUP.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\GLOBAL_SWARM_SETUP.md`
- **Size:** 7,089 chars, 116 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/GLOBAL_SWARM_SETUP.md`](./.agent-library/active/GLOBAL_SWARM_SETUP.md)

#### `.agent-library/active/HANDOFF_PROTOCOL.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\HANDOFF_PROTOCOL.md`
- **Size:** 169 chars, 10 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/HANDOFF_PROTOCOL.md`](./.agent-library/active/HANDOFF_PROTOCOL.md)

#### `.agent-library/active/KANBAN_WORKFLOW.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\KANBAN_WORKFLOW.md`
- **Size:** 2,499 chars, 67 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/KANBAN_WORKFLOW.md`](./.agent-library/active/KANBAN_WORKFLOW.md)

#### `.agent-library/active/SKILL_ROUTER.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\SKILL_ROUTER.md`
- **Size:** 366 chars, 12 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/SKILL_ROUTER.md`](./.agent-library/active/SKILL_ROUTER.md)

#### `.agent-library/active/SWARM_WORKFLOW.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\SWARM_WORKFLOW.md`
- **Size:** 3,575 chars, 100 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/SWARM_WORKFLOW.md`](./.agent-library/active/SWARM_WORKFLOW.md)

#### `.agent-library/active/TASK_PREFLIGHT.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\TASK_PREFLIGHT.md`
- **Size:** 205 chars, 10 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/TASK_PREFLIGHT.md`](./.agent-library/active/TASK_PREFLIGHT.md)

#### `.agent-library/active/VERIFICATION_PROTOCOL.md`

- **Path on disk:** `C:/Users/soren/.agent-library\active\VERIFICATION_PROTOCOL.md`
- **Size:** 331 chars, 21 lines
- **Copy in this repo:** [`AGENTS/.agent-library/active/VERIFICATION_PROTOCOL.md`](./.agent-library/active/VERIFICATION_PROTOCOL.md)


### Agent-specific instructions (.agent-library/agents/) (4 files)

#### `.agent-library/agents/generic/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agent-library\agents\generic\AGENTS.md`
- **Size:** 309 chars, 10 lines
- **Copy in this repo:** [`AGENTS/.agent-library/agents/generic/AGENTS.md`](./.agent-library/agents/generic/AGENTS.md)

#### `.agent-library/agents/hermes/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agent-library\agents\hermes\AGENTS.md`
- **Size:** 345 chars, 10 lines
- **Copy in this repo:** [`AGENTS/.agent-library/agents/hermes/AGENTS.md`](./.agent-library/agents/hermes/AGENTS.md)

#### `.agent-library/agents/kilo/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agent-library\agents\kilo\AGENTS.md`
- **Size:** 289 chars, 10 lines
- **Copy in this repo:** [`AGENTS/.agent-library/agents/kilo/AGENTS.md`](./.agent-library/agents/kilo/AGENTS.md)

#### `.agent-library/agents/opencode/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agent-library\agents\opencode\AGENTS.md`
- **Size:** 274 chars, 8 lines
- **Copy in this repo:** [`AGENTS/.agent-library/agents/opencode/AGENTS.md`](./.agent-library/agents/opencode/AGENTS.md)


### External skill packages (.agents/skills/) (6 files)

#### `.agents/skills/dbos-golang/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agents\skills\dbos-golang\AGENTS.md`
- **Size:** 2,641 chars, 92 lines
- **Copy in this repo:** [`AGENTS/.agents/skills/dbos-golang/AGENTS.md`](./.agents/skills/dbos-golang/AGENTS.md)

#### `.agents/skills/dbos-python/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agents\skills\dbos-python\AGENTS.md`
- **Size:** 2,748 chars, 95 lines
- **Copy in this repo:** [`AGENTS/.agents/skills/dbos-python/AGENTS.md`](./.agents/skills/dbos-python/AGENTS.md)

#### `.agents/skills/dbos-typescript/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agents\skills\dbos-typescript\AGENTS.md`
- **Size:** 2,728 chars, 94 lines
- **Copy in this repo:** [`AGENTS/.agents/skills/dbos-typescript/AGENTS.md`](./.agents/skills/dbos-typescript/AGENTS.md)

#### `.agents/skills/loki-mode/references/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agents\skills\loki-mode\references\AGENTS.md`
- **Size:** 23,078 chars, 1044 lines
- **Copy in this repo:** [`AGENTS/.agents/skills/loki-mode/references/AGENTS.md`](./.agents/skills/loki-mode/references/AGENTS.md)

#### `.agents/skills/postgres-best-practices/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agents\skills\postgres-best-practices\AGENTS.md`
- **Size:** 44,927 chars, 1491 lines
- **Copy in this repo:** [`AGENTS/.agents/skills/postgres-best-practices/AGENTS.md`](./.agents/skills/postgres-best-practices/AGENTS.md)

#### `.agents/skills/react-best-practices/AGENTS.md`

- **Path on disk:** `C:/Users/soren/.agents\skills\react-best-practices\AGENTS.md`
- **Size:** 60,499 chars, 2250 lines
- **Copy in this repo:** [`AGENTS/.agents/skills/react-best-practices/AGENTS.md`](./.agents/skills/react-best-practices/AGENTS.md)


### Home-level instructions (1 file)

#### `AGENTS.md`

- **Path on disk:** `C:/Users/soren/AGENTS.md`
- **Size:** 691 chars, 25 lines
- **Copy in this repo:** [`AGENTS/AGENTS.md`](./AGENTS.md)


---

## Loading order (how Hermes reads these)

When a Hermes session starts, the runtime reads instruction files in roughly this order:

1. **Home-level** `~/AGENTS.md` - top-level directives for the current machine and user.
2. **Active global** `.agent-library/active/*.md` - loaded in alphabetical order. These are the 9 protocol files:
   - `FABLE_MODE.md`
   - `GLOBAL_AGENT_BOOT.md`
   - `GLOBAL_SWARM_SETUP.md`
   - `HANDOFF_PROTOCOL.md`
   - `KANBAN_WORKFLOW.md`
   - `SKILL_ROUTER.md`
   - `SWARM_WORKFLOW.md`
   - `TASK_PREFLIGHT.md`
   - `VERIFICATION_PROTOCOL.md`
3. **Agent-specific** `.agent-library/agents/<variant>/AGENTS.md` - picked based on the active agent variant (currently `hermes`).
4. **Skill AGENTS.md** from `~/.agents/skills/*/AGENTS.md` - per-skill instructions, loaded only when that skill is invoked.

For the full text of every document, see the [`/AGENTS/`](./AGENTS/) directory in this repo.
