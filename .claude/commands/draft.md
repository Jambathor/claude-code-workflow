---
description: Collaboratively create a detailed design/technical document through conversation. Produces a rich doc that can be fed to /specify and /create_plan.
model: opus
---

# Draft

You are collaborating with the user to produce a detailed design document. This is the thinking step — before specs, before plans. The goal is to get the idea out of the user's head and onto paper in enough detail that /specify and /create_plan can work from it.

## Input

$ARGUMENTS should describe what they want to design. Can be a sentence, a paragraph, or a rambling braindump. All valid.

If empty: ask "What are you trying to build? Just describe it however it comes to mind."

## Process

### Phase 1: Listen and Understand

Read the user's input. Identify:
- The core problem they're solving
- Any architectural ideas they already have (even half-formed)
- Constraints they've mentioned (performance, cost, platform, timeline)
- Things they're excited about vs. unsure about

### Phase 2: Interview (3-5 rounds max)

Ask focused questions to fill gaps. NOT requirements-gathering (that's /specify's job). This is design exploration:

- "You mentioned X — have you thought about how that interacts with Y?"
- "There are two ways to approach this: A (simpler, limitation) or B (more complex, advantage). Which feels right?"
- "What's the worst case if this is slow? Does it need to be real-time or is eventual OK?"
- "Who's the user in the first 5 seconds? What do they see?"

Rules:
- Ask 2-3 questions per round, not 10
- If the user says "you decide" — decide and document why
- If the user is on a roll explaining something — don't interrupt with questions, let them finish
- Mirror back your understanding: "So the flow is: X → Y → Z, and the key constraint is W?"

### Phase 3: Write the Design Doc

Write to `docs/designs/YYYY-MM-DD-{description}.md`.

There is NO fixed template. The structure should match the content. But good design docs typically include some combination of:

- **Problem** — what's broken or missing, in concrete terms
- **Solution overview** — the approach in 2-3 sentences
- **Architecture** — how the pieces fit together (diagrams in ASCII/mermaid if helpful)
- **Data model / schemas** — what the data looks like
- **Flow / sequence** — what happens step by step (timeline if relevant)
- **Key decisions** — choices made and why (alternatives considered)
- **Edge cases** — what could go wrong, what happens at the boundaries
- **Open questions** — things still unresolved (these become /clarify fodder later)

The user's design doc about the game tree is a great example of what this should produce — rich, specific, with schemas, timelines, and worked examples. Aim for that level of detail when the feature warrants it. For smaller features, a shorter doc is fine.

### Phase 4: Review Together

Present the doc to the user. Ask:
- "Does this capture what you had in mind?"
- "Anything missing or wrong?"

Iterate until the user is satisfied.

### Phase 5: Bridge to Next Steps

```
Design doc saved: docs/designs/YYYY-MM-DD-{description}.md

Next steps:
- /specify docs/designs/YYYY-MM-DD-{description}.md
  → Generates formal spec with user stories, requirements, acceptance criteria
- Or feed this doc directly to /create_plan if you want to skip the formal spec
```

## Critical Rules

- **This is collaborative, not extractive.** You're designing together, not interviewing a stakeholder. Contribute ideas, push back, suggest alternatives.
- **Concrete over abstract.** "A caching layer" is vague. "Redis cache with 5-minute TTL on user profiles, invalidated on write" is useful.
- **Worked examples are the best documentation.** A session simulation (like the game doc's timing walkthrough) communicates more than 10 paragraphs of description.
- **ASCII diagrams > no diagrams.** Even rough box-and-arrow art helps. Don't skip visuals because they're not pretty.
- **Open questions are valuable output.** Not everything needs to be resolved in one session. Flagging unknowns is as useful as resolving them.
- **Match depth to complexity.** A login feature needs 1 page. A game architecture rework needs 10. Don't pad small features or compress big ones.
