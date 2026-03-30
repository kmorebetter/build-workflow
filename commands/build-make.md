---
name: build:make
description: Build it right — brainstorm, plan, execute, review. Orchestrates compound-engineering workflow commands. Use after validating an idea with /build:think or when you know what to build.
argument-hint: "[brief file path, feature description, or plan file path]"
---

# Phase 2: MAKE — Build It Right

This phase turns a validated idea into reviewed, working code.

## Input

<input> #$ARGUMENTS </input>

### Find Context

First, check for a validated brief from the think phase:

```bash
ls -t docs/briefs/*-brief.md 2>/dev/null | head -5
```

**If a recent brief exists** (within 14 days): read it and announce: "Found validated brief from [date]: [title]. Using as context."

**If a plan file was passed as input**: skip to Step 7 (execute the plan directly).

**If no brief and no arguments**: ask "What are you building? (Tip: run `/build:think` first for a validated brief, or describe the feature here.)"

---

## Step 5: Brainstorm Approaches

Explore requirements and approaches before committing to an implementation.

Load the brainstorming skill:

```
skill: brainstorming
```

If a brief exists, pass it as context so the brainstorming skill skips its own idea-refinement phase and focuses on exploring **approaches** — not re-validating the idea.

**Gate** — after brainstorming, use **AskUserQuestion**:

1. **Proceed to plan** — approach selected, move to step 6
2. **Explore more** — continue brainstorming different angles
3. **Back to think** — idea needs more validation, suggest `/build:think`

---

## Step 6: Create Implementation Plan

Research the project and produce a detailed implementation plan.

Load the CE planning workflow:

```
skill: workflows:plan
```

Pass the brainstorm output and brief as context. The plan skill will:
- Run local research agents on the codebase (patterns, conventions, learnings)
- Optionally run external research for risky or unfamiliar territory
- Run SpecFlow analysis for edge cases
- Produce a structured plan in `docs/plans/`

The plan skill offers its own post-generation options. If the user selects "Start /workflows:work", proceed to Step 7.

Otherwise, let the user refine the plan. When they're ready, continue.

---

## Step 7: Execute the Plan

Execute the plan with task tracking and incremental commits.

Load the CE work workflow:

```
skill: workflows:work
```

Pass the plan file path from Step 6 (e.g., `docs/plans/YYYY-MM-DD-feat-...-plan.md`).

The work skill will:
- Break the plan into TodoWrite tasks
- Set up a feature branch or worktree
- Execute tasks with continuous testing
- Create incremental commits at logical boundaries
- Track progress by checking off plan items

**This step does the actual coding.** It may run for a while on complex features. Let it work.

---

## Step 8: Review the Work

Run a comprehensive multi-agent code review.

Load the CE review workflow:

```
skill: workflows:review
```

Pass the current branch name or PR number. The review skill will:
- Spawn parallel review agents (security, performance, architecture, quality)
- Run ultra-thinking deep dive analysis
- Create todo files in `todos/` for findings
- Categorize by severity: P1 (blocks merge), P2 (should fix), P3 (nice-to-have)

**Gate** — after review, use **AskUserQuestion**:

1. **Ship it** — code passes review, run `/build:ship`
2. **Fix findings** — address P1/P2 issues, then re-run this step
3. **Revise plan** — major architectural issues found, go back to step 6

---

## Handoff

After successful review, tell the user:

> Code reviewed. Run `/build:ship` when ready to QA and deploy.
>
> **Summary:**
> - Brief: `docs/briefs/[file]`
> - Plan: `docs/plans/[file]`
> - Branch: `[branch-name]`
> - Review findings: [X] P1, [Y] P2, [Z] P3
