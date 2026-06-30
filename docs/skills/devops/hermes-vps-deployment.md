---
name: hermes-vps-deployment
description: Deploy Hermes Agent on a Linux VPS so you can talk to it from anywhere — phone via Telegram/Discord/WhatsApp, laptop via SSH tunnel, or paired with the Hermes Desktop GUI. Covers Docker Compose setup for the gateway + messaging platforms, common crash patterns, network/tunnel exposure, and end-to-end verification. Use when the user says "talk to Hermes from my phone", "remote gateway", "deploy Hermes on a VPS", "Telegram/Discord bot on my VPS", "Hermes desktop + VPS", or "I have a VPS, set up Hermes on it".
---

# Hermes Agent VPS Deployment

Deploy Hermes Agent gateway, workspace, and dashboard to a VPS. Covers Docker Compose setup, common crash patterns, and network proxying.

## Quick Start

```bash
mkdir -p /root/hermes-vps && cd /root/hermes-vps
```

### docker-compose.yml

```yaml
services:
  gateway:
    image: nousresearch/hermes-agent:latest
    restart: unless-stopped
    environment:
      - HERMES_DASHBOARD=1
      - HERMES_DASHBOARD_BIND=127.0.0.1
      - GATEWAY_ALLOW_ALL_USERS=true
      - HERMES_API_KEY=<your-secret-key>
      - OPENROUTER_API_KEY=sk-or-v1-<key>
      - OPENAI_API_KEY=sk-pro-<key>
      - MODEL_DEFAULT=gpt-4o
      - MODEL_PROVIDER=openai-api
      - MODEL_BASE_URL=https://api.openai.com/v1
      - TERMINAL_BACKEND=local
    entrypoint: ["/bin/sh", "-c", "hermes gateway start"]
    ports:
      - "8642:8642"
    volumes:
      - hermes-data:/opt/data
volumes:
  hermes-data:
    name: hermes-data
```

Start: `docker compose up -d`

## Critical Pitfalls

### 1. Container Crash Loop (Exit Code 2)
**Symptom:** Container starts, then crashes and restarts repeatedly.
**Cause:** One of:
- Gateway command uses wrong syntax (see #2 below)
- Dashboard binds to `0.0.0.0` without auth providers
- Model provider mismatch (model not available on configured API)

**Fix:** Check logs: `docker logs <container-name> 2>&1 | grep -iE "error|fail|fatal"`

### 2. Gateway Command Syntax
**WRONG:** `command: hermes gateway --port 8642 --host 0.0.0.0`
**WRONG:** `command: hermes gateway start --port 8642`

**CORRECT:** Use `entrypoint` with subcommand only:
```yaml
entrypoint: ["/bin/sh", "-c", "hermes gateway start"]
```

The `hermes gateway` CLI uses **subcommands** (start/run/stop/restart/status), NOT `--port`/`--host` flags. Port is set via `ENV` variables or config.yaml.

### 3. Dashboard Binding to 0.0.0.0
The dashboard refuses to bind to `0.0.0.0` without authentication providers registered.

**FIX:** Set `HERMES_DASHBOARD_BIND=127.0.0.1` (or use `HERMES_DASHBOARD_BIND=0.0.0.0` with a configured auth provider via `dashboard.basic_auth.username` in config.yaml).

### 4. Model Provider Mismatch
**Symptom:** `HTTP 401: Missing Authentication header` — model requested is not available on the configured provider's API.

**Example:** `model: o3` is OpenAI-only, but config had `base_url: https://openrouter.ai/api/v1` — OpenRouter doesn't host o3.

**Fix:** Match model to provider:
- `gpt-4o`, `o3` → `MODEL_PROVIDER=openai-api`, `MODEL_BASE_URL=https://api.openai.com/v1`
- `claude-sonnet-4`, `llama-3.1-70b` → `MODEL_PROVIDER=openrouter-api`, `MODEL_BASE_URL=https://openrouter.ai/api/v1`
- Use `openrouter/fusion` or `openrouter/auto` for multi-model routing

### 5. S6 Init System Overrides entrypoint
The `nousresearch/hermes-agent:latest` image uses s6-overlay which has its own service definitions. If the default service doesn't match your needs, use `entrypoint` to override the CMD, but the s6 init system will still run its own `hermes gateway` service. **Solution:** Use `entrypoint` with explicit `hermes gateway start` to bypass the default interactive TUI startup.

### 6. Interactive TUI Mode
**Symptom:** Container shows the ASCII art welcome screen and hangs, never starts listening on port 8642.

**Cause:** The gateway command enters interactive TUI mode when stdin is not a terminal or when no explicit `start` command is given.

**Fix:** Use `entrypoint: ["/bin/sh", "-c", "hermes gateway start"]` — the `start` subcommand runs as a daemon.

## Talking to Hermes From Your Phone / Remote Devices

The gateway on a VPS unlocks three access patterns. Pick the one that matches the user's situation — don't ask "which do you want" before you know they actually have a VPS / a phone messenger / a laptop.

| Pattern | When it fits | What's needed |
|---------|--------------|---------------|
| **A. Telegram / Discord / WhatsApp bot on VPS** | User wants to talk to Hermes from their phone. Gateway runs messaging platform. | `hermes gateway setup` inside the container; bot token from @BotFather; `TELEGRAM_ALLOWED_USERS` set to the user's numeric user ID (NOT @handle). |
| **B. Local Desktop GUI ↔ remote gateway** | User has Hermes Desktop on their laptop, wants the heavy work on VPS. | `HERMES_API_KEY` on VPS; tunnel (Tailscale/WireGuard/SSH) from desktop to VPS port 8642; desktop's `model.base_url` points at the tunnel. |
| **C. SSH tunnel + Dashboard in browser** | User wants web access from a laptop. | Same as B but the user opens `http://127.0.0.1:8642` in a browser after running the tunnel command. |

### Pattern A: Telegram (most common, do this first)

Inside the running container:

```bash
docker exec -it <container-name> hermes gateway setup
# Pick: Telegram → paste bot token from @BotFather → paste your numeric user ID
# Then restart:  docker restart <container-name>
```

The token and user ID are entered by the user directly into the setup prompt — **never ask the user to paste the bot token or any private key into chat**. The agent should only ask for the user's numeric Telegram user ID (which is public — get it from @userinfobot).

See `references/phone-messenger-access.md` for the full Telegram flow (BotFather steps, allowlist, verification commands) plus Discord/WhatsApp variants.

### Pattern B: Pair local Hermes Desktop with the VPS gateway

**Critical pitfall — verified by code search, not assumption:** the Hermes Desktop GUI does NOT have a single "gateway URL" field you can just point at a remote host. The cleanest pairing uses the same `model.base_url` config that the local gateway already uses, pointed at a tunnel endpoint. So the recipe is:

1. Pick a tunnel strategy (SSH tunnel is simplest; Tailscale gives a stable IP; WireGuard is manual). The decision affects the rest of the steps — surface it in your first reply, don't bury it.
2. Generate `HERMES_API_KEY` on the VPS with `openssl rand -hex 32`. Print it once and tell the user to save it (this is the ONE value the user must capture and type elsewhere).
3. On Windows, run the tunnel in a terminal that stays open:
   ```bash
   ssh -L 8642:127.0.0.1:8642 -N root@<VPS-IP>
   ```
4. On the desktop side, point Hermes at the tunnel:
   ```bash
   hermes config set model.base_url http://127.0.0.1:8642/v1
   hermes config set model.api_key <the-HERMES_API_KEY-from-step-2>
   ```
5. Verify end-to-end: `curl http://127.0.0.1:8642/v1/models -H "Authorization: Bearer ***"` should return a model list. If it does, the desktop will work.

**If the user wants the desktop GUI to keep working when they're away from home, the tunnel has to be persistent** — Windows Task Scheduler (same pattern as `remote-ollama-tunnel` skill), or a Tailscale/SSH keepalive. Don't ship a "manual ssh every time" setup to a user who said "I want to access this from outside."

### Pattern C: SSH tunnel + Dashboard in browser

After the same `ssh -L 8642:...` tunnel from Pattern B, open `http://127.0.0.1:8642` in a browser. The dashboard will be gated behind the same `HERMES_API_KEY`. If the dashboard refuses to bind to 0.0.0.0 (Pitfall #3 above), that's expected — the tunnel keeps it on loopback and the dashboard is happy.

## Common Pitfalls When Users Describe Their Goal

- **"I want to talk to Hermes from my phone"** does NOT mean "set up a VPS." It means: pick from Telegram (easiest, no infrastructure), SSH-tunnel to the existing desktop (free, fragile), or VPS gateway + Telegram (24/7, ~$4/mo). Ask which before building.
- **"I have a VPS, just use that"** still doesn't tell you: VPS OS, what's already running, whether Docker is installed, which model API key to use, what auth the user has (password vs key). Get these in one ask, not one-per-turn.
- **Never describe a UI flow you haven't verified.** If you say "go to Settings → X and paste the token," you must have grep'd for `X` in the codebase or test-driven the menu. The Hermes Desktop GUI is still moving; the field that "should" exist may not.
- **Don't ask the user to paste private credentials into chat** (bot tokens, API keys, SSH private keys, VPS passwords). Describe the file path they'll edit on the VPS and the line they'll type, with placeholders they fill in themselves.
- **Don't expose the gateway port publicly by accident.** Default `127.0.0.1:8642` is correct for tunnel-only access. `0.0.0.0:8642` requires `HERMES_API_KEY` to be set AND non-empty AND `dashboard.basic_auth` to be configured — skip it unless the user is sure they want it.

## Workspace + Gateway Setup

The Hermes Workspace is a separate Node.js app (not Docker) that connects to the gateway:

```bash
# Install workspace
cd /root && git clone <workspace-repo> hermes-workspace
cd /root/hermes-workspace && pnpm install

# Run with correct env vars
HERMES_API_URL=http://127.0.0.1:8642 \
HERMES_API_TOKEN=<same-secret-as-gateway> \
PORT=3002 \
node server-entry.js
```

## Nginx Reverse Proxy

```nginx
server {
    listen 3003;
    server_name <your-domain-or-ip>;
    
    location / {
        proxy_pass http://127.0.0.1:3002;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## API Key Distribution
1. Write keys to `/root/.config/council/keys.env` (host)
2. Copy to container: mount the host `.env` into the container
3. Ensure file permissions allow the `hermes` user to read: `chmod 644 /root/.hermes/.env`

## Verification Steps
1. `docker ps` — container running
2. `curl -s http://127.0.0.1:8642/v1/models -H "Authorization: Bearer <token>"` — returns model list
3. `curl -s -X POST http://127.0.0.1:8642/v1/chat/completions ...` — test a chat completion
4. Open workspace URL — verify "backend connected" (not "No compatible backend")
## References

- `references/phone-messenger-access.md` — Step-by-step setup for Telegram / Discord / WhatsApp bot access from a phone, with the exact `docker exec hermes gateway setup` flow, allowlist mechanics, and a smoke-test checklist
- `references/gateway-crash-patterns.md` — Detailed crash debugging recipes
- `references/model-provider-matrix.md` — Model-to-provider mapping