---
name: build
description: Full end-to-end feature workflow — think, make, ship — with gates between phases. The complete ceremony for building features right.
argument-hint: "[what you want to build]"
---

# /build — Full Feature Workflow

End-to-end feature development in three gated phases. Each phase has a go/no-go gate before proceeding.

## Input

<idea> #$ARGUMENTS </idea>

If empty, ask: "What do you want to build?"

---

## Phase 1: THINK (Steps 1-4)

Validate the idea before writing code.

Use the Skill tool to invoke `build:think`, passing the idea as arguments. This phase runs:
1. 95% confidence interview — clarify what you actually want
2. Office hours challenge — devil's advocate pushback
3. CEO review gate — product validation via gstack
4. Eng review gate — technical feasibility via gstack

**Output:** Validated brief in `docs/briefs/`

**Phase gate:** The think phase ends with a go/no-go. Only proceed if the user chooses to continue.

---

## Phase 2: MAKE (Steps 5-8)

Build it right with structured planning and execution.

Use the Skill tool to invoke `build:make`. The brief from Phase 1 is auto-detected from `docs/briefs/`. This phase runs:
5. Brainstorm approaches — explore options via CE
6. Implementation plan — research + structured plan via CE
7. Execute the plan — task tracking + incremental commits via CE
8. Code review — multi-agent review ensemble via CE

**Output:** Reviewed code on a feature branch with PR

**Phase gate:** The make phase ends with review results. Only proceed if code passes review.

---

## Phase 3: SHIP (Steps 9-11)

Prove it works and deploy.

Use the Skill tool to invoke `build:ship`. This phase runs:
9. QA testing — real browser, real clicks via gstack
10. Capture learnings — five subagents extract lessons via CE
11. Ship it — deploy via gstack

**Output:** Deployed feature + learnings in `docs/solutions/`

---

## Phase Entry Points

You don't have to start from Phase 1 every time:

| Command | When to use |
|---------|-------------|
| `/build [idea]` | Full ceremony — new feature from scratch |
| `/build:think [idea]` | Just validate an idea, decide if it's worth building |
| `/build:make [brief or description]` | You know what to build, skip validation |
| `/build:ship [PR or branch]` | Code is ready, just need QA + deploy |

## Dependencies

This workflow requires two other tools to be installed:

- **gstack** — provides: plan-ceo-review, plan-eng-review, qa, ship
- **compound-engineering** — provides: brainstorming, workflows:plan, workflows:work, workflows:review, workflows:compound

If a required skill is missing, stop and tell the user which dependency needs to be installed.
