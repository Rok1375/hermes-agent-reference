---
name: remote-service-incident-debugging
description: Debug a stuck or broken production service running on a remote host (VPS, container, server) over SSH, with no source tree at the local cwd. Use when a user reports an app is "stuck on a loading screen", "frozen on splash", "won't connect", returns 502, or has a dead listener — and the only way in is SSH.
type: tool_workflow
version: 1.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [debugging, remote, ssh, production, incident, webapp, splash, 502, listener, vps, devops]
    related_skills: [systematic-debugging, hermes-vps-deployment, delegating-to-ai-agents, dogfood]
---

# Remote Service Incident Debugging

Reverse-engineer a broken production service when you can only reach it over SSH, you have no source tree locally, and the user describes symptoms in user-language ("stuck on splash", "page won't load", "502 error").

This skill is the bridge between "user reports a symptom in a browser" and "I have a root cause and a verified fix on the remote host." It supplements `systematic-debugging` (general phase model) with the specific shape remote production incidents take: process state, listening sockets, reverse proxies, public vs loopback binding, and "the process isn't supervised."

## When to use

- User reports a web app / API is stuck on a loading screen, splash, or progress bar.
- User reports a page returns 502 Bad Gateway, 504, or "connection refused".
- User reports a service "ran fine yesterday" but is dead now and won't restart on its own.
- You can SSH into the host; the source code is on the remote, not local.
- The user cannot see the host's logs directly; you must produce them.

**Do not use** for: code-level bugs in source you have locally (use `systematic-debugging`), local dev-server issues (`/agents` in your local cwd), or container-internal debugging when you have direct shell access to the container (`docker exec` is enough).

## The 9-step protocol

Work in this exact order. Each step's output feeds the next.

### 1. Project location

There is no project at your local cwd. **Find it on the remote first.**

```bash
ssh user@host "ls -la /root /home 2>/dev/null; find / -maxdepth 4 \\( -name package.json -o -name pyproject.toml -o -name Cargo.toml -o -name go.mod -o -name docker-compose.yml \\) -not -path '*/node_modules/*' 2>/dev/null | head -20"
```

- Look for: `package.json` + `vite.config.*` / `next.config.*` (Node web), `pyproject.toml` (Python), `docker-compose.yml` (containers), `Dockerfile`, `wrangler.jsonc`, `flake.nix`.
- Note the project root, package manager (lockfile), and `scripts.start` from `package.json` if present.

### 2. Tech stack + intended start

From the project root:

```bash
ssh user@host "cd /root/<project> && cat package.json | jq -r '.scripts, .dependencies | keys'"
```

- Identify: frontend port (Vite default 5173, Next 3000), backend port (often 3000/3001/8642/etc), database, Docker usage, env files.
- Identify the canonical start command: `pnpm dev`, `node server-entry.js`, `npm start`, etc.
- Look for an existing `start-*.sh` or `scripts/start-stable.sh` — **use it if it exists.** It writes a PID, polls a readiness endpoint, and has been hardened by previous sessions.

### 3. Capture the failure (network-side, before touching code)

Always run these on the **remote host** (or against its public IP from your local machine, depending on the user's path). Curl with `-w` to capture timing and code in one line.

```bash
# Direct to the suspected backend port
curl -s -o /dev/null -w "code=%{http_code} time=%{time_total}s size=%{size_download}\n" -m 5 http://127.0.0.1:<port>/

# Through any reverse proxy (nginx / caddy / cloudflare)
curl -s -o /dev/null -w "code=%{http_code} time=%{time_total}s size=%{size_download}\n" -m 5 http://127.0.0.1:<proxy-port>/

# The splash's polling endpoint specifically (find it in the splash component source)
# Usually /api/auth-check, /api/health, /health, or /api/status
curl -s -m 5 http://127.0.0.1:<port>/api/auth-check

# Verbose, to see whether the failure is RST, timeout, or refused
curl -v -m 5 http://127.0.0.1:<port>/ 2>&1 | head -20
```

The pattern `code=000 time=0.0001s` means **ECONNREFUSED** — the listener is dead, not slow. Distinguish from `code=502` (proxy can't reach upstream) and `code=200` (everything works, the bug is elsewhere).

### 4. Trace the splash/loading chain in the source

Most "stuck" web apps have a React/Vue component that polls an endpoint and shows a failure panel after a timeout. Find it.

```bash
ssh user@host "cd /root/<project> && grep -rln -E 'splash|loading|Loading' src/ 2>/dev/null | head -20"
ssh user@host "cd /root/<project> && sed -n '1,200p' src/components/connection-startup-screen.tsx"
ssh user@host "cd /root/<project> && cat src/lib/<auth-file>.ts"
```

Extract:
- The component that owns the splash state.
- The async function that must resolve for the splash to dismiss.
- The exact endpoint the splash polls (e.g. `fetchClaudeAuthStatus()` → `/api/auth-check`).
- The timeout at which the splash shows a failure panel (often 5s).

This tells you what "fixed" means: the polled endpoint must return the success JSON before the timeout.

### 5. Process state — is the service even running?

```bash
ssh user@host "ps -ef | grep -E 'node|server-entry|<project-name>' | grep -v grep"
ssh user@host "ss -tlnp 2>/dev/null | grep -E '<port>|<other-port>'"
```

Match the PID against what was expected. If the process is dead, that's the first finding. If `ss -tlnp` doesn't show the port, the process is **not listening** — even if the process is alive (it could be mid-startup, or bound to the wrong interface).

### 6. Log triage

```bash
# Find logs
ssh user@host "ls -la /tmp/*.log 2>/dev/null; find /root/<project> -maxdepth 3 -name '*.log' 2>/dev/null"

# For services started by nohup or systemd:
ssh user@host "journalctl -u <service>.service --no-pager -n 100 2>/dev/null"

# OOM or kill detection:
ssh user@host "dmesg 2>/dev/null | grep -iE 'killed|oom|node' | tail -20"
```

Distinguish:
- **Process started clean, died silently** → look for `journalctl`, `dmesg`, supervisor's restart log.
- **Process never started** → look at build output, env validation in the start script.
- **Process in crash loop** → check the service's last log line for the error pattern. `systemctl status <svc> --no-pager` shows restart count and exit code.

### 7. Diagnosis statement

Before fixing, write a one-sentence root-cause hypothesis and a one-sentence verification step. Example:

> "Root cause: `node server-entry.js` (PID 112822) exited and is not supervised, so port 3002 is unbound; nginx on :3003 returns 502; the React splash polls `/api/auth-check` forever. Fix: run `scripts/start-stable.sh` (rebuilds, starts, polls readiness). Verify: `curl /api/auth-check` returns 200 with `authRequired:false`."

If you can't state a root cause, **stop** and gather more evidence. The fix is not "restart the service" — the fix is "make the service stay up."

### 8. Apply the smallest confirmed fix

Common fixes in order of likelihood:

1. **Service died, no supervisor** → restart it. **Then** add a systemd unit with `Restart=on-failure` and `RestartSec=5` so it survives future crashes. Don't ship the restart without the unit.
2. **Service alive, port not listening** → middleware or upstream (DB, gateway) is blocking startup. Read the log; the "Configured API" or "Connected to" line is usually the last successful step.
3. **Service listening on `127.0.0.1` only, user reaches it via public IP** → check the proxy (`/etc/nginx/sites-enabled/`); fix the `proxy_pass` target.
4. **Service listening on `0.0.0.0` but splash still polls forever** → CORS, or a mismatch between the build's hardcoded API URL and the runtime's URL. Check the build's `import.meta.env.VITE_*` or `process.env.*` and the actual served HTML.
5. **Service is in a tight crash loop** → fix the upstream first. `claude-bridge.service` looping with `ERROR: ANTHROPIC_API_KEY env var is required` is a config issue, not a code issue.

When the fix is a script: back up first, then run it through Hermes's `background=true` (see pitfall #1 below). When the fix is a file edit: back up with `cp -a file file.backup.<timestamp>`.

### 9. Verify end-to-end through the user's path

The fix isn't verified until the user can hit the URL and see the splash dismiss. Always curl the **exact same path the user hits**:

```bash
# Direct (loopback)
curl -s -w "\ncode=%{http_code} time=%{time_total}s\n" -m 10 http://127.0.0.1:<backend-port>/<splash-endpoint>

# Through the public proxy the user reaches
curl -s -w "\ncode=%{http_code} time=%{time_total}s\n" -m 10 http://127.0.0.1:<proxy-port>/<splash-endpoint>

# The readiness probe the start script uses
curl -s -o /dev/null -w "code=%{http_code} time=%{time_total}s size=%{size_download}\n" -m 10 http://127.0.0.1:<backend-port>/<readiness-path>

# The static asset hashes change with every build — fetch the index and check the new hashes
curl -s http://127.0.0.1:<proxy-port>/ | grep -oE 'assets/main-[^"]+\.js' | head -1
```

If all three return 200 and the splash's specific endpoint returns the success JSON, the fix is verified. If the user reports a stale page after the fix, tell them to **hard refresh** (Ctrl+Shift+R / Cmd+Shift+R) — old service workers and HTML caches will keep the splash alive even after the server is fixed.

## Pitfalls

### 1. The runtime blocks `nohup ... &` and `setsid` inside foreground `terminal` calls

Hermes's `terminal` tool explicitly refuses commands using shell-level background wrappers (nohup, disown, setsid, trailing `&`) when called in foreground mode. The error is:

> "Foreground command uses shell-level background wrappers (nohup/disown/setsid). Use terminal(background=true) so Hermes can track the process, then run readiness checks and tests in separate commands."

**Fix:** use `terminal(background=true, notify_on_complete=true, ...)` for the long-running command. The remote command itself should NOT wrap with `nohup` — Hermes tracks lifecycle for you. For builds and short probes, foreground is fine. For server starts, use `background=true` and poll a readiness endpoint in subsequent foreground calls.

### 2. Don't SSH-then-background from inside an SSH

If you need to start a long-running service on the remote via SSH, do NOT chain `nohup ... &` inside the remote command. The remote's shell exits when SSH returns, and depending on `nohup`'s setup the process may be SIGHUPed. Use `terminal(background=true)` with the SSH command itself as the long-running thing, and have Hermes track the SSH session. The remote command then runs as a normal foreground process under that tracked SSH.

### 3. Don't trust `curl` 200 on `/` as proof

A 200 on the root path can come from a cached static index, a reverse proxy, or a different app. The splash bug is almost always in `/api/*` or a specific route. Always curl the **exact** endpoint the splash polls, with `-v` if the first attempt is suspicious.

### 4. Don't restart blindly

"Restart it" is not a diagnosis. If you don't know why the process died, restarting just delays the next crash. Always:
- Read the last 50 lines of the service's log before restarting.
- Check `journalctl` for the exit code.
- Check `dmesg` for OOM kills.
- Check the supervisor's restart counter (e.g. `claude-bridge.service ... restart counter is at 11487` means it has crashed 11487 times).

### 5. `ps -p <pid>` with no output doesn't mean the process is gone — but it usually does

If `ps -p <pid>` returns the row, the process is alive. If it returns nothing, the process is gone. The output is small and unambiguous; don't over-interpret empty results.

### 6. `ss -tlnp` only shows currently listening sockets

A 0-row result for the port means the listener is dead, not just slow. If you see the port in `ss -tlnp` but `curl` fails, the issue is the proxy, the firewall, or the binding (loopback vs `0.0.0.0`).

### 7. The build changed asset hashes — old browser tabs stay stuck

Every successful build changes the JS/CSS asset hashes (e.g. `main-DQjpdlaH.js` → `main-IpfdxMXe.js`). The HTML preloads the new hashes, but the user's browser has the old HTML cached. After a rebuild, hard refresh is required. If the user reports "still stuck" after a verified fix, it's almost always browser cache.

### 8. Don't expose secrets in `curl -v` or `dmesg`

Some auth endpoints echo the bearer token in the URL. Use `-o /dev/null` to discard bodies when the URL contains a token. `.env` files should be `sed 's/=.*/=<redacted>/'`-ed before display.

### 9. `claude-bridge.service`-style tight crash loops pollute the journal

A service that crashes every 5s (`RestartSec=5`) writes hundreds of journal lines per hour. When you grep `journalctl` for your real symptom, exclude noise. Use `--since "10 min ago"` and `grep -v <known-spammy-svc>`.

### 10. Multiple apps on different ports — don't conflate them

`/etc/nginx/sites-enabled/` may serve different apps on different ports. The "loading screen" the user describes belongs to one of them. `ss -tlnp` + `ls /etc/nginx/sites-enabled/` + `curl -I` on each port is the disambiguation move.

## Backups before edits

```bash
# Always timestamp backups
ssh user@host "cp -a <file> <file>.backup.\$(date +%Y%m%d-%H%M%S)"
```

**Never overwrite without a backup.** The protocol's strict constraint applies more here than anywhere: remote production is not disposable.

## Verification matrix

The fix is "verified" only when all rows are green.

| Check | Command | Pass criteria |
|---|---|---|
| Process alive | `ps -p <pid> -o pid,etime,stat` | row returned, `stat` is `Sl` (sleeping) or `R` (running), not `Z` (zombie) |
| Backend port listening | `ss -tlnp \| grep <port>` | row returned with the right PID |
| Splash polling endpoint | `curl -s http://127.0.0.1:<port>/<splash-path>` | `200` + success JSON in body |
| Public path through proxy | `curl -s http://127.0.0.1:<proxy-port>/<splash-path>` | `200` + same success JSON (no `502`) |
| Readability / shell of root | `curl -s -o /dev/null -w "%{http_code} %{size_download}" http://127.0.0.1:<port>/` | `200`, size > 0 (HTML shell) |
| Gateway / upstream health | `curl -s http://127.0.0.1:<gateway-port>/health` | `200` with `status:ok` |
| Log shows connection | `tail -10 <service>.log` | "Connected to X" or similar boot-complete line |
| Build clean | `tail -10 <build>.log` | `built in N s`, no errors |

## Reporting

When the fix is done, the user needs to know:

1. **Verdict** (Complete / Partial / Failed)
2. **Root cause** (one sentence)
3. **Files changed** (only what you actually wrote; never claim a file was changed if it wasn't)
4. **Commands run** (concrete, paste-ready, ordered)
5. **Services restarted** (named, with PIDs)
6. **Verification results** (the matrix above, with actual outputs)
7. **Remaining risks** (the supervisor gap, browser cache, etc.)
8. **Daily commands** (how to start/stop/status from the user's SSH session)
9. **Confidence score** (1-10; usually 7-9 for a verified remote fix)

Never claim "fixed" if the verification matrix has any red row. "Service is up" is not the same as "splash dismisses" — verify the splash endpoint specifically.

## References

- `references/remote-ssh-recipes.md` — concrete one-liners for the "no project path, no alias, no key hint" situation: probe keys against `known_hosts`, find the project root, build a `ps` + `ss` + `lsof` cross-check, read the project manifest via `jq`, detect whether a supervisor is in place.
- `references/nginx-upstream-debug.md` — when the proxy is the suspect: read `sites-enabled`, test direct vs proxied, interpret `nginx` error log lines, fix `proxy_pass` / `proxy_read_timeout` / WebSocket upgrade gotchas, the loopback-vs-`0.0.0.0` subtlety.
- `references/supervisor-gap-checklist.md` — the post-incident checklist for "the process died and no one restarted it": minimal systemd unit template, the right `Restart=` policy per use case, a `kill -9` + watch-it-come-back verification step, Docker / PM2 / venv variants.
