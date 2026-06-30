---
name: remote-ollama-tunnel
description: "Configure Hermes Desktop to connect to a remote Ollama instance via SSH tunnel. Covers tunnel setup, Hermes config, model context enforcement, and persistence on Windows."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [ollama, ssh, tunnel, vps, hermes, configuration, context-window]
    related_skills: [hermes-agent, agent-os-vps-deployment]
---

# Remote Ollama Tunnel for Hermes

**Class-level skill**: Connect any Hermes Desktop instance to a remote local-model server (Ollama or OpenAI-compatible/vLLM) running on a VPS/GPU host, with SSH/tunnel exposure, Hermes custom-provider config, model context checks, and Windows persistence.

**Not the right skill for:** "I want to talk to Hermes from my phone" / "set up a Telegram bot" / "pair my desktop with a VPS Hermes gateway." Those are `model.base_url` / messaging-platform questions, not model-server questions. See `hermes-vps-deployment` and its `references/phone-messenger-access.md` for the gateway + messaging flow.

## When to Use

- Hermes on local machine (Windows/macOS/Linux)
- Ollama running in Docker on remote VPS, bound to `127.0.0.1:11434` (not public)
- OpenAI-compatible/vLLM endpoint on a remote GPU host (for example Vast.ai) exposed via public port, Cloudflare/Instance Portal tunnel, or SSH tunnel
- Need ≥64K context window for tool use (e.g., `qwen2.5-coder-64k:latest`)
- SSH key auth already configured, or provider API access is available for inspecting instance metadata/logs

## Quick Start

```bash
# 1. Start tunnel
ssh -f -N -L 11434:127.0.0.1:11434 user@your-vps

# 2. Verify tunnel
curl http://localhost:11434/v1/models

# 3. Configure Hermes (use `hermes config` CLI)
hermes config set model.default qwen2.5-coder-64k:latest
hermes config set model.context_length 65536
hermes config set model.ollama_num_ctx 65536
hermes config set model.provider custom
hermes config set model.base_url http://localhost:11434/v1
hermes config set model.api_key ollama
hermes config set custom_providers.0.model qwen2.5-coder-64k:latest

# 4. Force reload model at full context on VPS
ssh user@your-vps "docker exec ollama ollama stop qwen2.5-coder-64k:latest"

# 5. Test
hermes chat -q "Hello" -Q
```

## Prerequisites

| Component | Requirement |
|-----------|-------------|
| Local Hermes | Windows 11 / macOS / Linux |
| Remote Ollama | Docker container, `127.0.0.1:11434` |
| SSH | Key-based auth (ed25519 recommended) — see `references/ssh-key-safety.md` for the safe setup flow and host-key verification procedure |
| Model | Built with `num_ctx 65536` (e.g., `qwen2.5-coder-64k:latest`) |

**CRITICAL — read `references/ssh-key-safety.md` before first connection.** Two failure modes that look routine but are not:
1. **Never ask the user to paste a private SSH key in chat.** Generate the keypair locally, show them only the public part, have them add it to `~/.ssh/authorized_keys` on the VPS.
2. **Never auto-accept a `REMOTE HOST IDENTIFICATION HAS CHANGED` warning.** Cross-check the new fingerprint against the VPS provider's web console before pinning `known_hosts`.

## Step-by-Step Procedure

### 1. Establish SSH Tunnel

```bash
ssh -f -N -L 11434:127.0.0.1:11434 user@vps-host
```

**Verify**:
```bash
curl http://localhost:11434/v1/models
# Must list your target model
```

**Pitfall**: `bind: Address already in use` → Kill existing ssh processes on port 11434:
```powershell
netstat -ano | findstr 11434
taskkill /PID <pid> /F
```

### 2. Configure Hermes (config.yaml)

**Use `hermes config` CLI** — direct file edits are blocked.

```bash
hermes config set model.default qwen2.5-coder-64k:latest
hermes config set model.context_length 65536
hermes config set model.ollama_num_ctx 65536
hermes config set model.provider custom
hermes config set model.base_url http://localhost:11434/v1
hermes config set model.api_key ollama
hermes config set custom_providers.0.model qwen2.5-coder-64k:latest
```

**Required config block**:
```yaml
model:
  default: qwen2.5-coder-64k:latest
  context_length: 65536
  ollama_num_ctx: 65536
  provider: custom
  base_url: http://localhost:11434/v1
  api_key: ollama
custom_providers:
  - name: Local (localhost:11434)
    base_url: http://localhost:11434/v1
    api_key: ollama
    model: qwen2.5-coder-64k:latest
```

**Constraints**:
- Two-space YAML indentation only
- Backup first: `cp config.yaml config.yaml.backup`

### 3. Force Model Reload at Full Context on VPS

```bash
ssh user@vps "docker exec ollama ollama stop qwen2.5-coder-64k:latest"
ssh user@vps "docker exec ollama ollama ps"  # Confirm empty
```

After first Hermes request, verify context:
```bash
ssh user@vps "docker exec ollama ollama ps"
# CONTEXT column must show 65536
```

**Critical**: Current Ollama servers use `OLLAMA_CONTEXT_LENGTH` (not `OLLAMA_NUM_CTX`) to set the default context for `ollama serve` / Docker. For Docker, recreate or run the container with e.g. `-e OLLAMA_CONTEXT_LENGTH=65536` and verify with `docker exec ollama ollama serve --help` and `docker exec ollama env | grep OLLAMA_CONTEXT_LENGTH`.

**Hard cap**: A Modelfile `PARAMETER num_ctx 65536` cannot override GGUF metadata if the model advertises a smaller training context. If logs show `requested context size too large for model num_ctx=65536 n_ctx_train=32768`, Ollama clips to 32768 and `ollama ps` will never show 65536 for that model. Confirm with `ollama show <model> --verbose` (`context length` / `qwen2.context_length`). Fix requires a model/GGUF whose metadata advertises >=64k context (or a different serving stack with the required rope/YaRN support), not just rebuilding the Modelfile.

### 4. Restart Hermes & Test

```bash
# Kill existing Hermes
taskkill /F /IM hermes.exe  # Windows
pkill hermes                # Linux/macOS

# Test
hermes chat -q "Say hello and confirm you can use tools" -Q
# Must NOT show "context window below minimum 64,000" error
```

## Persistence: Windows Task Scheduler

Tunnel dies on reboot. Create logon task:

**PowerShell**:
```powershell
schtasks /create /tn "Ollama Tunnel" /tr "ssh -f -N -L 11434:127.0.0.1:11434 user@vps-host" /sc onlogon /rl highest
```

**Or import XML template**: See `templates/ollama-tunnel-task-scheduler.xml` (replace UserId).

## Verification Script

```bash
./scripts/verify-ollama-tunnel.sh user@vps-host qwen2.5-coder-64k:latest
```

Checks: tunnel connectivity, model listing, explicit 65536 context API call, VPS context column.

## Model Selection for Hermes Tool Mode

Hermes normal operation sends tool schemas. A model can have enough context but still fail if Ollama reports no `tools` capability.

Verify candidate models before configuring Hermes:

```bash
ssh user@vps "docker exec ollama ollama show <model> --verbose | sed -n '1,40p'"
```

Required signs:
- `context length` is at least `65536`
- `Capabilities` includes `tools`

Pitfalls found in practice:
- `codeqwen:7b` has real 64K context and is code-focused, but Ollama reports only `completion`; Hermes tool mode fails with `HTTP 400: ... does not support tools`.
- `qwen3:4b` reports `completion`, `tools`, and `thinking`, with 262K context, and can load at 65,536 context; it is a viable small Hermes-compatible fallback.
- `Mrs_peanutbutt3r/deepseek-r1-coder-tools:7b` reports `tools`, `thinking`, and `completion`, with 131K context, and loads at 65,536 context on a 16GB CPU-only VPS. It passes the basic OpenAI tool-schema probe, but quality can be poor: in one Hermes smoke test it ignored `Reply exactly OK` and hallucinated a GitHub/skills task. Treat community `*-tools` models as candidates to test, not trusted defaults.
- CPU-only VPSes can still time out: Hermes' startup/tool prompt can be ~20k tokens. On a 4-vCPU Hostinger KVM 4, `qwen3:4b` loaded at 65,536 context but took many minutes to prefill the prompt.

Quick API tool-capability probe:

```bash
curl -sS http://localhost:11434/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"<model>","messages":[{"role":"user","content":"Reply exactly OK"}],"max_tokens":5,"tools":[{"type":"function","function":{"name":"noop","description":"do nothing","parameters":{"type":"object","properties":{}}}}]}'
```

## OpenAI-Compatible / vLLM Endpoints on Vast.ai

For Vast.ai or similar GPU-hosted vLLM deployments, see `references/vastai-vllm-openai-compatible.md` before editing Hermes config. Key workflow: save the provider API key as an environment variable, inspect instance metadata/logs, identify the actual internal vLLM port, then verify `curl -i <base>/v1/models` reaches vLLM without Caddy/Basic Auth before blaming Hermes.

Recommended Hermes `providers:` shape for model-picker visibility:

```yaml
model:
  default: <model-id>
  provider: openai-compatible
  base_url: https://<tunnel-or-host>/v1
  api_key: none
providers:
  openai-compatible:
    name: OpenAI Compatible
    api: https://<tunnel-or-host>/v1
    api_key: none
    default_model: <model-id>
    transport: chat_completions
```

Use the unprefixed provider key (e.g. `openai-compatible`) in `model.provider` for the picker row; `custom:<name>` may resolve at runtime but can confuse “I don’t see it in Models” checks.

## Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| "context window below minimum 64,000" | Model loaded at 32768 | Set `OLLAMA_CONTEXT_LENGTH=65536`, restart/recreate Ollama, and verify model metadata supports >=64k |
| `requested context size too large ... n_ctx_train=32768` in logs | GGUF metadata hard-caps context despite Modelfile `num_ctx` | Use a corrected >=64k/128k GGUF/model or different serving stack; Modelfile rebuild alone is insufficient |
| `HTTP 400: ... does not support tools` from Hermes | Model lacks Ollama `tools` capability despite enough context | Choose a model whose `ollama show --verbose` capabilities include `tools`; `codeqwen:7b` is not enough for normal Hermes tool mode |
| Tunnel works but model not listed | Wrong model name / not pulled | `docker exec ollama ollama pull <model>` |
| Hermes times out | Model loading or CPU-only prompt eval is too slow | Check `docker logs ollama`; even a 4B CPU model can take many minutes to process Hermes' ~20k-token prompt |
| Port 11434 in use | Old tunnel / local Ollama | Kill process, or change local port + update config |

## Support Files

| File | Purpose |
|------|---------|
| `references/hermes-ollama-vps-tunnel.md` | Detailed session notes, commands, troubleshooting |
| `references/hostinger-kvm4-local-hermes-models.md` | Hostinger KVM 4 CPU-only model findings for Hermes: `codeqwen:7b`, `qwen3:4b`, and DeepSeek coder-tools results |
| `references/vastai-vllm-openai-compatible.md` | Vast.ai/vLLM OpenAI-compatible endpoint setup: provider shape, model-picker slug, port/tunnel inspection, and Basic Auth/Caddy pitfalls |
| `references/ssh-key-safety.md` | **Read before first connection.** Safe agent→VPS key flow, host-key verification, terminology precision (public key vs. fingerprint), what-not-to-do |
| `scripts/verify-ollama-tunnel.sh` | Automated verification (tunnel + model + context) |
| `templates/ollama-tunnel-task-scheduler.xml` | Windows Task Scheduler import for persistent tunnel |

## Related Skills

- `hermes-agent` — Hermes configuration, CLI, providers
- `agent-os-vps-deployment` — VPS setup for agent workloads