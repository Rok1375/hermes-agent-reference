# hermes-agent-reference

> Auto-generated reference index of all **99 bundled skills** and **20 protocol/agent instruction files** that ship with Hermes Agent on this machine.

This repo is a faithful, machine-readable mirror of:

- `C:\Users\soren\AppData\Local\hermes\skills\` — the 99 skill `SKILL.md` files across 36 categories
- `C:\Users\soren\.agent-library\active\` — 9 global protocol files (boot, preflight, routing, verification, fable, swarm, handoff, kanban)
- `C:\Users\soren\.agent-library\agents\*\AGENTS.md` — 4 agent-specific instructions (generic, hermes, kilo, opencode)
- `C:\Users\soren\.agents\skills\*\AGENTS.md` — 5 third-party skill packages + 1 nested loki-mode reference
- `C:\Users\soren\AGENTS.md` — 1 home-level instruction

Generated on **2026-06-30** by Hermes Agent (model: MiniMax-M3 / provider: opencode-go).

## What's in here

```
.
├── README.md                  ← you are here
├── SKILLS.md                  ← master index of all 99 bundled skills (by category)
├── AGENTS.md                  ← master index of all 20 protocol/agent instruction files
├── LICENSE                    ← MIT
├── AGENTS/                    ← verbatim copies of the 20 instruction files
│   ├── .agent-library/active/ ← 9 global protocol files
│   ├── .agent-library/agents/ ← 4 agent variants
│   ├── .agents/skills/        ← 5 third-party skill packages + 1 loki-mode reference
│   │   ├── dbos-golang/AGENTS.md
│   │   ├── dbos-python/AGENTS.md
│   │   ├── dbos-typescript/AGENTS.md
│   │   ├── postgres-best-practices/AGENTS.md
│   │   ├── react-best-practices/AGENTS.md
│   │   └── loki-mode/references/AGENTS.md
│   └── AGENTS.md              ← home-level instructions
└── docs/
    ├── categories/            ← 36 per-category deep-dive docs (full SKILL.md bodies)
    │   ├── README.md          ← categories index
    │   ├── core.md
    │   ├── autonomous-ai-agents.md
    │   └── ...                ← one .md per category
    └── skills/                ← 99 per-skill full-text dumps
        ├── core/
        │   ├── self-improvement-memory.md
        │   └── skill-router.md
        └── ...                ← one .md per skill, organized by category
```

## How to navigate

1. **Start at [`SKILLS.md`](./SKILLS.md)** — full table of all 99 skills with descriptions, versions, and tags, grouped by category.
2. **Drill into a category** — pick a category from `SKILLS.md` and open `docs/categories/<category>.md` for the full text of every skill in that category.
3. **Or browse single skills** — `docs/skills/<category>/<skill>.md` has the original `SKILL.md` body.
4. **For protocol/agent docs** — start at [`AGENTS.md`](./AGENTS.md) and follow links into `AGENTS/`.

## Regenerating

This repo is meant to be regenerated whenever the local Hermes installation changes. To refresh:

1. Re-run the inventory script (extracted from the generation session) against the same source paths.
2. Re-render `SKILLS.md`, `AGENTS.md`, `docs/categories/*`, `docs/skills/*`.
3. Commit and push.

## About the source

Hermes Agent by Nous Research. Model: MiniMax-M3 (MiniMax, founded early 2022, AGI-focused).
This reference is a personal documentation artifact, not an official Nous Research product.

---

**Note on the credential used to push this repo:** the GitHub PAT used to create and push this repo (`ghp_…`) was used once for the API and should be **revoked** by the owner (https://github.com/settings/tokens) after the push completes. It is **not** stored anywhere in this repo.
