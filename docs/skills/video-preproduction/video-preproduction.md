---
name: video-preproduction
description: Video pre-production in two phases — (1) VET a tool/repo/product actually works before committing recording time, then (2) BUILD the simplest reproducible steps the presenter records from. Use when the user says "pre-production", "pre-prod", "test before filming", "vet this tool", "try it out", "sandbox test", "viability check", "does this actually work", "figure out simplest steps", "reproducible steps", "demo prep", "build the use case".
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [video, pre-production, demo, vetting, reproducible-steps, youtube, tutorial]
---

# Video Pre-Production

Two phases of getting a video ready to record:

1. **VET** — does the tool/repo/product actually work as advertised? Find the demo-worthy moments. Make a go/no-go decision before investing recording time.
2. **BUILD** — once it's a go, figure out the simplest reproducible path to the end outcome and write it up as a clean step-by-step file the presenter records from.

Phase 1 is for tool/repo/product videos. Phase 2 is for any build/use-case video. Many videos need both; some need only one.

## Outline structure (scaffold, adapt per video)

A high-level beat-by-beat skeleton with a few talking points under each — not a full script. The presenter fills exact intro and wording at record time; the outline locks the order and the points that must be hit.

Default scaffold (adapt to genre):

1. **Hook (first ~15 seconds).** Open with the payoff, not the setup. Stacked list of wild outcomes. If a credibility proof exists (a famous dev endorsing it, a real result), drop it here.
2. **Problem + promise.** Name the old pain, then state the agenda plainly: "in this video I'm going to show you A, B, and C."
3. **Retention hook + objection crushing.** "If you're serious about X, watch until the end." Pre-empt the viewer's excuses fast.
4. **Setup / install.** Step-by-step from scratch, plain English, zero assumed knowledge. Plug the resource bundle.
5. **Main demo (the meat).** The walkthrough. Every step explained with its WHY. This is where the pre-production phases pay off.
6. **Sponsor / external beat.** Weave the sponsor natively, not bolted on — the value must be a real beat in the workflow.
7. **Soft mid-video CTA.** Slip in the subscribe naturally.
8. **Outro CTA.** Recap the transformation → push action → links.

### Retention rules

- Open on the outcome, never the backstory.
- State "what you get by watching to the end" early, and re-tease it mid-video with a *different* reason.
- Crush objections before the viewer can raise them.
- Every section must add new value; no repeated examples or points.

### What makes an outline bad

- A weak/slow hook that buries the payoff.
- "A video about everything" — no single hero use-case as the spine (covering 5 things = covering nothing).
- A sponsor segment that feels bolted on.
- No retention hook / no clear reason to keep watching.
- Steps with no WHY behind them.

## Phase 1 — VET (tool/repo/product viability)

### 1. Understand the tool

- Read the video doc if one exists.
- Identify what the tool claims to do and the hook for the video.
- Ask what specific claims/features need to be verified.

### 2. Set up in sandbox

- Clone / install into a sandbox directory, step by step, explaining each step.
- Document setup friction (viewers will hit the same issues).
- **Never install into the main project directory.** Always use `sandbox/`.

### 3. Run the four-test viability check

Escalating difficulty. Each test must pass before the next:

| Test | Purpose | What it proves |
|---|---|---|
| **Sanity** | Trivial task | The tool runs at all |
| **Stress** | Complex / form-heavy task | Handles real-world complexity |
| **Self-healing** | Task requiring adaptation | The key differentiator works |
| **Real-workflow** | The actual use case | It's useful, not just a demo toy |

- Run one at a time. After each: what happened, did it match expectations, any surprises.
- Diff any files the tool modified.

### 4. Document results

Summarize: which tests passed / failed, where the "money shot" demo moments are, gotchas viewers will hit, go/no-go recommendation. Update the video doc.

### 5. Go/no-go decision

Present findings. The presenter decides whether to commit.

## Phase 2 — BUILD (simplest reproducible steps)

Once vetted, figure out the cleanest path to the end outcome so the presenter records cleanly.

**Core principle: Start with the end outcome. Then find the simplest reproducible path to it.** Trial-and-error is fine during exploration — NOT in the final output. Strip everything that wasn't actually needed.

### 1. Define the end outcome

Write exactly what the use case looks like when it works. What does the presenter demonstrate? What's the money shot? If unclear, ask — don't guess.

### 2. Explore freely (trial and error allowed)

Figure out how to achieve the outcome. Try, fail, iterate. Messy on purpose — none of it goes in the final output yet.

### 3. Strip to the simplest reproducible path

Remove every unneeded step, dead end, abandoned attempt, debugging detour. Keep only what a fresh user needs to reproduce the result.

### 4. Write the clean step-by-step markdown

One markdown file with:
- The end outcome (what we're building / showing)
- Step-by-step instructions, in order
- For EACH step: a short WHY — what it does, what breaks without it
- Non-obvious gotchas (things that look broken but aren't)
- Known issues: what they look like, how to fix

Plain English. The pre-prod person may not be a developer. The presenter must read this on camera and explain every step without guessing.

### 5. Verify the steps work — multiple times

Not optional. Run the stripped-down steps from scratch on a fresh setup, more than once (catch flakiness / hidden state). If anything fails, fix the file and run again. Only hand to the presenter once it reproduces cleanly every time. They should hit zero surprises while recording.

## Communicating with the presenter (CRITICAL)

The presenter needs to understand every step deeply enough to explain it on camera. Non-negotiable.

**Before each step:** tell them what you're about to do (one sentence) and WHY it's needed.

**After each step:**
- Report whether it worked as expected.
- **"What just happened:"** — explain in plain language what the step actually did under the hood. No jargon without explanation.
- **"Why we need it:"** — connect the step to the overall goal. Why can't we skip it?
- **"Next step:"** — preview what's next, ask before proceeding.

**Pacing rules:**
- ONE step at a time. Never batch or rush ahead.
- Wait for the go-ahead before each step.
- If something fails, explain what went wrong and why before fixing.
- Keep explanations concise but thorough — short sentences, plain language.
- NEVER assume the presenter already understands a concept. Nothing is "obvious."

**Test on the exact same setup the presenter will record on.** Recording on a VPS → test on a VPS. Recording on localhost → test on localhost. Never test in one environment and expect steps to work in a different one on camera.

## Pitfalls

- **Don't rush tests.** The presenter needs to understand every step for the video.
- **Don't skip the self-healing test** (phase 1) — usually the key differentiator. If it fails, rethink the angle.
- **Don't install into main project dirs.** Always `sandbox/`.
- **Don't assume marketing claims are true.** The whole point is to verify.
- **Don't include trial-and-error in the final steps** (phase 2). Minimal reproducible path only.
- **Don't skip the WHY on any step.** No reasoning = unusable for recording.
- **Don't guess the end outcome.** Ask.

## Verification

- **Phase 1:** Four tests run and documented; video doc updated with go/no-go.
- **Phase 2:** Clean step-by-step file reproduces the outcome cleanly, every time, on a fresh setup.
- The presenter has enough understanding to explain the whole thing on camera.
