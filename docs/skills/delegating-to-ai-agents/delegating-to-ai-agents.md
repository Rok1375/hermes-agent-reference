---
name: delegating-to-ai-agents
description: Generic patterns for sending prompts to and polling output from other AI agents (Codex CLI, Claude Code, Pi Agent, OpenCode, Hermes, or any other autonomous coding CLI). Use when delegating work to another agent, polling for output, or orchestrating agent-to-agent work. Covers TUI single-line prompt rules, short polling cadence, interactive-CLI flags, and the SSH-then-launch anti-pattern for remote machines.
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [agents, orchestration, delegation, polling, TUI, multi-agent]
---

# Delegating to AI Agents

Patterns for reliable communication with autonomous AI agents running as separate processes. Tool-specific paths are generalized; only the durable behavioral rules are kept.

## Polling cadence — keep sleeps SHORT

When waiting on another agent, use short `sleep` intervals so you check often. Don't `sleep 30`. Start with `sleep 3-5`; if the agent isn't done, `sleep 5` again and re-check. Scale up only for genuinely heavy tasks (large build, long refactor).

Fast-streaming models (e.g. opus-fast, gpt-4-class models at xhigh reasoning) usually respond in seconds. A 30-second wait is almost always wasted.

## Sending prompts — NEVER put newlines in the message body

When sending a prompt to a TUI agent (Codex, Claude Code, Pi, OpenCode, Hermes) via a send-keys equivalent (`cmux send`, `tmux send-keys`, etc.), the message text **must be a single line**. In these TUIs a newline = Enter, so a multi-line string submits after the first line and the rest arrive as separate mid-turn "steering" messages — cutting off / fragmenting your prompt.

Fixes (pick one):
- Send the whole prompt as **one line** (use `". "` or `"; "` instead of line breaks), then one explicit `send-key enter`.
- For long / multi-step instructions, **write them to a file** and tell the agent to read it: `send '<agent> read /tmp/task.md and follow it'`, then `send-key enter`.

## Driving interactive CLIs

Different agents have different invocation needs. Pick the right mode:

| Agent | Default mode | Notes |
|---|---|---|
| **Codex CLI** | Interactive TUI (needs `pty=true` when driven programmatically) | Native kernel-level sandbox; strong for `codex exec` in CI. Reads `AGENTS.md`. |
| **Claude Code** | `--print --permission-mode bypassPermissions` (no PTY) | Reads `CLAUDE.md` / `.claude/`. Skills hot-reload mid-session. |
| **Pi Agent** | TUI (needs `pty=true`) | Minimal 4-tool core; self-extends via TS extensions and skills. Fast startup (1-2s). |
| **OpenCode** | TUI (needs `pty=true`) | Multi-provider, plan/agent modes. |
| **Hermes** | Python autonomous agent | Persistent self-improving agent; can *drive* other coding CLIs as a subagent. |

**Rule of thumb:** if the agent has a TUI, drive it through a PTY. If it has a `--print` mode, prefer that — no PTY needed.

## Managing a remote machine via an agent

To have an agent manage a remote VPS / server, **SSH in first and launch the agent ON the box** (e.g. `ssh user@host` then `codex --yolo` or `claude --print`), then drive that on-box agent. It has full local context and avoids fragile per-command SSH round-trips.

**Anti-pattern:** running an agent locally and telling it to SSH for every step. Each SSH is a context boundary, a credential re-entry, a timeout risk, and a chance for the agent to mis-handle interactive prompts. The aggregate cost is usually worse than just doing the work yourself.

## Auth and credentials

- **Never invent a key.** If an env var is missing, check if it's normally loaded from a shell profile. If still unset, ask the user — do not generate a fake value.
- **API-key auth vs subscription auth** differ per agent. Codex `/goal`-style features often require subscription auth, not API keys. Confirm the right one before launching.
- **Resume strips sensitive env vars** in some agents. Re-inject tokens at resume time if the agent needs them.

## The four (or more) agent model

Different agents optimize for different things. Quick reference:

- **For most coding tasks** → Codex CLI. Strongest on long-running SWE work, kernel-level sandbox, fastest startup.
- **For frontend / design / quick iteration** → Pi Agent or Claude Code. Tighter feedback loop on visual work.
- **For orchestration / persistent memory** → Hermes. Cross-session memory, cron, subagent delegation. Drives other agents as workers.
- **For plan-heavy / multi-provider** → OpenCode. Plan/agent modes, broad model support.

A good default for complex work: persistent agent (Hermes) as orchestrator, delegating execution to a coding CLI (Codex) running in a sibling terminal or pane.

## Hard rules

- **Never append `2>/dev/null` to agent CLI commands.** Errors go to stderr with exit code 1; suppressing them blinds you to your own ref/flag mistakes.
- **Always re-list handles / IDs before reusing them.** A ref from a previous turn may not still be valid.
- **Don't send input to surfaces you don't own.** Only target handles in the caller's workspace unless the user explicitly asks for cross-workspace routing.
- **Prefer `--print` / non-interactive modes in CI.** Reserve TUI / PTY for interactive use.
- **Pin to a specific version** of any agent CLI in production workflows; `latest` breaks.

## Pitfalls

- **TUI newlines in prompts.** The #1 cause of "agent is doing something weird." Always one line.
- **Long polling intervals.** Wastes wall-clock and CPU; switch to `sleep 3-5` with re-check.
- **Wrong flags for the agent.** `pty=true` for Codex / Pi / OpenCode; `--print --permission-mode bypassPermissions` for Claude Code.
- **Per-command SSH round-trips** to a remote box. SSH in once, launch the agent on the box, drive it locally.
- **Inventing an API key** when the env var is missing. Always ask.
- **Sending input to the wrong surface / pane / session.** Always re-list and anchor to the correct ref.
- **Letting a "run forever" agent loop drift** without a verifiable stop condition. Use the 4-part contract pattern (see `codex-goal-prompt-authoring` skill).
