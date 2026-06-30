# Category: interview-prep-helper

> Deep-dive for the **1 skill** in the `interview-prep-helper` category. Each section embeds the full `SKILL.md` body so this doc is self-contained.

**Skills in this category:**

- [`interview-prep-helper`](../skills/interview-prep-helper/interview-prep-helper.md) - Prepare for and run a structured job-candidate interview. Parses the latest candidate application, builds a TLDR with fl

---

## interview-prep-helper

- **Name:** `interview-prep-helper`
- **Version:** 1.0.0
- **Author:** Hermes Agent
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/interview-prep-helper/SKILL.md`

**Description:**

> Prepare for and run a structured job-candidate interview. Parses the latest candidate application, builds a TLDR with flags, generates 5-6 sharp custom questions, tracks asked questions + live notes during the call, and runs a post-interview debrief. Use when the user says "interview prep", "parse the candidate", "build interview questions", "compare candidates", "debrief", or "post-interview verdict".

**Full SKILL.md body:**

```markdown
---
name: interview-prep-helper
description: Prepare for and run a structured job-candidate interview. Parses the latest candidate application, builds a TLDR with flags, generates 5-6 sharp custom questions, tracks asked questions + live notes during the call, and runs a post-interview debrief. Use when the user says "interview prep", "parse the candidate", "build interview questions", "compare candidates", "debrief", or "post-interview verdict".
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [interview, hiring, recruiting, candidate, debrief]
---

# Interview Prep Helper

Structured workflow for preparing, running, and debriefing a candidate interview. Two files matter: a **master question file** (per role, source of truth) and a **per-candidate file** (live state during the interview).

## File layout (adapt to your conventions)

1. **Master question files (per role):** e.g. `hiring/interview-questions-{role}.md`
   - One master per role. Fallback to a generic `interview-questions.md` if no role-specific file.
   - Reusable questions only.
   - **Never edited during interviews.** Source of truth.
   - Numbered list 1-N.

2. **Per-candidate file:** `hiring/candidates/{firstname-lastname}.md` (kebab-case)
   - Created at the start of each interview by COPYING the role-appropriate master.
   - Holds: TLDR, generic questions, custom questions, return-questions, notes, asked log, post-interview verdict.
   - **All live actions happen in this file only.**

## Subcommands (triggers)

- "Next candidate" / "parse the CSV" → **Parse + Init**.
- "Give me X custom questions" / "attack the gaps" → **Generate**.
- "Mark #N as asked" → **Track**.
- "Note:" / "add to notes" / any in-passing remark → **Note**.
- "Compare candidates X and Y" / "CSV vs CSV" → **Compare**.
- "Debrief" / "interview over" / "extract my impressions" → **Post-Interview Verdict**.

## Procedure

### 1. Parse + Init candidate file

1. Confirm the role if unclear.
2. Find the newest application file (CSV / form export / PDF) in the user's downloads or inbox.
3. Read it fully. Build TLDR:
   - **Identity:** name, general location, current status
   - **The good:** concrete numbers, real achievements, verifiable experience
   - **Yellow/red flags:** evasions, vague claims, contradictions, mismatches
   - **What's MISSING:** dodged questions, blanks, no portfolio links
   - **Bottom line:** one-sentence verdict
4. If they reference a company / channel / person, **web search** to verify. Report concisely.
5. **Create the candidate file** by copying the role-specific master into it.
6. Add a `## Her Questions to Address (end of interview)` section if the applicant returned questions in the form.

### Candidate file template

```markdown
# Interview — {Full Name}

**Date:** YYYY-MM-DD
**Source:** <path to original application>

## TLDR
{Identity, the good, flags, missing, bottom line}

## Generic Questions
1. {copied from master}
2. {copied from master}
...

## Custom Questions
{added in step 2 — candidate-specific only}

## Their Questions to Address (end of interview)
{only if the applicant returned questions in the form}

1. {their question}
2. {their question}

## Notes during interview
(none captured)

## Already Asked
- ✅ {question text}
- ✅ [their] {their question} ← when interviewer addresses one of THEIR questions
```

### 2. Generate custom questions

5-6 questions. Append under `## Custom Questions`, **continuing the numbering** from where the generic list ended.

Rules:
- Direct, specific, uncomfortable when warranted. No softballs.
- Attack gaps (dodged questions, blanks, vague answers).
- Probe inconsistencies.
- Test whether stated experience holds up under specifics.
- Match the actual hiring need.
- Add one-line *italic context* under each — explaining what the question probes.

### 3. Track ✅ live (during interview)

When the interviewer says "mark #N as asked":
1. Find #N in `## Generic Questions` or `## Custom Questions`.
2. Remove that line.
3. Append to `## Already Asked` as `- ✅ {question text}`.
4. **DO NOT renumber.** Gaps are intentional.

When the interviewer says "add a new question 'X'": append to the active list with the next sequential number.

### 4. Note (during interview)

Append a bullet to `## Notes during interview` in the candidate file. Terse, one bullet per insight.

**Trigger phrases:** "note:", "add to notes", "observation:", or any in-passing remark about vibe / energy / setup / professionalism / red flag / green flag. Always capture — the interviewer won't always say "note:".

### 5. Compare candidates (application vs application)

When comparing two applications side-by-side:
1. Read both source applications (NOT the candidate files — keep impressions out).
2. Build a table covering: relevant commitments, current status, role-specific proof, scenario answers, quality signals, return-question count, polish.
3. End with a 1-2 sentence verdict on who's stronger **on paper**, explicitly noting interview impressions are excluded.

### 6. Post-Interview Verdict

When the interview ends ("interview over" / "extract my impressions" / "debrief"):

1. Collect structured input:
   - Overall verdict (strong yes / maybe / pass / backup)
   - Energy & vibe
   - Top strengths (multi-select)
   - Red/yellow flags (multi-select)
   - Next step (reject / paid trial / 2nd round / wait / offer)
   - Anything else (freeform)
2. Write a `## Post-Interview Verdict` section at the bottom of the candidate file. Use clear headers, bullet lists for strengths/flags, bold the verdict + next step.

## Pitfalls

- **Don't edit the master during a live interview.** All live actions happen in the candidate file.
- **Don't drop reusable questions during a master edit.** Many "custom" questions are role-generic. Ask before dropping.
- **Don't renumber when marking asked.** Numbers are burned. Gaps are intentional.
- **Don't restart numbering for custom questions.** Generic + custom share one continuous sequence.
- **Don't soften flags in TLDRs.** If the candidate is weak, say so directly.
- **Don't fabricate verification.** If web search finds nothing, say "couldn't verify."
- **Don't pre-fill answers.** Only mark a question asked when the interviewer explicitly says so.
- **Don't lose notes by overwriting.** Always Edit (append) — never Write the whole file unless reconstructing.
- **Don't mix interview impressions into an application-vs-application compare.** That defeats the purpose.

## Verification

- **Parse + Init:** candidate file exists with TLDR + Generic Questions + (Their Questions if any) + empty Custom/Notes/Asked sections.
- **Generate:** new `## Custom Questions` entries, numbering continues, italic context under each.
- **Track:** question gone from active list, present as `- ✅ ...` in Already Asked, remaining numbers unchanged.
- **Note:** new bullet in `## Notes`, nothing else changed.
- **Compare:** side-by-side table, no interview impressions, explicit "on paper" verdict.
- **Post-Interview Verdict:** new section at bottom, structured fields, bolded verdict + next step.

```

---
