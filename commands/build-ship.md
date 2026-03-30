---
name: build:ship
description: QA, capture learnings, and ship — the final gate before deploy. Use after code review passes.
argument-hint: "[PR number, branch name, or empty for current branch]"
---

# Phase 3: SHIP — Prove It Works and Deploy

This phase ensures quality, captures institutional knowledge, and ships.

## Input

<input> #$ARGUMENTS </input>

If no PR or branch specified, detect from current git state:

```bash
current_branch=$(git branch --show-current)
echo "Branch: $current_branch"
gh pr list --head "$current_branch" --json number,title,url --jq '.[0]' 2>/dev/null || echo "No open PR found"
```

---

## Step 9: QA Testing

Run real browser-based QA testing via gstack:

```
skill: qa
```

The gstack QA skill launches a headless browser for real navigation, real clicks, real assertions. Follow its full process:
- Happy path testing
- Edge case exploration
- Visual regression checks
- Console error detection
- Mobile viewport testing (if applicable)

**Gate** — after QA, use **AskUserQuestion**:

1. **Pass** — QA clean, proceed to capture learnings
2. **Bugs found** — fix the issues, then re-run QA
3. **Back to make** — significant issues discovered, needs more development work

---

## Step 10: Capture Learnings

Record what was learned so the next build cycle benefits.

Load the CE compound workflow:

```
skill: workflows:compound
```

The compound skill launches five parallel subagents to extract lessons:

1. **Context Analyzer** — problem type, component, symptoms
2. **Solution Extractor** — root cause, working solution, code examples
3. **Related Docs Finder** — cross-references with existing `docs/solutions/`
4. **Prevention Strategist** — how to avoid this class of problem
5. **Category Classifier** — organizes into the right `docs/solutions/` category

Output: a single structured file in `docs/solutions/[category]/[filename].md`

These learnings are automatically picked up by future `/build:make` plan phases — the CE plan skill searches `docs/solutions/` for relevant prior art.

---

## Step 11: Ship It

Deploy and close the loop via gstack:

```
skill: ship
```

The gstack ship skill handles the final deployment flow — push, PR merge, and deploy.

---

## Build Complete

After shipping, present the full journey summary:

```markdown
## Build Complete

**Feature:** [title from brief]
**Timeline:** [started] → [shipped]

**Artifacts:**
- Brief: docs/briefs/[file]
- Plan: docs/plans/[file]
- Learnings: docs/solutions/[file]
- PR: [url]

**Phases:**
- [x] Think — validated idea with CEO + eng review
- [x] Make — brainstormed, planned, built, reviewed
- [x] Ship — QA passed, learnings captured, deployed

Next time you `/build`, the plan phase already knows everything learned here.
```
