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
