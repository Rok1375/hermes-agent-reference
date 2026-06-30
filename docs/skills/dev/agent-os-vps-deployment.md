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
