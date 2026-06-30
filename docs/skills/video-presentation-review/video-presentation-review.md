---
name: video-presentation-review
description: Build and refine a video presentation slide-by-slide, alternating with the presenter. Covers the alternating explain/show structure, the coverage tracker, and the bullet-by-bullet iteration loop. Use when the user says "presentation slides", "slide outline", "video deck", "what's the next slide", "give me options for this bullet", or "presentation review".
type: tool_workflow
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [video, presentation, slides, deck, youtube, tutorial]
---

# Video Presentation Review

Build and refine video presentations slide-by-slide, alternating with the presenter.

## Visual style (default; adapt to channel brand)

- **Background:** White. **Text:** Black. Title in serif, big.
- **Key phrases:** Highlighted in a single accent color (yellow is a common default).
- **Each slide:** One concise title + ~3 bullet points (presenter notes telling them what to say).
- **Bullets:** Plain English sentences. NOT "word — explanation" format. The format feels stilted when read aloud.
- **Showcase slides:** Accent-color title on near-empty slide (just the action name + any URL).

## Alternating explain/show structure (THE core format)

Alternate ~2 explain slides → 1 showcase step, repeating. Never 20 min of talking then 20 min of demo. Dynamic, vibrant, alternating.

- **Explain slides** = educational, talked over.
- **Showcase slides** = live screen recording of setup / implementation.
- Big or lead-magnet moments can get 2 showcase steps back-to-back.

## Working files (one per video, in the video's folder)

- `EXPLAIN.md` — numbered list of all educational topics.
- `SHOWCASE.md` — numbered list of all setup / showcase steps.
- (optional) `topics.csv` — coverage tracker; column 2 = `mentioned` (YES/NO). Use to verify the deck covers 90-95% of the master doc.

Mark completed items in EXPLAIN.md / SHOWCASE.md with ✅ as slides get built.

## The iteration loop

The presenter builds the deck themselves (e.g. in Google Slides). They paste screenshots and ask for help one piece at a time. Your job per turn:

1. **They paste a slide screenshot.** Read it fully.
2. **They ask for the next move.** Usually one of:
   - "What should the next 2 explain slides be, and why?" → give top 3 options, each concise, with a one-line WHY. Recommend one.
   - "What should we show / setup next?" → name the SHOWCASE step(s) + why they pay off a promise just made.
   - "Give me N options for this bullet point." → give N plain-English sentences, then your pick + one-line reason.
   - "Mark these with the emoji." → add ✅ to the matching EXPLAIN / SHOWCASE rows.
3. **Keep flow continuity.** Every suggestion must flow naturally from the previous slide and pay off promises already made.

## Hard rules

- **Be very concise.** Short options, short WHY. No verbose essays.
- **Plain sentences only** in bullets — never the "label — sentence" format.
- **Fact-check claims** when asked, before they go on a slide.
- **ZERO REPETITION.** Never reuse an example, quote, screenshot, stat, or person across slides. Cross-check against earlier slides; flag duplicates.
- Every slide adds NEW value and earns its ~3 bullets.
- **Retention:** weave in distinct reasons to keep watching; pattern-break dense slides with simple / visual ones.

## Slide 1 & 2 (the hook frame)

- **Slide 1:** Big visual hook, often minimal text. Punchy, exciting — it's the first frame.
- **Slide 2:** What the video is + why to care. Bullet 3 = what the viewer gets by watching to the end.

## Pitfalls

- **Don't pad slide suggestions.** The presenter has 10 seconds to evaluate each option. Short, sharp, ranked.
- **Don't reuse the same example across slides.** Cross-check the deck so far.
- **Don't write "label — sentence" bullets.** The presenter reads them aloud; they read as stilted.
- **Don't skip the ✅ marking.** The coverage tracker is the only way to verify 90-95% topic coverage.
- **Don't suggest a SHOWCASE step that doesn't pay off a promise.** The explain→showcase alternation is the format.
- **Don't include meeting / call / time-locked items on the priority list.** Only actionable work items.
