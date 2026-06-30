# Category: core

> Deep-dive for the **2 skills** in the `core` category. Each section embeds the full `SKILL.md` body so this doc is self-contained.

**Skills in this category:**

- [`self-improvement-memory`](../skills/core/self-improvement-memory.md) - After completing any task, records what worked and what failed into persistent memory and proposes targeted updates to e
- [`skill-router`](../skills/core/skill-router.md) - Matches incoming requests to the most relevant Hermes skill and loads it automatically. Acts as the gateway between user

---

## self-improvement-memory

- **Name:** `self-improvement-memory`
- **Version:** 1.0.0
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/core\self-improvement-memory/SKILL.md`

**Description:**

> After completing any task, records what worked and what failed into persistent memory and proposes targeted updates to existing skills. The agent's own improvement loop — never repeat the same mistake twice.

**Full SKILL.md body:**

```markdown
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

```

---

## skill-router

- **Name:** `skill-router`
- **Version:** 1.0.0
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/core\skill-router/SKILL.md`

**Description:**

> Matches incoming requests to the most relevant Hermes skill and loads it automatically. Acts as the gateway between user intent and procedural knowledge. Runs before every task to ensure the right skill is active.

**Full SKILL.md body:**

```markdown
---
name: skill-router
description: "Matches incoming requests to the most relevant Hermes skill and loads it automatically. Acts as the gateway between user intent and procedural knowledge. Runs before every task to ensure the right skill is active."
version: 1.0.0
metadata:
  hermes:
    tags: [router, orchestration, skill-management, core]
    category: core
---

# Skill Router — Automatic Skill Dispatch

## When to Use
Use this skill **before every task**. This is the router that decides which skill (if any) matches the user's current request. Trigger on:
- Any new user message or command
- Any file edit or code task
- Any deployment, review, or agent dispatch request
- Any Telegram or external platform response

Do not skip this check. It is the mandatory pre-task lock.

## Procedure

### Step 1: Parse the Incoming Request
Read the user's message. Identify:
- **Intent** — what does the user want done?
- **Domain** — code, devops, research, writing, security, monitoring, etc.
- **Explicit skill mentions** — did they name a skill like `/prism-scan` or `deploy skill`?

### Step 2: Query Available Skills
Run `skills_list` to get all available skills with their names and descriptions. If a skills list was already loaded earlier in this session and nothing has changed, you may skip re-querying.

### Step 3: Match Request to Skills
For each skill, check:

1. **Name match** — does the skill name or category literally match the request domain?
2. **Description match** — does the skill's `description` field overlap with the user's intent?
3. **Explicit invocation** — did the user use a slash command (`/skill-name`) or say "use the X skill"?

Score matches as: exact (use immediately), partial (consider loading), or no match.

### Step 4: Load the Best-Matching Skill
For the highest-scoring match (or multiple if they're complementary):
1. Call `skill_view(name="matched-skill")` to load its full content
2. Follow the skill's **Procedure** to execute the task

If no skill matches, proceed with default reasoning — do not fabricate a skill match.

### Step 5: Route to the Right Tooling
Based on the matched skill's instructions:
- **Code tasks** → use terminal, write_file, patch, search_files
- **Research** → use web_search, web_extract, browser
- **Deployment** → use terminal (SSH/Docker commands), cronjob
- **Analysis** → use the prism/analysis skill instructions
- **Agent dispatch** → use delegate_task for multi-step parallel work
- **Memory/skills** → use memory, skill_manage

### Step 6: Execute and Validate
Perform the task following the loaded skill's procedure. After completion, run the skill's **Verification** section to confirm success.

## Pitfalls
- **Loading too many skills** — only load the most relevant 1-2 skills. Loading 5+ wastes context and confuses execution.
- **Matching by keyword only** — "deploy" might match a deployment skill OR a presentation skill. Read the description, not just the name.
- **Skipping the router** — never bypass this check. Even if the request seems trivial, run the pre-check silently.
- **Overwriting the pre-check** — the 12-question HERMES_ROUTER_PRECHECK runs before this skill loads; do not duplicate it, but do not skip it either. The full pre-check is documented in the companion reference file: `references/hermes-master-skill-router.md` — read it at session start to internalize the 12 questions.
- **Stale skills list** — if the session is long-running, re-query skills_list periodically; new skills may have been added.

## Verification
1. Confirm at least one skill was loaded (or confirm that no skill matched and default reasoning was used).
2. If a skill was loaded, verify the task output matches that skill's expected outcomes.
3. Log the route taken: `[skill-router] Routed: "<request>" → skill: <name> → outcome: success/fail`

```

---
