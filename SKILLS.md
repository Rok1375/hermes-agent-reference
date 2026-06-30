# Hermes Agent - Skills Reference

> Auto-generated reference index of all **99 bundled skills** shipped with Hermes Agent,
> organized by category. Each entry links to a deeper per-category doc in this repo and to
> the on-disk `SKILL.md` source it was extracted from.

**Source machine:** Windows 10 (`C:/Users/soren`)  
**Source path:** `C:/Users/soren/AppData/Local/hermes/skills/`  
**Generated:** 2026-06-30  
**Total skills:** 99 across **36 categories**

---

## Table of contents

- [core (2 skills)](#core)
- [autonomous-ai-agents (7 skills)](#autonomous-ai-agents)
- [software-development (10 skills)](#software-development)
- [github (6 skills)](#github)
- [devops (2 skills)](#devops)
- [dev (3 skills)](#dev)
- [mlops (7 skills)](#mlops)
- [data-science (1 skill)](#data-science)
- [productivity (9 skills)](#productivity)
- [research (5 skills)](#research)
- [creative (16 skills)](#creative)
- [media (4 skills)](#media)
- [email (1 skill)](#email)
- [note-taking (1 skill)](#note-taking)
- [smart-home (1 skill)](#smart-home)
- [social-media (1 skill)](#social-media)
- [apple (4 skills)](#apple)
- [fable-mode (1 skill)](#fable-mode)
- [prism-3way (1 skill)](#prism-3way)
- [prism-discover (1 skill)](#prism-discover)
- [prism-full (1 skill)](#prism-full)
- [prism-reflect (1 skill)](#prism-reflect)
- [prism-scan (1 skill)](#prism-scan)
- [hermes-council-vps (1 skill)](#hermes-council-vps)
- [agent-skill-authoring (1 skill)](#agent-skill-authoring)
- [codex-goal-prompt-authoring (1 skill)](#codex-goal-prompt-authoring)
- [computer-use (1 skill)](#computer-use)
- [cybersecurity-read-only-audit (1 skill)](#cybersecurity-read-only-audit)
- [deep-research-prompt (1 skill)](#deep-research-prompt)
- [delegating-to-ai-agents (1 skill)](#delegating-to-ai-agents)
- [dogfood (1 skill)](#dogfood)
- [interview-prep-helper (1 skill)](#interview-prep-helper)
- [remote-service-incident-debugging (1 skill)](#remote-service-incident-debugging)
- [video-preproduction (1 skill)](#video-preproduction)
- [video-presentation-review (1 skill)](#video-presentation-review)
- [yuanbao (1 skill)](#yuanbao)

---

## core

**2 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `self-improvement-memory` | 1.0.0 | After completing any task, records what worked and what failed into persistent memory and proposes targeted updates to existing skills. The agent's own improvement loop — never repeat the same mistake twice. | - |
| 2 | `skill-router` | 1.0.0 | Matches incoming requests to the most relevant Hermes skill and loads it automatically. Acts as the gateway between user intent and procedural knowledge. Runs before every task to ensure the right skill is active. | - |

## autonomous-ai-agents

**7 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `claude-code` | 2.2.0 | Delegate coding to Claude Code CLI (features, PRs). | - |
| 2 | `codex` | 1.0.0 | Delegate coding to OpenAI Codex CLI (features, PRs). | - |
| 3 | `council-orchestrator` | - | Multi-agent blackboard deliberation - shared /tmp/council/<id>/board.md, 3 rounds of draft/critique/vote, convergence at >=2/3 VOTE, chair synthesizes final.md. Works with any installed CLI agents. | - |
| 4 | `hermes-agent` | 2.3.0 | Configure, extend, or contribute to Hermes Agent. | - |
| 5 | `hermes-subagent-orchestrator` | 1.0.0 | Use when a task is complex and benefits from splitting into focused subagents: multi-step work, large coding tasks, debugging, repo analysis, design/motion systems, or research-heavy workflows. | - |
| 6 | `opencode` | 1.2.0 | Delegate coding to OpenCode CLI (features, PR review). | - |
| 7 | `self-improving-subagent-router` | 1.0.0 | Use before any task to classify, route to subagents, and after completion to reflect and improve skills. The self-improvement loop for Hermes subagent orchestration. | - |

## software-development

**10 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `adopt-external-tools` | 1.1.0 | Safely integrate external agent tools, skill repos, MCP servers, harnesses, and frameworks into Hermes without clobbering existing configuration, identity files, or memory rules. | - |
| 2 | `hermes-agent-skill-authoring` | 1.1.0 | Author in-repo SKILL.md: frontmatter, validator, structure, and writing-quality principles. | - |
| 3 | `node-inspect-debugger` | 1.0.0 | Debug Node.js via --inspect + Chrome DevTools Protocol CLI. | - |
| 4 | `plan` | 2.0.0 | Plan mode: write an actionable markdown plan to .hermes/plans/, no execution. Bite-sized tasks, exact paths, complete code. | - |
| 5 | `python-debugpy` | 1.0.0 | Debug Python: pdb REPL + debugpy remote (DAP). | - |
| 6 | `requesting-code-review` | 2.0.0 | Pre-commit review: security scan, quality gates, auto-fix. | - |
| 7 | `simplify-code` | 1.0.0 | Parallel 3-agent cleanup of recent code changes. | - |
| 8 | `spike` | 1.0.0 | Throwaway experiments to validate an idea before build. | - |
| 9 | `systematic-debugging` | 1.1.0 | 4-phase root cause debugging: understand bugs before fixing. | - |
| 10 | `test-driven-development` | 1.1.0 | TDD: enforce RED-GREEN-REFACTOR, tests before code. | - |

## github

**6 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `codebase-inspection` | 1.0.0 | Inspect codebases w/ pygount: LOC, languages, ratios. | - |
| 2 | `github-auth` | 1.1.0 | GitHub auth setup: HTTPS tokens, SSH keys, gh CLI login. | - |
| 3 | `github-code-review` | 1.1.0 | Review PRs: diffs, inline comments via gh or REST. | - |
| 4 | `github-issues` | 1.1.0 | Create, triage, label, assign GitHub issues via gh or REST. | - |
| 5 | `github-pr-workflow` | 1.1.0 | GitHub PR lifecycle: branch, commit, open, CI, merge. | - |
| 6 | `github-repo-management` | 1.1.0 | Clone/create/fork repos; manage remotes, releases. | - |

## devops

**2 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `hermes-vps-deployment` | - | Deploy Hermes Agent on a Linux VPS so you can talk to it from anywhere — phone via Telegram/Discord/WhatsApp, laptop via SSH tunnel, or paired with the Hermes Desktop GUI. Covers Docker Compose setup for the gateway + messaging platforms, common crash patterns, network/tunnel exposure, and end-to-end verification. Use when the user says "talk to Hermes from my phone", "remote gateway", "deploy Hermes on a VPS", "Telegram/Discord bot on my VPS", "Hermes desktop + VPS", or "I have a VPS, set up Hermes on it". | - |
| 2 | `remote-ollama-tunnel` | 1.0.0 | Configure Hermes Desktop to connect to a remote Ollama instance via SSH tunnel. Covers tunnel setup, Hermes config, model context enforcement, and persistence on Windows. | - |

## dev

**3 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `agent-os-vps-deployment` | - | Deploy OpenCloud Agent OS (multi-agent orchestration system) to a VPS with working parallel and swarm execution engines. | - |
| 2 | `github-skill-import` | 1.0.0 | Clone a GitHub repo containing Hermes skills (or skill documentation), read specified files, extract the SKILL.md schema/frontmatter format, and create new skills following that schema. End-to-end workflow for importing external skill patterns. | - |
| 3 | `token-optimizer` | - | Find the ghost tokens. Audit Claude Code or Codex setup, see where context goes, fix it. Use when context feels tight. | - |

## mlops

**7 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `audiocraft-audio-generation` | 1.0.0 | AudioCraft: MusicGen text-to-music, AudioGen text-to-sound. | - |
| 2 | `evaluating-llms-harness` | 1.0.0 | lm-eval-harness: benchmark LLMs (MMLU, GSM8K, etc.). | - |
| 3 | `huggingface-hub` | 1.0.0 | HuggingFace hf CLI: search/download/upload models, datasets. | huggingface,  hf,  models,  datasets,  hub,  mlops |
| 4 | `llama-cpp` | 2.1.2 | llama.cpp local GGUF inference + HF Hub model discovery. | - |
| 5 | `segment-anything-model` | 1.0.0 | SAM: zero-shot image segmentation via points, boxes, masks. | - |
| 6 | `serving-llms-vllm` | 1.0.0 | vLLM: high-throughput LLM serving, OpenAI API, quantization. | - |
| 7 | `weights-and-biases` | 1.0.0 | W&B: log ML experiments, sweeps, model registry, dashboards. | - |

## data-science

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `jupyter-live-kernel` | 1.0.0 | Iterative Python via live Jupyter kernel (hamelnb). | - |

## productivity

**9 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `airtable` | 1.1.0 | Airtable REST API via curl. Records CRUD, filters, upserts. | - |
| 2 | `google-workspace` | 1.1.0 | Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python. | - |
| 3 | `maps` | 1.2.0 | Geocode, POIs, routes, timezones via OpenStreetMap/OSRM. | - |
| 4 | `nano-pdf` | 1.0.0 | Edit PDF text/typos/titles via nano-pdf CLI (NL prompts). | - |
| 5 | `notion` | 2.0.0 | Notion API + ntn CLI: pages, databases, markdown, Workers. | - |
| 6 | `ocr-and-documents` | 2.3.0 | Extract text from PDFs/scans (pymupdf, marker-pdf). | - |
| 7 | `petdex` | 1.0.0 | Install and select animated petdex mascots for Hermes. | - |
| 8 | `powerpoint` | - | Create, read, edit .pptx decks, slides, notes, templates. | - |
| 9 | `teams-meeting-pipeline` | 1.1.0 | Operate the Teams meeting summary pipeline via Hermes CLI — summarize meetings, inspect pipeline status, replay jobs, manage Microsoft Graph subscriptions. | - |

## research

**5 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `arxiv` | 1.0.0 | Search arXiv papers by keyword, author, category, or ID. | - |
| 2 | `blogwatcher` | 2.0.0 | Monitor blogs and RSS/Atom feeds via blogwatcher-cli tool. | - |
| 3 | `llm-wiki` | 2.1.0 | Karpathy's LLM Wiki: build/query interlinked markdown KB. | - |
| 4 | `polymarket` | 1.0.0 | Query Polymarket: markets, prices, orderbooks, history. | polymarket,  prediction-markets,  market-data,  trading |
| 5 | `research-paper-writing` | 1.1.0 | Write ML papers for NeurIPS/ICML/ICLR: design→submit. | - |

## creative

**16 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `architecture-diagram` | 1.0.0 | Dark-themed SVG architecture/cloud/infra diagrams as HTML. | - |
| 2 | `ascii-art` | 4.0.0 | ASCII art: pyfiglet, cowsay, boxes, image-to-ascii. | - |
| 3 | `ascii-video` | - | ASCII video: convert video/audio to colored ASCII MP4/GIF. | - |
| 4 | `baoyu-infographic` | 1.56.1 | Infographics: 21 layouts x 21 styles (信息图, 可视化). | - |
| 5 | `claude-design` | 1.0.0 | Design one-off HTML artifacts (landing, deck, prototype). | - |
| 6 | `comfyui` | 5.1.0 | Generate images, video, and audio with ComfyUI — install, launch, manage nodes/models, run workflows with parameter injection. Uses the official comfy-cli for lifecycle and direct REST/WebSocket API for execution. | - |
| 7 | `design-md` | 1.0.0 | Author/validate/export Google's DESIGN.md token spec files. | - |
| 8 | `excalidraw` | 1.0.0 | Hand-drawn Excalidraw JSON diagrams (arch, flow, seq). | - |
| 9 | `humanizer` | 2.5.1 | Humanize text: strip AI-isms and add real voice. | - |
| 10 | `manim-video` | 1.0.0 | Manim CE animations: 3Blue1Brown math/algo videos. | - |
| 11 | `p5js` | 1.0.0 | p5.js sketches: gen art, shaders, interactive, 3D. | - |
| 12 | `popular-web-designs` | 1.0.0 | 54 real design systems (Stripe, Linear, Vercel) as HTML/CSS. | design,  css,  html,  ui,  web-development,  design-systems,  templates |
| 13 | `pretext` | 1.0.0 | Use when building creative browser demos with @chenglou/pretext — DOM-free text layout for ASCII art, typographic flow around obstacles, text-as-geometry games, kinetic typography, and text-powered generative art. Produces single-file HTML demos by default. | - |
| 14 | `sketch` | 1.0.0 | Throwaway HTML mockups: 2-3 design variants to compare. | - |
| 15 | `songwriting-and-ai-music` | - | Songwriting craft and Suno AI music prompts. | songwriting,  music,  suno,  parody,  lyrics,  creative |
| 16 | `touchdesigner-mcp` | 1.1.0 | Control a running TouchDesigner instance via twozero MCP — create operators, set parameters, wire connections, execute Python, build real-time visuals. 36 native tools. | - |

## media

**4 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `gif-search` | 1.1.0 | Search/download GIFs from Tenor via curl + jq. | - |
| 2 | `heartmula` | 1.0.0 | HeartMuLa: Suno-like song generation from lyrics + tags. | - |
| 3 | `songsee` | 1.0.0 | Audio spectrograms/features (mel, chroma, MFCC) via CLI. | - |
| 4 | `youtube-content` | - | YouTube transcripts to summaries, threads, blogs. | - |

## email

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `himalaya` | 1.1.0 | Himalaya CLI: IMAP/SMTP email from terminal. | - |

## note-taking

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `obsidian` | - | Read, search, create, and edit notes in the Obsidian vault. | - |

## smart-home

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `openhue` | 1.0.0 | Control Philips Hue lights, scenes, rooms via OpenHue CLI. | - |

## social-media

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `xurl` | 1.1.1 | X/Twitter via xurl CLI: post, search, DM, media, v2 API. | - |

## apple

**4 skills** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `apple-notes` | 1.0.0 | Manage Apple Notes via memo CLI: create, search, edit. | - |
| 2 | `apple-reminders` | 1.0.0 | Apple Reminders via remindctl: add, list, complete. | - |
| 3 | `findmy` | 1.0.0 | Track Apple devices/AirTags via FindMy.app on macOS. | - |
| 4 | `imessage` | 1.0.0 | Send and receive iMessages/SMS via the imsg CLI on macOS. | - |

## fable-mode

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `fable-mode` | 1.0 | Fable-style workflow discipline for all agent operations with user preference embedding | workflow,  discipline,  fable,  code-review,  tdd |

## prism-3way

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `prism-3way` | - | Three orthogonal analytical operations (WHERE/WHEN/WHY) + cross-operation synthesis. Each operation attacks the problem from a fundamentally different angle. The disagreements between the three ARE the valuable output. Works on any domain — code, business, strategy, design, text. | - |

## prism-discover

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `prism-discover` | - | Discover all possible analysis domains for an artifact. Finds obvious and non-obvious angles — architecture, security, but also marketing positioning, user psychology, regulatory implications, teaching value. Use before prism-scan or prism-full to explore what's worth investigating. | - |

## prism-full

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `prism-full` | - | Full Prism: multi-pass structural analysis with mandatory adversarial self-correction. Designs custom analytical passes, executes them with chaining, then attacks its own findings before synthesizing. Use for maximum depth on important code or artifacts. | - |

## prism-reflect

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `prism-reflect` | - | Constraint transparency: analyzes an artifact structurally, then analyzes what its own analysis concealed. Produces a conservation law AND a constraint report showing what was maximized, what was sacrificed, and what to investigate next. The only AI skill that knows what it can't see. | - |

## prism-scan

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `prism-scan` | - | Structural analysis through dynamically generated cognitive lenses. Generates the optimal analytical lens for the specific code/artifact, then executes it. Finds conservation laws, structural invariants, and concrete bugs that vanilla analysis misses. Use on any code file, system design, or text artifact. | - |

## hermes-council-vps

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `hermes-council-vps` | - | Run a multi-agent LLM council on a Hetzner Ubuntu VPS using local Ollama models (no API key needed) plus optional OpenRouter/OpenAI/Anthropic providers via the `council` CLI. Use when the user wants a free, self-hosted multi-agent deliberation system or wants to set up the `council` orchestrator on a fresh VPS. | - |

## agent-skill-authoring

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `agent-skill-authoring` | 1.0.0 | How to write effective agent skills (SKILL.md) — anatomy, progressive disclosure, description-routing contract, design patterns, anti-patterns, ship checklist. Use when the user says "create a skill", "improve this skill", "why isn't my skill triggering", or is editing a SKILL.md file. | - |

## codex-goal-prompt-authoring

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `codex-goal-prompt-authoring` | 1.0.0 | Explain and write effective instructions for persistent self-checking agent loops (Codex /goal, or analogous long-running autonomous task loops with auto-continue). Use when the user mentions a long-running autonomous agent run, a goal loop, wants to kick off an overnight job, or asks how to write a goal prompt. | - |

## computer-use

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `computer-use` | 2.0.0 | \| | - |

## cybersecurity-read-only-audit

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `cybersecurity-read-only-audit` | 1.0.0 | Read-only exposure audit of the user's machine and projects for a CVE, breach, malicious package, or other security advisory, then write a structured report. Use when the user shares a breach / CVE / malware / supply-chain advisory and asks "am I affected", "scan my system for X", "are we vulnerable to Y". | - |

## deep-research-prompt

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `deep-research-prompt` | 1.0.0 | Write a single-paragraph Deep Research prompt to hand to a human researcher or a deep-research AI. Use when the user wants a research brief, a "deep research prompt", a one-paragraph task for a researcher, or asks "what should our researcher look for". Produces ONE tight paragraph with full context, numbered sub-questions, and per-finding output format. | - |

## delegating-to-ai-agents

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `delegating-to-ai-agents` | 1.0.0 | Generic patterns for sending prompts to and polling output from other AI agents (Codex CLI, Claude Code, Pi Agent, OpenCode, Hermes, or any other autonomous coding CLI). Use when delegating work to another agent, polling for output, or orchestrating agent-to-agent work. Covers TUI single-line prompt rules, short polling cadence, interactive-CLI flags, and the SSH-then-launch anti-pattern for remote machines. | - |

## dogfood

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `dogfood` | 1.0.0 | Exploratory QA of web apps: find bugs, evidence, reports. | - |

## interview-prep-helper

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `interview-prep-helper` | 1.0.0 | Prepare for and run a structured job-candidate interview. Parses the latest candidate application, builds a TLDR with flags, generates 5-6 sharp custom questions, tracks asked questions + live notes during the call, and runs a post-interview debrief. Use when the user says "interview prep", "parse the candidate", "build interview questions", "compare candidates", "debrief", or "post-interview verdict". | - |

## remote-service-incident-debugging

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `remote-service-incident-debugging` | 1.1.0 | Debug a stuck or broken production service running on a remote host (VPS, container, server) over SSH, with no source tree at the local cwd. Use when a user reports an app is "stuck on a loading screen", "frozen on splash", "won't connect", returns 502, or has a dead listener — and the only way in is SSH. | - |

## video-preproduction

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `video-preproduction` | 1.0.0 | Video pre-production in two phases — (1) VET a tool/repo/product actually works before committing recording time, then (2) BUILD the simplest reproducible steps the presenter records from. Use when the user says "pre-production", "pre-prod", "test before filming", "vet this tool", "try it out", "sandbox test", "viability check", "does this actually work", "figure out simplest steps", "reproducible steps", "demo prep", "build the use case". | - |

## video-presentation-review

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `video-presentation-review` | 1.0.0 | Build and refine a video presentation slide-by-slide, alternating with the presenter. Covers the alternating explain/show structure, the coverage tracker, and the bullet-by-bullet iteration loop. Use when the user says "presentation slides", "slide outline", "video deck", "what's the next slide", "give me options for this bullet", or "presentation review". | - |

## yuanbao

**1 skill** in this category.

| # | Skill | Version | Description | Tags |
|---|-------|---------|-------------|------|
| 1 | `yuanbao` | 1.0.0 | Yuanbao (元宝) groups: @mention users, query info/members. | - |

---

## How to use this index

- **To load a skill in Hermes:** use `skill_view(name="<skill-name>")` from a chat - the name column above is the loader key.
- **To find the on-disk source:** see the per-category deep-dive docs in this repo, which embed the full SKILL.md body for every entry.
- **To add a skill:** drop a new `SKILL.md` into `C:/Users/soren/AppData/Local/hermes/skills/<category>/<skill-name>/` and re-run the generator.

## See also

- [`AGENTS.md`](./AGENTS.md) - reference for the 20 protocol/agent instruction files
- [`/docs/categories/`](./docs/categories/) - per-category deep-dive docs with full SKILL.md bodies
- [`/docs/skills/`](./docs/skills/) - one file per skill, for grep/diff-friendly browsing
