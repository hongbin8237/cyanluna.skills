---
name: model-route
description: >
  Model routing guidelines for Task tool subagents.
  Routes tasks to optimal Claude model (Haiku/Sonnet/Opus) via Task tool's model parameter.
  NOT auto-triggered. Use as a reference when delegating work to subagents.
---

# Model Routing Guide

This skill provides model routing guidelines for cost optimization when using the Task tool.

## How It Works

Claude Code's Task tool has a built-in `model` parameter that routes subagents to different models **without modifying global settings**. This is the correct and safe way to route models.

```
Task(subagent_type="...", model="haiku|sonnet|opus", prompt="...")
```

## Quick Decision Tree

```
Is the task simple (Q&A, single file, typo fix, git ops)?
  -> Yes: model="haiku"
  -> No: Does it require analysis of 1-5 files or moderate code changes?
    -> Yes: model="sonnet"
    -> No: Is it deep architecture, 6+ files, or complex system design?
      -> Yes: model="opus" (or omit -- it's the default)
```

## Routing Table

| model    | When to use                                          | Cost  |
|----------|------------------------------------------------------|-------|
| `haiku`  | Git ops, file reads, typo fixes, simple Q&A, configs | $     |
| `sonnet` | Code analysis, feature impl, bug fix, test writing   | $$    |
| `opus`   | Architecture, large refactor, security, system design | $$$$  |

## Important

- **Do NOT modify `~/.claude/settings.json`** to switch models
- **Do NOT auto-trigger** on every request -- only apply when using Task tool
- **Main conversation always uses the user's configured model** -- only subagents are routed
- **When in doubt, choose the lower-cost model** and escalate only if needed

## Project-Specific Customizations

Each project can add a **Model Selection** section to CLAUDE.md to override these defaults:

```markdown
## Model Selection for This Project

### Haiku (Fast)
- Project-specific simple tasks

### Sonnet (Balanced, Recommended Default)
- Project-specific medium tasks

### Opus (Deep)
- Project-specific complex tasks
```

**Priority order:**
1. Explicit override: `[Opus] task` or `[Haiku] task` -> Use specified model
2. Project guide: Check CLAUDE.md -> Follow if available
3. This routing table -> Fallback

## Full Guidelines

See the project CLAUDE.md "Model Routing Guidelines" section for detailed tier definitions, examples, and scenario-based quick reference.
