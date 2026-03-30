---
name: build:think
description: Validate the idea before building — 95% confidence interview, challenge, CEO gate, eng gate. Use before committing to build anything non-trivial.
argument-hint: "[what you want to build]"
---

# Phase 1: THINK — Validate Before Building

This phase ensures you're building the right thing before writing a line of code.

## Input

<idea> #$ARGUMENTS </idea>

If the idea above is empty, ask: "What do you want to build?" Do not proceed without a clear starting point.

---

## Step 1: Clarify — 95% Confidence Interview

Interview the user until you have 95% confidence about what they **actually want**, not what they think they should want.

**Rules:**
- Ask ONE question at a time using the **AskUserQuestion** tool
- Prefer multiple-choice when natural options exist
- Dig beneath surface requests — ask "why" at least twice
- Distinguish between the **problem** and the user's **proposed solution**
- Validate assumptions explicitly: "I'm assuming X — correct?"
- Track your confidence level and share it: "I'm at ~70% confidence — a few more questions"

**Question progression:**
1. What problem does this solve? Who feels the pain?
2. What does success look like? How will you know it's working?
3. What's the scope boundary — what is explicitly NOT included?
4. Are there existing patterns in this codebase to follow?
5. What's the timeline/urgency?

**Exit when:**
- You could explain this feature to a different engineer and they'd build the right thing
- Or the user says "proceed" / "good enough"

**Output:** Write a 3-5 sentence **Feature Brief** summarizing: the problem, who it's for, what success looks like, and key constraints. Present it to the user for confirmation.

---

## Step 2: Challenge — Office Hours

Now push back on the brief. Play devil's advocate. Present challenges ONE AT A TIME using **AskUserQuestion**, letting the user respond before the next:

1. **Problem validation:** "Is this actually a problem worth solving, or a nice-to-have? What evidence do you have?"
2. **Scope challenge:** "What's the absolute smallest version that would prove or disprove the value?"
3. **Alternatives:** "What if you didn't build this at all? What breaks?"
4. **Hidden complexity:** "What's the hardest part you haven't mentioned yet?"
5. **Timing:** "Why now? What changes if you wait a week?"

After the challenges, revise the Feature Brief based on the user's answers. Present the updated brief.

---

## Step 3: CEO Review Gate

Invoke the gstack CEO/founder review to stress-test the product thinking:

Use the Skill tool to invoke `plan-ceo-review`, passing the Feature Brief as context. The skill provides four modes: scope expansion, selective expansion, hold scope, and scope reduction. Follow its full process.

**Gate decision** — after the CEO review, use **AskUserQuestion** to ask:

1. **Proceed** — idea is validated, move to eng review
2. **Revise** — update the brief based on feedback, re-run this step
3. **Kill** — idea isn't worth building right now (end workflow)

---

## Step 4: Engineering Review Gate

Invoke the gstack engineering review:

Use the Skill tool to invoke `plan-eng-review`, passing the Feature Brief + CEO review outcome as context. The skill evaluates technical feasibility, architecture risks, and implementation complexity.

**Gate decision** — after the eng review, use **AskUserQuestion** to ask:

1. **Proceed to build** — ready for `/build:make`
2. **Revise** — update brief based on technical feedback, loop back
3. **Kill** — too risky or complex right now (end workflow)
4. **Save for later** — write brief to file, stop here

---

## Output

Write the validated brief to:

```
docs/briefs/YYYY-MM-DD-<kebab-case-topic>-brief.md
```

Use this frontmatter:

```yaml
---
title: [Brief Title]
status: validated
date: YYYY-MM-DD
ceo_review: pass
eng_review: pass
next_step: /build:make
---
```

Then tell the user:

> Brief saved to `docs/briefs/[filename]`. Run `/build:make` when you're ready to start building.
