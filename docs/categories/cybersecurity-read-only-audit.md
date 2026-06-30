# Category: cybersecurity-read-only-audit

> Deep-dive for the **1 skill** in the `cybersecurity-read-only-audit` category. Each section embeds the full `SKILL.md` body so this doc is self-contained.

**Skills in this category:**

- [`cybersecurity-read-only-audit`](../skills/cybersecurity-read-only-audit/cybersecurity-read-only-audit.md) - Read-only exposure audit of the user's machine and projects for a CVE, breach, malicious package, or other security advi

---

## cybersecurity-read-only-audit

- **Name:** `cybersecurity-read-only-audit`
- **Version:** 1.0.0
- **Author:** Hermes Agent
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/cybersecurity-read-only-audit/SKILL.md`

**Description:**

> Read-only exposure audit of the user's machine and projects for a CVE, breach, malicious package, or other security advisory, then write a structured report. Use when the user shares a breach / CVE / malware / supply-chain advisory and asks "am I affected", "scan my system for X", "are we vulnerable to Y".

**Full SKILL.md body:**

```markdown
---
name: cybersecurity-read-only-audit
description: Read-only exposure audit of the user's machine and projects for a CVE, breach, malicious package, or other security advisory, then write a structured report. Use when the user shares a breach / CVE / malware / supply-chain advisory and asks "am I affected", "scan my system for X", "are we vulnerable to Y".
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [security, audit, CVE, vulnerability, read-only, supply-chain]
---

# Cybersecurity Read-Only Audit

Run a read-only exposure check against a CVE, breach, malicious package, or security advisory, and produce a structured verdict report.

## Hard rules

- **Read-only.** No installs, removes, upgrades, restarts, network calls, or file modifications outside the local audit folder.
- **No `sudo`.** Never.
- **One report per invocation.** Always end by writing the `.md` file (even when verdict is "Not affected" — the audit trail matters).
- If a check requires a state-changing command, **skip it** and note "not checked (would require state change)" in the table. Do not run it.

## Workflow

1. **Identify scope.** Extract from the advisory: package/binary name, affected versions, platform, attack vector (supply chain / RCE / local / network).
2. **Run checks in parallel** — issue multiple tool calls in one message. Pick checks relevant to the advisory type; don't run all of them.
3. **Build the table** as you go. Each row = one check + concrete result (version, path, "None", "N/A").
4. **Write the report** to `~/Documents/cyber/YYYY-MM-DD-<short-kebab-slug>.md` (or the user's configured audit folder). Use today's date from the environment.
5. **Tell the user** the verdict in one line plus path to the report.

## Check menu (pick what's relevant)

```bash
# --- Node / npm ecosystem (supply-chain advisories) ---
which npm pnpm yarn; npm root -g; pnpm root -g 2>/dev/null
ls /opt/homebrew/lib/node_modules                                  # global npm
find ~ -maxdepth 8 -type d -name "<pkg>" 2>/dev/null \
  | grep -v -E "(Library/Caches|\.Trash)"                          # installed copies
find <work-dirs> -maxdepth 8 -type f \
  \( -name "package.json" -o -name "package-lock.json" \
     -o -name "pnpm-lock.yaml" -o -name "yarn.lock" \) 2>/dev/null \
  | xargs grep -l "<pkg>" 2>/dev/null                              # direct + transitive

# --- Python ecosystem ---
which python3 pip pipx uv
pip list 2>/dev/null | grep -i "<pkg>"
find <work-dirs> -maxdepth 6 \( -name "requirements*.txt" -o -name "pyproject.toml" \
  -o -name "poetry.lock" -o -name "uv.lock" \) 2>/dev/null | xargs grep -l "<pkg>" 2>/dev/null

# --- Homebrew / system binaries (macOS) ---
brew list --versions <formula> 2>/dev/null
which <binary>; <binary> --version 2>/dev/null

# --- Running processes / listeners (RCE / network CVEs) ---
pgrep -lf "<binary>"
lsof -iTCP -sTCP:LISTEN -P -n 2>/dev/null | grep "<port>"

# --- Env vars that change exposure ---
grep -r "<VAR>" ~/.zshrc ~/.zprofile ~/.config 2>/dev/null

# --- Editor / browser extensions (IDE-targeted advisories) ---
ls ~/.vscode/extensions 2>/dev/null | grep -i "<ext>"
```

If the advisory mentions an ecosystem not above (Rust cargo, Go modules, Ruby gems, Docker images), apply the same pattern: global install path + manifest grep + running processes.

## Report template

File: `~/Documents/cyber/YYYY-MM-DD-<short-kebab-slug>.md`

```markdown
# <Subject> — Audit

**Date:** YYYY-MM-DD
**Host:** <machine name and OS>

## <CVEs | Advisory> in scope

- **<ID or source> "<Name>"** — <one-line description>. <Affected versions or scope>.

## Audit results

| Check | Result |
|---|---|
| <Check 1> | <Result> |
| <Check 2> | <Result> |

## Verdict

**<Not affected. | Affected. | Partially affected.>**

- <Rationale bullet 1>
- <Rationale bullet 2>

## Action taken

None — diagnostic only, no files modified, no <packages installed/removed | services started/stopped | firewall rules changed>.

## Follow-ups

- <Actionable item, or "None" if truly nothing>
```

Match the tone of existing reports in the audit folder — terse, factual, bulleted, no hedging.

## Verdict wording

- **Not affected.** — package/binary absent, or installed but patched, or not running and not exposed.
- **Affected.** — vulnerable version present *and* reachable by the attack vector.
- **Partially affected.** — present but mitigated (installed but service not running, listener bound to loopback only). Spell out the mitigation in the bullets.

## When to break the read-only rule

Never on your own. If the verdict is "Affected", list the remediation command in **Follow-ups** and stop. The user runs it.

## Pitfalls

- **Don't write the report without running checks.** Empty results tables = fabrication.
- **Don't extrapolate "absent here" → "absent everywhere."** Search the work directories the user actually uses.
- **Don't run remediation without explicit authorization.** Always Follow-ups only.
- **Don't omit the date or host.** The audit trail is the point.
- **Don't pad the verdict.** One short paragraph, then bullets.

```

---
