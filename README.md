# build-workflow

End-to-end feature workflow plugin for Claude Code. Three phases, eleven steps, zero yolo deploys.

Orchestrates [gstack](https://github.com/garrytan/gstack) and [compound-engineering](https://github.com/EveryInc/compound-engineering-plugin) into a single gated pipeline.

## The Flow

```
/build:think                    /build:make                       /build:ship
 ┌─────────────────────┐        ┌──────────────────────┐         ┌──────────────────┐
 │ 1. Clarify (95%)    │        │ 5. Brainstorm        │         │ 9.  QA testing    │
 │ 2. Challenge        │──gate──│ 6. Plan              │──gate───│ 10. Learnings     │
 │ 3. CEO review       │        │ 7. Execute           │         │ 11. Ship          │
 │ 4. Eng review       │        │ 8. Code review       │         │                   │
 └─────────────────────┘        └──────────────────────┘         └──────────────────┘
       docs/briefs/            docs/brainstorms/ + docs/plans/       docs/solutions/
```

| Step | Phase | Source | What Happens |
|------|-------|--------|--------------|
| 1 | Think | Custom | 95% confidence interview — clarify what you actually want |
| 2 | Think | Custom | Devil's advocate challenge — poke holes in the idea |
| 3 | Think | gstack | CEO/founder review — product-level validation |
| 4 | Think | gstack | Engineering review — technical feasibility gate |
| 5 | Make | CE | Brainstorm approaches — explore before committing |
| 6 | Make | CE | Research + implementation plan — agents scan your codebase |
| 7 | Make | CE | Execute with task tracking — incremental commits |
| 8 | Make | CE | Multi-agent code review — security, performance, architecture |
| 9 | Ship | gstack | Browser-based QA — real clicks, real assertions |
| 10 | Ship | CE | Capture learnings — 5 subagents extract lessons |
| 11 | Ship | gstack | Ship and deploy |

## Install

### Step 1: Install prerequisites

You need two tools installed. If you already have them, skip to Step 2.

#### gstack (provides: CEO review, eng review, QA, ship)

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
```

Verify it installed correctly:

```bash
ls ~/.claude/skills/gstack/plan-ceo-review/SKILL.md && echo "gstack OK"
```

#### compound-engineering (provides: brainstorm, plan, work, review, compound)

```bash
claude plugin install EveryInc/compound-engineering-plugin
```

Or manually add to `~/.claude/settings.json` (merge into your existing keys — do **not** copy the whole block):

```json
{
  "extraKnownMarketplaces": {
    "compound-engineering-plugin": {
      "source": {
        "source": "github",
        "repo": "EveryInc/compound-engineering-plugin"
      }
    }
  },
  "enabledPlugins": {
    "compound-engineering@compound-engineering-plugin": true
  }
}
```

### Step 2: Install this plugin

```bash
claude plugin install kmorebetter/build-workflow
```

This installs for **your user** (all projects). To install for a specific project or team:

| Scope | Command | Settings file | Who gets it |
|-------|---------|---------------|-------------|
| User (default) | `claude plugin install ... --scope user` | `~/.claude/settings.json` | You, all projects |
| Project (shared) | `claude plugin install ... --scope project` | `.claude/settings.json` in repo | Everyone on the repo |
| Project (local) | `claude plugin install ... --scope local` | `.claude/settings.local.json` | Just you, just this repo |

Or manually add to `~/.claude/settings.json` (merge into your existing keys):

```json
{
  "extraKnownMarketplaces": {
    "build-workflow": {
      "source": {
        "source": "github",
        "repo": "kmorebetter/build-workflow"
      }
    }
  },
  "enabledPlugins": {
    "build-workflow@build-workflow": true
  }
}
```

### Step 3: Restart Claude Code

Restart your session. Verify the commands are available:

```
/build
```

You should see `/build`, `/build:think`, `/build:make`, `/build:ship` in your command list.

### Troubleshooting

| Problem | Fix |
|---------|-----|
| `/build` not showing up | Check `enabledPlugins` has `"build-workflow@build-workflow": true` |
| gstack skills not found | Verify `~/.claude/skills/gstack/` exists with skill directories inside |
| CE commands not found | Check `enabledPlugins` has `"compound-engineering@compound-engineering-plugin": true` |
| JSON parse error on restart | Ensure no comments (`//`) in `settings.json` — JSON does not support comments. Run `python3 -m json.tool ~/.claude/settings.json` to find syntax errors |

## Usage

### Full ceremony (new feature from scratch)

```
/build "Add user authentication with OAuth"
```

Runs all 11 steps with gates between phases. You approve each transition.

### Just validate an idea

```
/build:think "Refactor the payment system"
```

Stops after step 4. Writes a validated brief to `docs/briefs/`. Come back later with `/build:make`.

### Skip validation, start building

```
/build:make
```

Auto-detects the most recent brief from `docs/briefs/`. Or pass a description directly:

```
/build:make "Add rate limiting to the API — simple token bucket, 100 req/min per user"
```

### Code is ready, just ship it

```
/build:ship
```

Detects the current branch and open PR. Runs QA, captures learnings, deploys.

```
/build:ship 42
```

Target a specific PR by number.

## How It Works

The plugin is a **thin orchestrator**. It doesn't re-implement anything — it invokes gstack and compound-engineering skills in sequence with gates between phases.

### Phase gates

Each phase ends with a user decision:
- **Proceed** — move to the next phase
- **Revise** — loop back and refine
- **Kill** — stop, this isn't worth building

Gates are presented via AskUserQuestion. You stay in control.

### Artifact handoff

State passes between phases via the filesystem, not conversation context. This means you can `/build:think` today and `/build:make` tomorrow in a fresh session.

```
docs/briefs/       ← /build:think output (validated feature briefs)
docs/brainstorms/  ← CE brainstorming output (approach explorations)
docs/plans/        ← CE plan output (implementation plans)
docs/solutions/    ← CE compound output (captured learnings)
```

Each phase auto-detects artifacts from the previous phase.

### The compound loop

Step 10 writes learnings to `docs/solutions/`. Step 6 (CE plan) searches `docs/solutions/` for relevant prior art. So each build cycle makes the next one smarter:

```
Build 1: solve problem → document solution
Build 2: plan phase finds solution → avoids the same mistake
Build 3: even faster, compounding knowledge
```

## Customization

### Think phase (most personal)

Edit `commands/build-think.md` to change:
- **Interview questions** — adjust the 95% confidence interview in Step 1
- **Challenge topics** — change the devil's advocate pushback in Step 2
- **Gate criteria** — how strict the proceed/revise/kill decisions are

### Skip phases

Not every feature needs the full ceremony. Quick fixes:

```
/build:make "fix the broken email validation"   # Skip think, jump to building
/build:ship                                      # Skip think + make, just QA and ship
```

### Adding steps

The plugin is just markdown command files. To add a step (e.g., design review between steps 4 and 5):

1. Edit `commands/build-think.md` to add a new step
2. Push to GitHub
3. Run `claude plugin update build-workflow` to pull the change

## Uninstall

Remove from `~/.claude/settings.json`:

```jsonc
// Remove from enabledPlugins:
"build-workflow@build-workflow": true   // delete this line

// Remove from extraKnownMarketplaces:
"build-workflow": { ... }              // delete this block
```

Restart Claude Code. The `/build` commands will no longer appear.

This does NOT uninstall gstack or compound-engineering — those are independent tools you may want to keep.
