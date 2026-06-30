---
name: self-improvement-memory
description: "After completing any task, records what worked and what failed into persistent memory and proposes targeted updates to existing skills. The agent's own improvement loop — never repeat the same mistake twice."
version: 1.0.0
metadata:
  hermes:
    tags: [self-improvement, memory, skill-management, core]
    category: core
---

# Self-Improvement Memory — Post-Task Reflection

## When to Use
Use this skill after completing any non-trivial task (3+ tool calls) or after any of these events:
- A task succeeded but required workarounds or debugging
- A task failed and had to be retried with a different approach
- A new workflow or configuration was discovered
- A pitfall was encountered that isn't documented in any existing skill
- The user explicitly said "save that" or "remember this for next time"
- An existing skill's instructions were outdated or incorrect

## Procedure

### Step 1: Reflect on What Happened
After the task is complete and verified, answer these questions:

1. **What worked well?** — Which steps were smooth, fast, or correct on the first try?
2. **What failed or was tricky?** — Which steps needed retries, workarounds, or corrections?
3. **What was discovered?** — New commands, configs, endpoints, or approaches learned during this task.
4. **Was an existing skill used?** — If yes, did it cover the actual workflow accurately?

### Step 2: Record Durable Facts to Memory
Save compact, durable facts that will matter in future sessions. Use `memory` with `target="memory"` or `target="user"` as appropriate.

**What to save:**
- Environment details that were discovered (e.g. "Port 8080 is taken, use 8081 instead")
- Tool quirks or platform-specific behaviors
- User preferences revealed during the task
- Stable configurations that won't change weekly

**What NOT to save:**
- Task progress or what you did today (use session_search for that)
- Temporary state or one-off file paths
- Anything that will be stale in 7 days

### Step 3: Propose Skill Updates
If an existing skill was used and found to be incomplete, wrong, or missing steps:

1. Load the skill with `skill_view(name="skill-name")`
2. Identify the specific outdated or missing section
3. **Show the proposed change to the user** — present the old_string and new_string as a diff-like summary. Include enough surrounding context so the edit is unambiguous.
4. **Wait for explicit user confirmation** before calling `skill_manage(action="patch", ...)`. Never apply skill edits silently or without approval — patches change procedures the user depends on being stable.

**Also: review newly installed or externally-adopted skills.**
When this session involved installing skills from an external source (cloned repo, ECC, GitHub, etc.):

1. Read each newly installed skill's SKILL.md from the source before writing
2. Check for missing provenance, hardcoded paths, environment-specific assumptions, or adapter notes that future sessions need
3. Add a `## Source` section to the skill body (not frontmatter) noting the origin, commit SHA, and any adaptation decisions
4. Present each proposed addition to the user for confirmation before writing

**Patch triggers:**
- Procedure steps were wrong → update the numbered steps
- Pitfalls were encountered but not documented → add them to Pitfalls
- Verification steps were insufficient → expand the verification section
- Trigger conditions were too narrow → broaden the "When to Use" section

### Step 4: Offer to Create a New Skill
If the task was complex (5+ tool calls) or involved a genuinely new workflow:

```
This was a multi-step process. Want me to save this as a skill
so I can reuse it next time?
```

Only create if the user confirms. Use `skill_manage(action="create", ...)` with the full SKILL.md format.

## Pitfalls
- **Saving too much to memory** — compact facts only. If it won't matter in 3 months, skip it or save as a skill instead.
- **Patching skills not in session index** — skills on disk but not loaded in the current session (newly created skills, or files written before `/reload-skills`) cannot be patched via `skill_manage`. Either run `/reload-skills` first, or document the needed change and defer it.
- **Patching without reading first** — always load the current skill content before patching. Never guess what's in there.
- **Creating skills for one-offs** — a one-liner command is not a skill. Reserve skills for 3+ step procedures.
- **Skipping reflection on fast tasks** — even a 2-minute task can reveal a useful fact. Take 5 seconds to reflect.
- **Duplicating existing knowledge** — check if the fact is already in memory or a skill before adding.

## Verification
1. Memory was updated (check output from memory tool).
2. If a skill was patched, confirm the old content was replaced (re-read the skill with `skill_view`).
3. If a new skill was created, confirm it appears in `skills_list`.
4. The reflection took ≤10 seconds — don't over-analyze simple tasks.
