# Global Agent Instructions - ALWAYS READ BEFORE ANY TASK

## Purpose

This is the universal preflight boot file. All agents must read and follow this before starting work.

## Preflight Sequence

Before any task:
1. Read nearest project AGENTS.md, CLAUDE.md
2. Read .agent-library/active/TASK_PREFLIGHT.md
3. Run skill router from .agent-library/active/SKILL_ROUTER.md
4. Select relevant skills
5. Apply Fable mode only if enabled
6. Identify task type: coding, debugging, frontend, infra, agent-config, prompt, research
7. Load matching agent adapter

## Task Types
| Type | Skills |
|------|--------|
| coding | coding, repo-analysis, testing, verification |
| debug | debugging, safety, testing, verification |
| design | design, web-ui, testing, verification |
| infra | infra, safety, verification |

## Verification Protocol
Follow VERIFICATION_PROTOCOL.md before claiming "done".
