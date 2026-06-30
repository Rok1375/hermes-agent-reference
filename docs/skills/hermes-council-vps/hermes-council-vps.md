---
name: hermes-council-vps
description: Run a multi-agent LLM council on a Hetzner Ubuntu VPS using local Ollama models (no API key needed) plus optional OpenRouter/OpenAI/Anthropic providers via the `council` CLI. Use when the user wants a free, self-hosted multi-agent deliberation system or wants to set up the `council` orchestrator on a fresh VPS.
---

# Hermes Council VPS

**What it is**: A self-contained multi-agent LLM deliberation system on a Hetzner Ubuntu 24.04 VPS. Default agents are 4 local Ollama models (free, no API key). Optional OpenRouter / OpenAI / Anthropic via env-loaded keys.

**Why it exists**: The user wanted a multi-agent council on their VPS, prioritizing free / local models. Cloud API keys are nice but optional; local Ollama covers 90% of the value.

## When to use

- User says "set up a council", "multi-agent deliberation", "multi-LLM blackboard"
- User has a Hetzner / similar Ubuntu 24.04 VPS reachable by SSH
- User wants to minimize cost (free local models) without losing optionality for cloud providers
- Hermes Agent or other LLM clients need to be wired to a local Ollama instance

## When NOT to use

- User wants single-agent LLM (use plain `ollama run` instead)
- User has no VPS and doesn't want one
- User requires GPU (Ollama CPU-only is enough; GPU just speeds it up)

## Architecture

```
council ask "task"
  ├─ /tmp/council/<id>/{task,board,state.json,final}.md
  ├─ /tmp/council/<id>/drafts/r{N}_{agent}.md   (1 per agent per round)
  ├─ /tmp/council/<id>/critiques/r{N}_{agent}.md
  ├─ /tmp/council/<id>/votes/r{N}_{agent}.md
  └─ 3 phases × 3 rounds = 9 model calls per agent
```

- **Round**: 1 → N (3 default)
- **Phase per round**: draft → critique → vote
- **Convergence**: ≥ 2/3 of votes are `VOTE` (not `CONTINUE`) → exit early
- **Chair synthesis**: final round's drafts + critiques → `final.md`

## Setup (10 commands, ~15 min on 4 vCPU VPS)

1. `ssh root@<vps>`  (assumes key already in `~/.ssh/authorized_keys`)
2. `apt update && apt install -y curl ca-certificates` (Ubuntu 24.04 needs this; Node/npm and pipx follow)
3. `curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && apt install -y nodejs python3-pip` (Node 22 + pip)
4. `curl -fsSL https://ollama.com/install.sh | sh && systemctl enable --now ollama` (Ollama systemd)
5. `ollama pull qwen2.5-coder:7b llama3.1:8b deepseek-r1:7b phi3:mini` (16 GB; use 8 GB limit per model: `-b 512 -ub 512`)
6. `npm i -g @openai/codex@latest opencode-ai@latest @kilocode/cli@latest @anthropic-ai/claude-code@latest agi@latest`
7. `scp council.py root@<vps>:/usr/local/bin/council && chmod +x /usr/local/bin/council`
8. Optional: `scp claude-bridge root@<vps>:/usr/local/bin/ && scp claude-bridge.service root@<vps>:/etc/systemd/system/ && systemctl enable --now claude-bridge` (needs `ANTHROPIC_API_KEY` in `/root/.config/council/keys.env`)
9. `council list` — should print `no councils`
10. `council ask "Reply with: hello"` — should produce a council folder + `final.md`

## Verified non-interactive incantations (2026-06-28)

| CLI | Command | Env var |
|-----|---------|---------|
| codex | `printenv OPENAI_API_KEY | codex login --with-api-key` once, then `codex exec --skip-git-repo-check "prompt"` | `OPENAI_API_KEY` |
| opencode | `opencode run --model opencode/mimo-v2.5-free "prompt"` (also verified: `opencode/nemotron-3-ultra-free`, `opencode/north-mini-code-free`, `openrouter/google/gemma-4-26b-a4b-it:free`) | `OPENCODE_API_KEY` / `OPENROUTER_API_KEY` / `OPENAI_API_KEY` |
| kilo | `kilo run --model opencode/mimo-v2.5-free "prompt"` (also verified with OpenCode free models) | `OPENCODE_API_KEY` |
| claude | `claude -p "prompt" --output-format text` | `ANTHROPIC_API_KEY` |
| agy | official Google Antigravity CLI: `agy --print "prompt"` after interactive Google OAuth login | browser/OAuth login; does **not** use OpenRouter/OpenAI/OpenCode keys |
| legacy `agi` npm package | installed as `/usr/bin/agi`, but this is **not** Google Antigravity `agy`; it wants `AGI_API_KEY` or `ANTHROPIC_API_KEY` | optional / deprecated here |
| ollama | `curl http://127.0.0.1:11434/v1/chat/completions` | none (local) |

## Council CLI env-var pattern

`/root/.config/council/keys.env` (chmod 600, chown root) with:
```bash
export OPENROUTER_API_KEY="..."
export OPENAI_API_KEY="..."
export OPENCODE_API_KEY="..."
export ANTHROPIC_API_KEY="..."  # for Claude bridge
```

Each CLI subprocess loads this via systemd `EnvironmentFile=-/root/.config/council/keys.env` (the `-` prefix makes it optional). The orchestrator's `run_agent()` checks the required env var per provider and skips the agent gracefully if missing.

## Pitfalls (verified 2026-06-28 to 2026-06-29)

- **Bash heredoc on `ssh` mangles multi-line content.** Always `write_file` locally then `scp`.
- **Terminal tool redaction layer** strips `sk-or-v1-…` strings in every transport. Cannot be bypassed in-session. Fix: user re-supplies via direct SSH from their own terminal.
- **Ollama systemd bind**: default config binds to `127.0.0.1:11434` only. To expose: edit `/etc/systemd/system/ollama.service` and set `OLLAMA_HOST=0.0.0.0:11434` (or tunnel).
- **3 concurrent llama-server workers** at 5-6 GB each = 14-15 GB total RAM. Tight on a 15 GB VPS. Lower `-b 512 -ub 512` if you OOM.
- **Ollama can drop HTTP connections under multi-model load** (`http.client.RemoteDisconnected: Remote end closed connection without response`). The orchestrator must catch `http.client.RemoteDisconnected`, `ConnectionResetError`, and generic exceptions around `urlopen()` so one failed vote does not crash the whole run.
- **CPU-only 4-agent × 3-round** council can fail or take 30-60 min on 4 vCPU. Prefer 2-agent × 3-round (`qwen2.5-coder:7b` + `llama3.1:8b`) for reliable acceptance tests; use `--rounds=1 --timeout=240` for fast smoke tests.
- **Hermes full-agent session via the tunnel hangs**: Hermes startup prompt is ~20K tokens; 4 vCPU CPU-only is too slow. Use council (focused single calls) instead.
- **qwen2.5-coder:7b context = 32K**, not 131K. Use llama3.1:8b or phi3:mini for long prompts.
- **deepseek-r1:7b + qwen3:4b use thinking mode**; for direct Q&A they may emit only reasoning and no final answer. Use llama3.1:8b or phi3:mini for fastest non-thinking replies.
- **Hermes Docker gateway crashes on restart loop**: setting `HERMES_DASHBOARD=1` with `API_SERVER_HOST=0.0.0.0` requires dashboard auth, but no auth provider is configured → container crashes. Fix: set `HERMES_DASHBOARD_HOST=127.0.0.1` so dashboard binds loopback only, bypassing the auth gate.
- **Hermes Docker gateway enters interactive chat mode**: without explicit `command: ["gateway", "run"]`, the container defaults to interactive chat which fails with "Input is not a terminal (fd=0)" and exits → restart loop. Fix: always pass `command: ["gateway", "run"]` in docker-compose AND set `stdin_open: false` + `tty: false`.
- **Workspace "No compatible backend detected"**: Hermes Workspace on port 3002 needs `HERMES_API_URL` AND `HERMES_API_TOKEN` in its `.env` matching the gateway's `API_SERVER_KEY`. Without the token, the workspace can't authenticate to the gateway. Also ensure the workspace process is restarted after `.env` changes.
- **Workspace port conflicts with nginx**: nginx may already occupy port 3000 (used by council-ui). Use a different port (e.g. 3002 for workspace, 3003 for nginx→workspace proxy).

## Launch command

```bash
systemctl start council-ui
```

Or restart:
```bash
systemctl restart council-ui
```

Both `council-ui` and `nginx` are enabled for auto-start on boot.

## Visual dashboard

The VPS has a Flask dashboard installed at `/opt/council-ui`, served by `council-ui.service` on `127.0.0.1:8787`.

### Local access (SSH tunnel)

```bash
ssh -i ~/.ssh/council-agent/id_ed25519 -L 8787:127.0.0.1:8787 root@2.24.114.201
```

Open `http://127.0.0.1:8787`.

### Public URL via Nginx reverse proxy

To expose on a public port (e.g. `srv1689401.hstgr.cloud:3000`):

1. Install nginx: `apt install -y nginx`
2. Write proxy config:

```nginx
server {
    listen 3000;
    server_name <hostname_or_domain>;

    location / {
        proxy_pass http://127.0.0.1:8787;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
        proxy_send_timeout 86400;
    }
}
```

3. Enable and start:
```bash
ln -sf /etc/nginx/sites-available/council-ui /etc/nginx/sites-enabled/council-ui
rm -f /etc/nginx/sites-enabled/default
systemctl reload nginx
```

4. Open firewall:
```bash
ufw allow 3000/tcp   # or: iptables -I INPUT -p tcp --dport 3000 -j ACCEPT
```

Dashboard features:
- visual provider/key/service cards
- dropdown model and CLI tests
- visual council builder (`COUNCIL_AGENTS` generation)
- deterministic persona presets (`Qwen Architect`, `Llama Strategist`, `Phi Scout`, `Deepseek Critic`, `OpenAI Operator`, `Gemma Oracle`)
- editable agent identity, role, provider, model, and glyph
- browser PTY terminal for real CLI use: `bash`, `codex`, `opencode`, `kilo`, `agy`, `claude`
- terminal controls for send, poll, stop, Ctrl-C, Esc, Ctrl-D, Tab
- run viewer for task/board/drafts/critiques/votes/final files
- masked key status; raw keys are never displayed
- `claude-bridge` start/stop/status controls

Dashboard files:
- `/opt/council-ui/app.py`
- `/opt/council-ui/templates/index.html`
- `/etc/systemd/system/council-ui.service`

Verification commands:

```bash
curl http://127.0.0.1:8787/healthz
curl http://127.0.0.1:8787/api/status
systemctl status council-ui
```

## Quick smoke test

```bash
TASK="What is 2+2? Reply with the digit."
COUNCIL_AGENTS='[{"name":"qwen","model":"qwen2.5-coder:7b","provider":"ollama"}]' \
  council ask --rounds=1 "$TASK"
cat /tmp/council/$(ls -t /tmp/council/ | head -1)/final.md
```

Expected: 1-line answer containing `4`, exit 0.

## Reference: where everything lives

- `C:\Users\soren\council.py` (local source) — scp'd to `/usr/local/bin/council`
- `C:\Users\soren\claude_bridge.py` — scp'd to `/usr/local/bin/claude-bridge`
- `C:\Users\soren\claude-bridge.service` — scp'd to `/etc/systemd/system/`
- `C:\Users\soren\README.md` — full user-facing docs
- `C:\Users\soren\DEPLOY.md` — fresh-VPS install recipe
- `C:\Users\soren\TEST_PLAN.md` — T1-T10 acceptance tests
- `C:\Users\soren\AUDIT.md` — 4-reviewer self-audit
- `C:\Users\soren\.ssh\council-agent/id_ed25519` — SSH key for `root@2.24.114.201`
