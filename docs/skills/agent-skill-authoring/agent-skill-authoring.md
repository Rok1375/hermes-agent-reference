---
name: agent-skill-authoring
description: How to write effective agent skills (SKILL.md) — anatomy, progressive disclosure, description-routing contract, design patterns, anti-patterns, ship checklist. Use when the user says "create a skill", "improve this skill", "why isn't my skill triggering", or is editing a SKILL.md file.
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [skills, authoring, SKILL.md, agentskills, meta]
---

# Agent Skill Authoring

A reference for writing SKILL.md files that route reliably, execute correctly, and stay lean.

## What an agent skill is

A folder containing a `SKILL.md` (YAML frontmatter + markdown body), plus optional `scripts/`, `references/`, `assets/`. Skills are an open standard adopted across 40+ agent products — a skill written once works across all compatible agents.

```text
my-skill/
├── SKILL.md          # Required: metadata + instructions
├── scripts/          # Optional: executable code (CLIs, validators)
├── references/       # Optional: detailed docs loaded only when needed
└── assets/           # Optional: templates, fonts, static files
```

## Progressive disclosure (the architectural core)

- **Level 1 (Discovery, ~100 tokens/skill):** Only `name` + `description` are in the system prompt. Agent knows the skill exists and roughly when it applies. Dozens of skills add negligible overhead.
- **Level 2 (Activation, <5,000 tokens):** When the user's request matches the description, the full body loads.
- **Level 3 (Execution, unbounded):** `references/foo.md` and `scripts/` load only as needed. Scripts can execute without their source being read.

This is why bundled content has no practical limit. Files don't burn tokens until accessed.

## Frontmatter anatomy

```markdown
---
name: skill-name
description: What this skill does AND when to use it. Include trigger phrases the user will say.
---
```

Hard constraints:
- `name` is lowercase, hyphens only, 1-64 chars, **exactly matches the parent folder name**
- Avoid `<` and `>` in frontmatter (can inject into the system prompt)
- Invalid YAML silently prevents loading

## The description is the routing contract

The description is the only thing the agent sees before deciding to load. If a skill doesn't trigger, the description is wrong 95% of the time, not the body.

Include three elements:
1. **What** the skill does (one phrase)
2. **When** to use it (trigger phrases, situations)
3. **Differentiator** vs related skills (prevents routing conflicts)

Pattern: `"X via Y. Use for [situations]. [Differentiator: no Z required / faster than W / handles edge case V]."`

## Two design philosophies

**Pattern A — Capability primitive (tool wrapper).** Thin wrapper over a deterministic CLI or script. Logic in code, SKILL.md teaches invocation.
- 30-80 lines, mostly command examples
- Use when: the bottleneck is "the agent can't do X"

**Pattern B — Process primitive (cognitive discipline).** Encodes a methodology. Pure prompt engineering, no scripts.
- Validation loops, checklists, explicit procedure
- Use when: the bottleneck is "the agent's output quality is bad"

Mature setups use both. A gives better tools; B gives better methods.

## Do this

- **Bash-first, prose-second.** Concrete `bash ...` blocks with inline comments beat prose. The agent pattern-matches on syntax.
- **Push determinism into code.** Anything fragile, repetitive, or where variation is a bug → script. Markdown for judgment only.
- **Build validation loops.** State the verify → fix → re-verify loop explicitly. Single biggest quality improvement.
- **State-check before action.** Verify state first, then branch. `First check if X is configured: [cmd]. If not, walk the user through setup.`
- **Just-in-time loading.** Tell the agent exactly when to read each referenced file.
- **Document output formats.** If a script returns structured data, show the agent what it looks like.
- **Defer to `--help` for completeness.** List the 80% in SKILL.md. Tell the agent to run `tool --help` for the rest.
- **Compose primitives, don't bundle workflows.** One skill = one capability or one discipline.
- **Cite established principles.** If a skill encodes TDD / DDD / red-green-refactor, name the source.

## Don't do this

- **Don't re-teach what the model already knows.** No Python syntax tutorials. No "what is git." Every line should add context the model lacks.
- **Don't include human-facing docs.** No README.md, CHANGELOG.md, INSTALLATION_GUIDE.md inside the skill folder. Skills are for agents.
- **Don't write vague descriptions.** Bad: "A helpful skill for documents." Good: "Fill PDF form fields and flatten completed PDFs. Use when the user mentions fillable PDF forms."
- **Don't bundle library code.** Install via npm/pip. Don't paste source into the skill.
- **Don't write monolithic mega-skills.** If one skill does design + planning + implementation + testing + deployment, you've built a framework. Split it.
- **Don't assume the agent will infer.** Be explicit. Bad: "Then deploy it." Good: "Run `npm run deploy:staging` and wait for HTTP 200 from /healthz."
- **Don't write style-only variants.** Tone/formatting belongs in user preferences or system prompt, not a skill.
- **Don't ignore failure modes.** For every step that can fail, document what failure looks like and what to do.
- **Don't include time-sensitive info.** "As of Q4 2024..." rots fast. Fetch live data or omit.
- **Don't use absolute paths.** Always relative. Forward slashes regardless of OS. Use runtime placeholders for skill-dir references.
- **Don't trust unfamiliar skills.** Audit `scripts/` for outbound network calls, hidden instructions, file access outside expected scope. Sandbox first.

## Authoring workflow

1. **Identify the gap.** Run the agent on real tasks. Where does it consistently fail or need re-prompting? That's a skill candidate.
2. **Decide the pattern.** Capability primitive (need new tools) or process primitive (need better methodology)?
3. **Draft the description first.** What + when + differentiator. Read it back: would the agent know when to fire it?
4. **Write the smallest body that works.** Start under 30 lines. Add only when testing reveals gaps.
5. **Move detail to `references/`** once SKILL.md crosses 100 lines.
6. **Test triggering.** Ask the agent something the skill should handle without naming it. If it doesn't fire, fix the description.
7. **Test execution.** Invoke explicitly. If output is wrong, fix the body.
8. **Adversarial test.** Have another LLM ask: "What edge cases break this skill?" Patch the gaps.
9. **Version control.** Treat skills as code. Tag, branch, review.

## Debugging

- "Which skill did you use?" — ask the agent post-task. Fastest routing debug.
- **Routing fails → description problem.** Add specific trigger phrases.
- **Execution fails → body problem.** Add explicit steps, examples, or validation.
- **Skills snapshot at session start.** Edits during a session require a restart.
- **Test against the weakest model you'll deploy on.** Stronger models forgive vague skills; weaker models expose them.

## Ship checklist

- [ ] Frontmatter `name` matches folder name
- [ ] Description includes what + when + differentiator
- [ ] Description includes likely user trigger phrases
- [ ] Body under 100 lines (or length is justified)
- [ ] No human-facing docs inside the skill folder
- [ ] No time-sensitive information
- [ ] Relative paths only
- [ ] State-check before action where applicable
- [ ] Validation loop documented
- [ ] Output format documented if relevant
- [ ] Tested with weak and strong models
- [ ] Tested for both correct triggering and correct execution
- [ ] Skill does one thing
- [ ] Composes cleanly with related skills
- [ ] Version controlled
