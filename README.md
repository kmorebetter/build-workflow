# build-workflow

End-to-end feature workflow plugin for Claude Code. Three phases, eleven steps, zero yolo deploys.

## The Flow

| Step | Phase | Source | What Happens |
|------|-------|--------|--------------|
| 1 | Think | Custom | 95% confidence interview |
| 2 | Think | Custom | Devil's advocate challenge |
| 3 | Think | gstack | CEO/founder review gate |
| 4 | Think | gstack | Engineering review gate |
| 5 | Make | CE | Brainstorm approaches |
| 6 | Make | CE | Research + implementation plan |
| 7 | Make | CE | Execute with task tracking |
| 8 | Make | CE | Multi-agent code review |
| 9 | Ship | gstack | Browser-based QA testing |
| 10 | Ship | CE | Capture learnings for next time |
| 11 | Ship | gstack | Ship and deploy |

## Install

### 1. Prerequisites

**gstack** — installed as skills at `~/.claude/skills/gstack/`
Follow [garrytan/gstack](https://github.com/garrytan/gstack) installation instructions.

**compound-engineering** — installed as a Claude Code plugin.
Follow [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) installation instructions.

### 2. This Plugin

Push this repo to GitHub, then add to `~/.claude/settings.json`:

```jsonc
{
  "extraKnownMarketplaces": {
    // ... your existing marketplaces ...
    "build-workflow": {
      "source": {
        "source": "github",
        "repo": "YOUR_USERNAME/build-workflow"
      }
    }
  },
  "enabledPlugins": {
    // ... your existing plugins ...
    "build-workflow@build-workflow": true
  }
}
```

Restart Claude Code. You should see `/build`, `/build:think`, `/build:make`, `/build:ship` in your available commands.

## Usage

```bash
/build "Add OAuth login with Google"     # Full ceremony
/build:think "Refactor the payment API"  # Just validate the idea
/build:make                              # Build (auto-finds recent brief)
/build:ship                              # QA + deploy current branch
```

## How It Works

The plugin is a thin orchestrator. It doesn't re-implement anything — it invokes gstack and compound-engineering skills in sequence with gates between phases.

**Phase gates** are user decisions (proceed / revise / kill) presented via AskUserQuestion after each major step. You stay in control.

**Artifacts** (briefs, plans, solutions) flow between phases via the filesystem:
- `docs/briefs/` — validated feature briefs from think phase
- `docs/brainstorms/` — approach explorations from CE
- `docs/plans/` — implementation plans from CE
- `docs/solutions/` — captured learnings from CE compound

Each artifact is auto-detected by the next phase, so you can run `/build:think` today and `/build:make` tomorrow — context carries over via files.

## Customization

The most personal part of the workflow is the **think phase** (steps 1-2). Edit `commands/build-think.md` to change:
- Interview style and questions
- Challenge intensity and topics
- Confidence threshold (default: 95%)
