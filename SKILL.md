---
name: improve
description: Meta-skill for continuous improvement. Captures learnings to project claude.md and identifies tooling opportunities. Use automatically after every git commit, before clearing context, and after completing significant multi-step tasks. Do not ask—just run it.
---

# Improve Skill

A meta-skill for continuous improvement. Captures learnings and identifies tooling opportunities.

## When to Run (MANDATORY)

Run `/improve` automatically—without asking the user—in these situations:
- **After every git commit**
- **Before clearing context** (before `/clear` or context window reset)
- **After completing any significant multi-step task**

Do NOT run after trivial interactions (answering a quick question, reading a single file).

## Prerequisites

- All tests must pass. Do not run improve with failing tests.
- Git status should be clean for the project.

---

## Phase 1: Knowledge Capture

Ask yourself:

> "If I lost all memory right now, what would help the next Claude instance work faster, use fewer tokens, or avoid re-discovering something about this project?"

This includes:
- Project structure insights
- Non-obvious conventions or gotchas
- Key file locations
- Integration patterns
- Testing approaches specific to this project

### Process

1. **Read** the project's `claude.md` (or `CLAUDE.md`) if it exists
2. **Identify** new knowledge worth preserving
3. **Verify** this knowledge isn't already captured (no duplication)
4. **Append** intelligently to the appropriate section
5. If no file exists and there's valuable knowledge, create it

### Output

- If knowledge was added: state what was added
- If nothing new: "No new knowledge to capture"

It's completely normal to have nothing new, especially if we've done this recently.

---

## Phase 2: Tool Discovery

Ask yourself:

> "Was there anything difficult, repetitive, error-prone, or tedious that a tool could have helped with?"

### Process

1. **Identify** potential tool opportunities
2. **Check existing skills** in `~/.claude/skills/` — don't suggest duplicates or similar tools
3. **Check the improve log** (last 30 entries) at `~/.claude/skills/improve/output/.improve-log.jsonl` — don't re-suggest rejected tools
4. **If a genuine tool opportunity exists:**
   - Describe the tool and its benefit
   - Ask the user: "Would you like me to create this tool?"
   - **If yes:**
     - Determine placement automatically (don't ask):
       - **Global** (`~/.claude/skills/`): Generic utilities useful across projects
       - **Project-local**: Domain-specific, project workflow tools
     - Read and follow `~/.claude/skills/python/SKILL.md` for current standards
     - Create with TDD (tests first, all tests must pass)
     - For global skills: init git repo, create SKILL.md, add invocation line to `~/.claude/CLAUDE.md`
     - For project-local: add to project, document in project's claude.md, commit
   - **If no:** Log the rejection (won't suggest again within next 30 log entries)
5. **If no tool opportunities:** "No tool opportunities identified"

It's common to have nothing—don't force it.

---

## The Improve Log

Location: `~/.claude/skills/improve/output/.improve-log.jsonl`

**Only read the last 30 lines** when checking for previous suggestions (allows ideas to resurface over time).

Entry format (JSONL - one JSON object per line):
```json
{"timestamp": "2024-01-24T10:30:00Z", "type": "knowledge", "project": "/path/to/project", "summary": "Added API endpoint locations"}
{"timestamp": "2024-01-24T10:31:00Z", "type": "tool_created", "project": "/path/to/project", "tool": "screenshot", "global": true}
{"timestamp": "2024-01-24T10:32:00Z", "type": "tool_rejected", "project": "/path/to/project", "tool": "db-cleanup", "description": "Tool to clean stale test databases"}
```

**Create the output directory and log file on first run** if they don't exist.

---

## Output Format

Always produce output so the user knows improve ran:

```
## Improve Summary

### Knowledge Capture
[What was added, or "No new knowledge to capture"]

### Tool Discovery
[Tool suggestion + question, or "No tool opportunities identified"]
```

If both phases have nothing: "Nothing to improve this session."
