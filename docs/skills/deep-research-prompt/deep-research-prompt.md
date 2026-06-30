---
name: deep-research-prompt
description: Write a single-paragraph Deep Research prompt to hand to a human researcher or a deep-research AI. Use when the user wants a research brief, a "deep research prompt", a one-paragraph task for a researcher, or asks "what should our researcher look for". Produces ONE tight paragraph with full context, numbered sub-questions, and per-finding output format.
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [research, prompt-engineering, brief, deep-research]
---

# Deep Research Prompt

Goal: turn a vague research need into ONE self-contained paragraph the researcher can act on with zero back-and-forth.

## Rules

- **One paragraph.** No headers, no bullet list in the deliverable.
- **Prompt the job, not the topic.** Give search handles (timeframe, ranking, source type, decision logic) — not just a subject.
- **Lead with the goal + decision.** First sentence = the single question the research must answer and the decision/use it informs.
- **Embed all context.** Names, dates, product, prior known facts, constraints. The researcher must not need to ask anything or guess.
- **Number the sub-questions inline** (1, 2, 3…) so coverage is explicit. Keep to 3-6. One mission per prompt — don't cram unrelated questions.
- **State constraints.** What to include, what to avoid (e.g. "only non-Chinese competitors", "no marketing fluff").
- **Source hierarchy.** Prefer primary sources (official docs, GitHub, papers, filings, changelogs); forums/X/Reddit are weak signal only, never factual proof.
- **Contradiction handling.** If sources conflict, separate confirmed facts / inference / unresolved uncertainty — don't force fake consensus. Flag low-confidence claims for verification.
- **Constrain output hard, method loosely.** Be strict on the deliverable; leave the search path flexible.
- **Demand a fixed output per finding:** source link + specific claim + one-line "why it matters / why a viewer should care".
- Verifiable, citable facts only. No opinions.
- **Last sentence:** instruct them to output everything into a single detailed markdown file.

## Process

1. Pull context from the relevant project files / conversation (dates, names, known facts, audience, end use).
2. Identify the ONE question the research answers.
3. Draft 3-6 numbered sub-questions that fully cover it.
4. Add include/avoid constraints + the per-finding output format.
5. Compress to one clean paragraph. Cut filler.

## Template

> Research [TOPIC + key identifying facts] to answer one question: [THE QUESTION] — for [DECISION / END USE]. Find: (1) …; (2) …; (3) …; (4) …. [Constraints: include X, avoid Y.] Prefer primary sources; treat forums/social as weak signal only; if sources conflict, separate fact from inference and flag what needs verification. For each point, give the source link, the specific claim, and a one-line "why it matters". No marketing fluff — verifiable, citable facts only. Output everything into a single detailed markdown file.

## Pitfalls

- **Don't write a multi-paragraph brief.** The whole point is one paragraph the researcher can act on without back-and-forth.
- **Don't bury the question.** Lead with the goal + decision in sentence one.
- **Don't forget the "why it matters".** Findings without stakes are a list, not research.
- **Don't add a bullet list in the deliverable.** Inline numbering only.
- **Don't accept marketing copy as fact.** Constrain explicitly: "no marketing fluff."
