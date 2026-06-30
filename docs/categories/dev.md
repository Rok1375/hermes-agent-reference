# Category: dev

> Deep-dive for the **3 skills** in the `dev` category. Each section embeds the full `SKILL.md` body so this doc is self-contained.

**Skills in this category:**

- [`agent-os-vps-deployment`](../skills/dev/agent-os-vps-deployment.md) - Deploy OpenCloud Agent OS (multi-agent orchestration system) to a VPS with working parallel and swarm execution engines.
- [`github-skill-import`](../skills/dev/github-skill-import.md) - Clone a GitHub repo containing Hermes skills (or skill documentation), read specified files, extract the SKILL.md schema
- [`token-optimizer`](../skills/dev/token-optimizer.md) - Find the ghost tokens. Audit Claude Code or Codex setup, see where context goes, fix it. Use when context feels tight.

---

## agent-os-vps-deployment

- **Name:** `agent-os-vps-deployment`
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/dev\agent-os-vps-deployment/SKILL.md`

**Description:**

> Deploy OpenCloud Agent OS (multi-agent orchestration system) to a VPS with working parallel and swarm execution engines.

**Full SKILL.md body:**

```markdown
---
name: agent-os-vps-deployment
category: dev
description: Deploy OpenCloud Agent OS (multi-agent orchestration system) to a VPS with working parallel and swarm execution engines.
---

# Agent OS VPS Deployment

Deploy the OpenCloud Agent OS Team Room — a multi-agent orchestration framework — to a remote VPS with fully functional parallel and swarm execution modes.

## Prerequisites
- SSH root access to target VPS
- Python 3.10+ on VPS
- Source project on local Windows under `~/Desktop/lm/`

## Deployment Steps

### 1. VPS Setup
```bash
# Install dependencies
ssh root@<VPS_IP> "apt-get update && apt-get install -y python3-pip python3-yaml nodejs npm"
pip3 install --break-system-packages pyyaml
```

### 2. Transfer Project
```bash
# Compress and SCP
cd "/c/Users/<user>/Desktop/lm"
tar czf /tmp/opencloud-agent-os.tar.gz opencloud-agent-os/
scp /tmp/opencloud-agent-os.tar.gz root@<VPS_IP>:/opt/opencloud-agent-os/
ssh root@<VPS_IP> "cd /opt/opencloud-agent-os && tar xzf opencloud-agent-os.tar.gz --strip-components=1 && rm opencloud-agent-os.tar.gz"
```

### 3. Build Engine Package
Create `/opt/opencloud-agent-os/team-room/engine/` Python package with:

- **`agent.py`** — Agent roster (Hermes, OpenCode, Kilo, AGY, Praison) + parallel/sequential dispatch via threading
- **`room.py`** — Room lifecycle: create room, write intake, agent context, debate, decision card, execution plan, review, final, audit
- **`parallel.py`** — Runs `dispatch_all_parallel()` → writes agent artifacts → Hermes synthesis
- **`swarm.py`** — Runs `dispatch_sequential()` through configurable pipeline (warroom, studio, circle, team, decide, redteam, refine)
- **`conductor.py`** — Routes commands: `/parallel` → parallel engine, `/warroom` → swarm engine, etc.
- **`__main__.py`** — Entry point called by shell scripts

### 4. Replace Shell Scripts
Replace no-op placeholders in `scripts/` with wrappers that call `python3 engine/__main__.py <mode> <args>`:
- `run-parallel.sh` → calls `engine/__main__.py parallel <task>`
- `run-swarm.sh` → calls `engine/__main__.py swarm <mode> <task>`
- `run-agent.sh` → calls `engine/__main__.py agent <name> <task>`
- `team-room.sh` → calls `engine/__main__.py swarm team <task>`
- `team-room-cli.py` — Full CLI with read-only and execution commands
- `decide.py` — Decision engine wrapper

### 5. Update Telegram Handlers
Replace no-op handlers in `telegram/handlers/` with real Python imports from engine.

### 6. Wire Gateway CLIs

The project ships `/opt/opencloud-agent-os/gateway/<name>/<name>` simulation stubs (~204 lines each) that only print fake output. They must be replaced with real proxy wrappers.

**6a. Install real CLIs on VPS**
```bash
npm install -g @anthropic-ai/claude-code@latest
npm install -g @openai/codex
```
Verify: `claude --version` (should return e.g. `2.1.195 (Claude Code)`), `codex --version`.

**6b. Detect stubs vs real binaries**
```bash
file /opt/opencloud-agent-os/gateway/opencode/opencode  # "Python script" — it's a stub
head -1 /opt/opencloud-agent-os/gateway/opencode/opencode  # "#!/usr/bin/env python3"
wc -l /opt/opencloud-agent-os/gateway/opencode/opencode  # 204 lines of simulation
file /usr/local/bin/claude  # "ELF 64-bit executable" — real binary
```

**6c. Replace each gateway stub with a real proxy**

For CLIs with real binaries (`claude`, `codex`, `docker`, `tmux`), write a proxy that calls `os.execvp` to the real binary. Include team-mode routing for agent CLIs:

```python
#!/usr/bin/env python3
"""Real claude gateway — passes through to installed claude binary."""
import sys, os
CLAUDE_BIN = "/usr/local/bin/claude"
PROMPT = " ".join(sys.argv[1:]) if len(sys.argv) > 1 else ""
if PROMPT in ("--version", "-v", "version"):
    subprocess.run([CLAUDE_BIN, "--version"]); sys.exit(0)
team_triggers = ["/team", "/parallel", "/warroom", "/studio", "/circle",
                  "using four agents", "debate mode", "swarm mode"]
is_team = any(t in PROMPT.lower() for t in team_triggers)
if is_team and os.path.exists("/opt/opencloud-agent-os/team-room"):
    os.chdir("/opt/opencloud-agent-os/team-room")
    os.execvp("python3", ["python3", "engine/__main__.py", "parallel", PROMPT])
else:
    os.execvp(CLAUDE_BIN, [CLAUDE_BIN] + sys.argv[1:] if sys.argv[1:] else [CLAUDE_BIN])
```

For agent personas without real binaries (`hermes`, `opencode`, `agy`, `kilo`, `kiro`, `openclaw`), route directly to the team-room engine:
```python
os.chdir("/opt/opencloud-agent-os/team-room")
os.execvp("python3", ["python3", "engine/__main__.py", "single", PROMPT])
```

**6d. Add gateways to system PATH**
```bash
cat > /etc/profile.d/agent-os-gateways.sh << "EOF"
#!/bin/sh
for d in /opt/opencloud-agent-os/gateway/*/; do
  case ":$PATH:" in
    *:"$d":*) ;;
    *) PATH="$d:$PATH" ;;
  esac
done
export PATH
EOF
chmod +x /etc/profile.d/agent-os-gateways.sh
```

**6e. Verify all gateways**
```bash
for g in claude codex hermes opencode docker tmux agy kilo kiro openclaw; do
  which "$g" && "$g" --version 2>&1 | head -1
done
# Test team routing
claude "using four agents build a docker compose file" 2>&1 | head -5
# Should say "EXECUTION COMPLETE" and show agent outputs, not simulation text
```

### 7. Install Additional AI CLIs

The base deployment only installs `claude` and `codex`. Add more CLIs for agent personas:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
nvm use 22 2>/dev/null  # openclaw requires Node 22+

# Kilo Code - full AI coding agent CLI
npm install -g @kilocode/cli

# OpenCode CLI (the current package is `opencode-ai`, binary name is `opencode`)
npm install -g opencode-ai
# (older 2.x builds were `lildax` - if you have that, symlink it as `opencode`:
#  ln -sf /usr/local/bin/lildax /usr/local/bin/opencode)

# Optional: AGY bridge (Google Antigravity via MCP - needs Google OAuth)
npm install -g agy-bridge

# Optional: Kiro provider plugin for opencode (needs AWS Builder ID)
npm install -g opencode-kiro

# Optional: AGI (AGI desktop automation CLI - free on npm, anthropic-compatible)
npm install -g agi
```

Verify:
```bash
codex --version        # 0.142.3+
opencode --version     # 1.17.x
kilo --version         # 7.3.x
claude --version       # 2.1.x
agi --help             # exits 1 without AGI_API_KEY - that's expected
agy-bridge --help      # MCP server entrypoint
```

**Detected naming traps (June 2026):** `agy` and `kiro` as standalone binaries
do NOT exist on npm. Use `agy-bridge` (MCP) for Google Antigravity, and the
`opencode-kiro` plugin for AWS Kiro. Don't waste time hunting for the bare
binary names.

### 8. API Key Bridge - Single Key for All CLIs

A single OpenRouter API key can power most CLIs via env vars and URL overrides, avoiding separate keys per provider.

Create `/etc/profile.d/agent-os-keys.sh`:
```bash
cat > /etc/profile.d/agent-os-keys.sh << "KEYEOF"
#!/bin/sh
export OPENROUTER_API_KEY=***         # Kilo auto-detects
export OPENCODE_API_KEY=***           # OpenCode (lildax)
export OPENCODE_PROVIDER="openrouter"
export OPENCODE_MODEL="auto"
export ANTHROPIC_BASE_URL="https://openrouter.ai/api/v1"
export ANTHROPIC_API_KEY=***          # Same as OpenRouter key
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
export OPENAI_API_KEY=***             # Same as OpenRouter key
export KILO_DEFAULT_MODEL="kilo-auto/free"
KEYEOF
chmod +x /etc/profile.d/agent-os-keys.sh
```

**Claude Code OpenRouter Bridge**: Setting `ANTHROPIC_BASE_URL` to OpenRouter and `ANTHROPIC_API_KEY` to the OpenRouter key routes Claude Code through OpenRouter instead of direct Anthropic. Use `claude --model openrouter/anthropic/claude-sonnet-4` or `claude --model openrouter/auto`.

**Kilo env auto-detection**: Kilo lists `OPENROUTER_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENCODE_API_KEY` under "Environment" in `kilo auth list` automatically when set. No `kilo auth login` needed.

**OpenClaw provider config** (`~/.openclaw/openclaw.json`):
```json
{
  "auth": {
    "profiles": {"openrouter": {"provider": "openrouter", "mode": "api_key"}},
    "order": {"openrouter": ["openrouter"]}
  },
  "agents": {
    "default": {"model": "openrouter/auto", "provider": "openrouter"}
  }
}
```

Gateway wrappers should inject env vars before execvp. For claude:
```python
os.environ.setdefault("ANTHROPIC_BASE_URL", "https://openrouter.ai/api/v1")
os.environ.setdefault("ANTHROPIC_API_KEY", os.environ.get("OPENROUTER_API_KEY", ""))
os.execvp("/usr/local/bin/claude", ["claude"] + sys.argv[1:])
```

**IMPORTANT — Codex gateway must STRIP conflicting env vars** before execvp. The OpenRouter key stored in `OPENAI_API_KEY` causes 401 errors on Codex's native Responses API:
```python
os.environ.pop("OPENAI_API_KEY", None)
os.environ.pop("OPENAI_BASE_URL", None)
os.environ.pop("OPENROUTER_API_KEY", None)
CODEX_BIN = "/usr/local/bin/codex"
os.execvp(CODEX_BIN, [CODEX_BIN] + sys.argv[1:])
```

See `references/api-key-bridge.md` for full env var table and CLI-specific config details.
See `references/cli-auth-methods.md` for per-CLI authentication methods (device auth, API key login, env var detection).

## Agent Architecture

| Agent | Role | Model | Mode |
|-------|------|-------|------|
| Hermes | Conductor / Synthesizer | hermes-3-llama-405b | Governor |
| OpenCode | Builder / Implementer | opencode-go-profile | Parallel + Sequential |
| Kilo | Reviewer / Red Team | kilo-cli-profile | Gate |
| AGY | Scout / Operator | agy-cli-profile | Support |
| Praison | Workflow Team | virtual-swarm-runner | Optional |

## Room Lifecycle
```
00-intake.md → 01-agent-os-context.md → 02-parallel-{agents}.md
→ 07-debate.md → 09-decision-card.md → 10-execution-plan.md
→ 11-review.md → 12-final.md → audit.json
```

## Commands
```bash
# Execution (live agent dispatch)
python3 scripts/team-room-cli.py /parallel "task"
python3 scripts/team-room-cli.py /warroom "task"
python3 scripts/team-room-cli.py /team "task"
python3 scripts/team-room-cli.py /decide "question"
python3 scripts/team-room-cli.py /circle "task"

# Read-only
python3 scripts/team-room-cli.py /teamstatus
python3 scripts/team-room-cli.py /rooms
python3 scripts/team-room-cli.py /room <id>
```

## Verification
```bash
# Run a parallel mode test
python3 engine/__main__.py parallel "Test task"
# Run a swarm mode test
python3 engine/__main__.py swarm warroom "Test task"
# Check rooms
python3 scripts/team-room-cli.py /rooms
# Check gateway CLIs are live
claude --version  # should return real version, not simulation
which claude      # should resolve to gateway dir
```

## Pitfalls

### Simulation stubs masquerading as CLIs
The project ships Python "gateway" scripts (204 lines each) under `/opt/opencloud-agent-os/gateway/<name>/<name>` that only print fake simulation output. They do NOT call any real CLI binary. Detect them with:

```bash
file /opt/opencloud-agent-os/gateway/opencode/opencode  # "Python script" not "ELF executable"
head -1 /opt/opencloud-agent-os/gateway/opencode/opencode  # "#!/usr/bin/env python3"
wc -l /opt/opencloud-agent-os/gateway/opencode/opencode  # ~204 lines of print() spam
```

Fix: replace with real proxy scripts using `os.execvp` (see `references/gateway-proxy-pattern.md`).

### Real CLIs not installed
`npm install -g @anthropic-ai/claude-code` and `npm install -g @openai/codex` are required. Check with `npm ls -g --depth=0`. The binary may be symlinked from a `.exe` name (ELF binary named `claude.exe` symlinked as `claude`).

### Gateways not on PATH
After deploying proxy scripts, add to system PATH:
```bash
cat > /etc/profile.d/agent-os-gateways.sh << "EOF"
#!/bin/sh
for d in /opt/opencloud-agent-os/gateway/*/; do
  case ":$PATH:" in
    *:"$d":*) ;;
    *) PATH="$d:$PATH" ;;
  esac
done
export PATH
EOF
chmod +x /etc/profile.d/agent-os-gateways.sh
```

### Team routing must be wired
Each gateway's proxy must detect team-mode keywords (`/parallel`, `/warroom`, `using four agents`, `debate mode`, `swarm mode`) and route to `engine/__main__.py` instead of the real CLI. Without this, `claude "using four agents build X"` runs the standard Claude agent, not the multi-agent engine.

### OpenRouter free models are the default
For a 4-agent council running 3 rounds (~13 LLM calls per task), use
OpenRouter's `:free` tier to keep cost at zero. The reference models are
`openai/gpt-oss-120b:free` (chair), `qwen/qwen3-coder:free` (code critic),
`meta-llama/llama-3.3-70b-instruct:free` (general critic). See
`references/api-key-bridge.md` for the full free-tier env-var setup and
the **`council-orchestrator`** skill for the full blackboard + rounds
deliberation pattern that runs on these CLIs.

## See also
- **`council-orchestrator`** - the blackboard + 3-round + quorum-vote
  pattern that runs on the CLIs this skill deploys. End-to-end reference
  implementation included.

```

---

## github-skill-import

- **Name:** `github-skill-import`
- **Version:** 1.0.0
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/dev\github-skill-import/SKILL.md`

**Description:**

> Clone a GitHub repo containing Hermes skills (or skill documentation), read specified files, extract the SKILL.md schema/frontmatter format, and create new skills following that schema. End-to-end workflow for importing external skill patterns.

**Full SKILL.md body:**

```markdown
---
name: github-skill-import
description: "Clone a GitHub repo containing Hermes skills (or skill documentation), read specified files, extract the SKILL.md schema/frontmatter format, and create new skills following that schema. End-to-end workflow for importing external skill patterns."
version: 1.0.0
metadata:
  hermes:
    tags: [github, skills, import, schema, workflow]
    category: dev
---

# GitHub Skill Import — Clone, Read, Extract Schema, Create Skills

## When to Use
Use this skill when the user provides:
- A GitHub repo URL containing skills, skill documentation, or skill templates
- A request to "clone these skills into memory" or "read files and create skills from them"
- A specification or guide that defines a schema format for skills
- A multi-part request that chains: clone → read specific files → extract schema → create skills → verify

## Procedure

### Step 1: Clone the Repository
```bash
git clone <repo-url>
```
If the repo was already cloned in this session, skip this step and use the existing directory.

### Step 2: Identify and Read Specified Files
Use `search_files(target="files", path="<repo>")` to discover files. For the requested files:
- Use `read_file()` to load each specified document
- If filenames are approximate, use `search_files(pattern="<keyword>", target="files", path="<repo>")` to locate them

### Step 3: Extract the Schema
From the documentation, identify:
- **Frontmatter fields** — name, description, version, metadata (tags, category)
- **Body sections** — When to Use (trigger conditions), Procedure (numbered steps), Pitfalls, Verification
- **Directory structure** — `~/.hermes/skills/<category>/<skill-name>/SKILL.md`

Output the schema cleanly for reference before creating skills.

### Step 4: Create Target Directories
```bash
mkdir -p ~/.hermes/skills/<category>/<skill-name>
```
Always ensure the parent directory exists before writing SKILL.md.

### Step 5: Write Skills Per Schema
Use `write_file()` to create each SKILL.md with:
- Valid YAML frontmatter (`---` delimiters)
- Full Procedure with numbered steps
- Explicit Pitfalls based on any errors encountered
- Verification section to confirm success

Use proper path: `C:\Users\<user>\AppData\Local\hermes\skills\<category>\<skill-name>\SKILL.md`

### Step 6: Verify Placement
```bash
ls -la ~/.hermes/skills/<category>/<skill-name>/SKILL.md
```
Confirm the files exist at the correct path with expected sizes.

## Pitfalls
- **MSYS/Git-bash path resolution** — `write_file` with a path like `/c/Users/...` resolves to `C:\c\Users\...` instead of `C:\Users\...`. Always verify after writing. If the file landed at the wrong path, recover with:
  ```bash
  cp /c/c/Users/soren/AppData/Local/hermes/skills/<cat>/<skill>/SKILL.md "/c/Users/soren/AppData/Local/hermes/skills/<cat>/<skill>/SKILL.md"
  rm -rf /c/c/Users/soren/AppData/Local/hermes/skills/<cat>
  ```
  To avoid the issue entirely: pass literal `C:\Users\...` strings to `write_file`, or write files via terminal with `cat > path <<'EOF' ... EOF`.
- **Cloning into existing directory** — if the repo directory already exists, `git clone` fails. Remove it first with `rm -rf <dir>` or skip the clone step.
- **Missing schema sections** — if the source docs don't clearly define a schema, infer from examples and note the gap in Pitfalls.
- **Multi-part request ordering** — when the user chains clone → read → schema → create → capture, do not skip intermediate outputs. Each step is a dependency for the next.

## Verification
1. `ls <repo>` shows the cloned repository.
2. `read_file` returns the requested documents with content.
3. Schema template is correct and complete.
4. `ls ~/.hermes/skills/<category>/<skill-name>/SKILL.md` confirms each skill was saved.
5. Re-read one skill file to validate frontmatter and body structure.
6. Confirm no files landed at a `C:\c\...` wrong-path artifact — `find ~/.hermes/skills -path "*\\c\\*" 2>/dev/null` should return empty.

```

---

## token-optimizer

- **Name:** `token-optimizer`
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/dev\token-optimizer/SKILL.md`

**Description:**

> Find the ghost tokens. Audit Claude Code or Codex setup, see where context goes, fix it. Use when context feels tight.

**Full SKILL.md body:**

```markdown
---
name: token-optimizer
description: Find the ghost tokens. Audit Claude Code or Codex setup, see where context goes, fix it. Use when context feels tight.
effort: high
---

# Token Optimizer

Audits a Claude Code or Codex setup, identifies context window waste, implements fixes, and measures savings.

**Target**: 5-15% context recovery through config cleanup, up to 25%+ with autocompact management.

---

## Step 0: Resolve measure.py, then gate on runtime (run this first)

Resolve the script path **once, before any phase or runtime decision**. Every
command below — including the runtime gate — depends on `$MEASURE_PY`, so it
must be set first:
```bash
MEASURE_PY=""
for f in "$HOME/.claude/skills/token-optimizer/scripts/measure.py" \
         "$HOME/.claude/plugins/cache"/*/token-optimizer/*/skills/token-optimizer/scripts/measure.py; do
  [ -f "$f" ] && MEASURE_PY="$f" && break
done
[ -z "$MEASURE_PY" ] && { echo "[Error] measure.py not found. Is Token Optimizer installed?"; exit 1; }
```

With `$MEASURE_PY` resolved, run the runtime gate as the **first executed
command**. Its output is a hard stop, not a hint:
```bash
python3 "$MEASURE_PY" report 2>&1 | head -1
```

- Prints **"Token Optimizer — OpenCode runtime detected."** → **STOP. Run none
  of the phases below.** Read `references/opencode-workflow.md` and follow it.
  The Claude Code phases scan and mutate `~/.claude`, which is the wrong target
  when the user is in OpenCode (issue #57).
- Prints any other **"… runtime detected."** notice (for example GitHub
  Copilot) → STOP and follow that runtime's guidance, for the same reason.
- Otherwise continue: if `TOKEN_OPTIMIZER_RUNTIME=codex` or a Codex environment
  is detected, read `references/codex-workflow.md` and follow its chat-first
  workflow instead of the phases below. Genuine Claude Code proceeds to Phase 0.

---

## Phase 0: Initialize (Claude Code)

`MEASURE_PY` was already resolved in Step 0 — do **not** re-resolve it.

Read `references/phase0-setup.md` for the full setup sequence: context window detection, pre-check, backup, coordination folder, hook checks, daemon setup, and smart compaction.

---

## Phase 0.5: Keep-Warm Consent (first run only, Claude Code)

Keep-Warm is opt-in and pays off only for API-key-billed Claude Code sessions. Ask once:

```bash
python3 "$MEASURE_PY" keepwarm-consent-status   # JSON: {billing_mode, consent, should_ask}
```

If `should_ask` is `false`, skip this phase silently (subscription users are never asked; declined/enabled users keep their choice). If `should_ask` is `true`, first compute the user's own projection, then present the pitch:

```bash
python3 "$MEASURE_PY" keepwarm-backfill --json --no-fence   # read modes."probe-only".net_usd
```

Read `net_usd` under `modes."probe-only"`. If it is a positive number, include it as the projection. If backfill errors, returns nothing, or `net_usd <= 0`, drop the dollar sentence entirely (do not invent a number) and use the no-data wording below.

> **Keep your prompt cache warm automatically?** When a Claude Code session pauses past its 1h cache window and resumes, the whole prefix is re-written at up to 2x input. Keep-Warm pings the cache just before expiry (~0.1x of the prefix, max 2 pings per pause) so a resume stays warm. A history-replay projection from your own last 30 days nets ~$<net_usd>/30d at the conservative probe-only setting. A tripwire auto-disables it if pings ever stop paying for themselves, and you can turn it off any time. Enable it?

No-data wording (when backfill yields no positive projection): drop the projection sentence and say "Your savings depend on your own pause-and-resume pattern; the dashboard will show your number once pings have fired."

Then record the answer (do this exactly once). **Record the yes/no FIRST**, so an interrupted run never strands an "asked" marker with no recorded answer:

```bash
# yes:
python3 "$MEASURE_PY" keepwarm-enable
# no:
python3 "$MEASURE_PY" keepwarm-disable
```

`keepwarm-enable` and `keepwarm-disable` are terminal states, so they already satisfy `should_ask`. Only if the user defers or ignores the question (records neither) run the shown-marker so they are not re-asked next run:

```bash
python3 "$MEASURE_PY" keepwarm-consent-asked          # mark shown (sticky); use ONLY when no enable/disable was recorded
```

`keepwarm-enable` records consent and installs the scheduler (macOS); on other OSes the scheduler is pending, so it is watchdog-only. It refuses on subscription with an honest message. To confirm it is armed:

```bash
python3 "$MEASURE_PY" keepwarm-scheduler status      # JSON: installed/loaded state (macOS)
python3 "$MEASURE_PY" keepwarm-tick --dry-run        # JSON: what the next tick would decide
```

---

## Phase 0.6: Star the repo (first run only, once ever)

A one-time, no-pressure offer to star the repo. It is gated so it only ever surfaces for someone who has already gotten value (the gate checks `gh` is available, the repo is not already starred, and the user has session history). Check the gate:

```bash
python3 "$MEASURE_PY" star-status   # JSON: {consent, gh_available, already_starred, has_value, should_ask}
```

If `should_ask` is `false`, skip this phase silently (already asked, already starred, no `gh`, no value yet, or disabled via `TOKEN_OPTIMIZER_STAR_ASK=0`). If `should_ask` is `true`, make the offer warmly and briefly, and make declining effortless:

> **Enjoying Token Optimizer?** If it's been saving you tokens, a GitHub star helps other people find it. Want me to star it for you? (One tap, and I won't ask again either way.)

Then record the answer exactly once:

```bash
# yes:
python3 "$MEASURE_PY" star-now        # runs gh api -X PUT /user/starred/...; sets consent=starred
# no:
python3 "$MEASURE_PY" star-decline    # terminal; never asked again
```

Only if the user defers or ignores the question (records neither) mark it shown so they are not re-asked:

```bash
python3 "$MEASURE_PY" star-consent-asked   # mark shown (sticky); use ONLY when no star/decline was recorded
```

---

## Phase 1: Quick Audit (Parallel Agents)

Read `references/agent-prompts.md` for all prompt templates.

Dispatch 6 agents in parallel:

| Agent | Output File | Model | Task |
|-------|-------------|-------|------|
| CLAUDE.md Auditor | `audit/claudemd.md` | sonnet | Size, duplication, tiered content, cache structure |
| MEMORY.md Auditor | `audit/memorymd.md` | sonnet | Size, overlap with CLAUDE.md |
| Skills Auditor | `audit/skills.md` | sonnet | Count, frontmatter overhead, duplicates |
| MCP Auditor | `audit/mcp.md` | sonnet | Deferred tools, broken/unused servers |
| Commands Auditor | `audit/commands.md` | haiku | Count, menu overhead |
| Settings & Advanced | `audit/advanced.md` | sonnet | Hooks, rules, settings, @imports, caching |

Pass `COORD_PATH` to each. Wait for all to complete. If any output file is missing, note the gap and proceed.

---

## Phase 2: Analysis

Read the **Synthesis Agent** prompt from `references/agent-prompts.md`. Dispatch with `model="opus"` (fallback: sonnet). It reads all audit files and writes `{COORD_PATH}/analysis/optimization-plan.md`. If missing, present raw audit files instead.

---

## Phase 3: Present Findings

Read `references/presentation-workflow.md` for the findings template, dashboard generation, and URL presentation logic. Generate the dashboard:
```bash
python3 $MEASURE_PY dashboard --coord-path $COORD_PATH
```
Wait for user decision before proceeding.

---

## Phase 4: Implementation

Read `references/implementation-playbook.md` for detailed steps. Available actions: 4A-4P covering CLAUDE.md, MEMORY.md, Skills, File Exclusion, MCP, Hooks, Cache, Rules, Settings, Descriptions, Compact Instructions, Model Routing, Smart Compaction, Quality Check, Version-Aware Optimizations, and Smart Routing. Templates in `examples/`. Always backup before changes. Present diffs for approval.

---

## Phase 5: Verification

Read the **Verification Agent** prompt from `references/agent-prompts.md`. Dispatch with `model="haiku"`. Re-measures everything and calculates savings. Present before/after comparison and behavioral next steps.

---

## Session Continuity: Cold-Resume-Lean

Reopen a forgotten/cold session cheaply, no `--resume`, no command. On a fresh
session, when the user naturally asks to continue prior work ("continue the X
work, check what we discussed last session"), the continuity hook reconstructs a
**lean** context for the right **same-project** prior session and injects it.

- **Selection** ("both"): if the user names a topic → keyword-match winner; if
  vague ("where we left off") → most-recent same-project session.
- **Token-free**: reconstruction reads checkpoints + `session_log` only (no LLM,
  no subprocess). The only cost is the fresh session's normal first turn.
- **Same-project = files touched** (path-prefix vs cwd), never a cross-project leak.
- **Savings** are credited as a realized `resume_lean` event (avoided cold-resume
  cache-rewrite minus the lean block), idempotent per target session, shown in the
  Savings view. Realized tier, same as `checkpoint_restore`.
- **Manual fallback**: `python3 $MEASURE_PY resume-lean` lists cold sessions;
  `resume-lean <#|session_id> --print` emits the block for `claude "$(...)"`.
- Ported across Claude Code, Codex, OpenClaw, opencode (checkpoint richness varies
  by platform; the lean block adapts to available fields).

---

## Reference Files

| Context | Read |
|---------|------|
| Codex runtime | `references/codex-workflow.md` |
| Phase 0 setup details | `references/phase0-setup.md` |
| Phase 1-2 agent prompts | `references/agent-prompts.md`, `references/token-flow-architecture.md` |
| Phase 3 presentation | `references/presentation-workflow.md` |
| Phase 4 implementation | `references/implementation-playbook.md`, `examples/` |
| CLI commands | `references/cli-reference.md` |
| Phase 3 checklist | `references/optimization-checklist.md` |
| Error handling | `references/error-recovery.md` |

---

## Core Rules

- Quantify everything (X tokens, Y%)
- Create backups before any changes
- Ask user before implementing
- Never delete files, always archive outside the skills directory
- Check dependencies before archiving (skills, MCP, deny rules can break other tools)
- Warn about side effects before each change
- Prefer project-level deny rules over global
- Show before/after diffs
- Frame savings as context budget (% of window), not dollar amounts

```

---
