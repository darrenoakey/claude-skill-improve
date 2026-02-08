---
name: improve
description: Meta-skill for continuous improvement. Captures project and global learnings, and aggressively identifies tool and skill opportunities. Use automatically after every git commit, before clearing context, and after completing significant multi-step tasks. Do not ask—just run it.
---

# Improve Skill

A meta-skill for continuous improvement. Captures learnings at project and global levels, and creates tools and skills from patterns worth codifying.

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

### 1a: Project Knowledge

Ask yourself:

> "If I lost all memory right now, what would help the next Claude instance work faster, use fewer tokens, or avoid re-discovering something about this project?"

This includes:
- Project structure insights
- Non-obvious conventions or gotchas
- Key file locations
- Integration patterns
- Testing approaches specific to this project

#### Process

1. **Read** the project's `claude.md` (or `CLAUDE.md`) if it exists
2. **Identify** new knowledge worth preserving
3. **Verify** this knowledge isn't already captured (no duplication)
4. **Append** intelligently to the appropriate section
5. If no file exists and there's valuable knowledge, create it

#### Output

- If knowledge was added: state what was added
- If nothing new: "No new project knowledge to capture"

It's completely normal to have nothing new, especially if we've done this recently.

### 1b: Global Advice

Ask yourself:

> "Did I learn something this session that applies across ALL projects — not just this one?"

This includes:
- Gotchas with common tools (git, docker, npm, brew, etc.)
- Patterns that prevent bugs universally
- Environment/OS quirks
- CLI flags or behaviors that are non-obvious
- Workflow improvements that apply everywhere

#### Process

1. **Identify** cross-project learnings from this session
2. **Read** `~/.claude/CLAUDE.md`
3. **Check** the `## Learned Advice` section (at the bottom, before Gemini memories) for duplication
4. **If new and valuable:** append a one-liner with context to the `## Learned Advice` section
5. **If the section doesn't exist:** create it above the `## Gemini Added Memories` line
6. **Log** to improve log with type `"global_advice"`

#### Output

- If advice was added: state what was added
- If nothing new: "No new global advice"

---

## Phase 2: Tool & Skill Discovery

**Bias toward creating. A small, imperfect tool is better than no tool.**

Run through this checklist actively:

1. Did I repeat a multi-step process that could be a single command?
2. Did I write boilerplate that could be templated?
3. Did I discover a technique or pattern worth codifying as a reusable skill?
4. Did I do something manually that could be automated?
5. Did I have to look something up that a skill could encode as institutional knowledge?

If ANY answer is yes, you have a candidate. There are two types of output:

- **Tool** = executable utility (script in `~/bin/` or `~/.claude/skills/*/`). Has code that runs.
- **Skill** = SKILL.md with codified knowledge, technique, or behavioral pattern in `~/.claude/skills/`. Skills don't need executable code — they can be pure documentation that guides future Claude sessions.

### Process

1. **Identify** potential tool or skill opportunities using the checklist above
2. **Check existing skills** in `~/.claude/skills/` — don't suggest duplicates or similar tools
3. **Check the improve log** (last 30 entries) at `~/.claude/skills/improve/output/.improve-log.jsonl` — don't re-suggest rejected items
4. **If a genuine opportunity exists:**
   - Describe the tool or skill and its benefit
   - Ask the user: "Would you like me to create this tool/skill?"
   - **If yes — Tool:**
     - Determine placement automatically (don't ask):
       - **Global** (`~/.claude/skills/`): Generic utilities useful across projects
       - **Project-local**: Domain-specific, project workflow tools
     - Read and follow `~/.claude/skills/python/SKILL.md` for current standards
     - Create with TDD (tests first, all tests must pass)
     - For global skills: init git repo, create SKILL.md, add invocation line to `~/.claude/CLAUDE.md`
     - For project-local: add to project, document in project's claude.md, commit
   - **If yes — Skill:**
     - Create directory in `~/.claude/skills/<name>/`
     - Write `SKILL.md` with frontmatter (name, description) and codified knowledge
     - Init git repo
     - Add to `~/.claude/CLAUDE.md` if warranted
     - Log type: `"skill_created"`
   - **If no:** Log the rejection (won't suggest again within next 30 log entries)
     - Log type: `"tool_rejected"` or `"skill_rejected"`
5. **If no opportunities after the checklist:** "No tool or skill opportunities identified"

---

## The Improve Log

Location: `~/.claude/skills/improve/output/.improve-log.jsonl`

**Only read the last 30 lines** when checking for previous suggestions (allows ideas to resurface over time).

Entry format (JSONL - one JSON object per line):
```json
{"timestamp": "2024-01-24T10:30:00Z", "type": "knowledge", "project": "/path/to/project", "summary": "Added API endpoint locations"}
{"timestamp": "2024-01-24T10:30:00Z", "type": "global_advice", "project": "/path/to/project", "summary": "Added git rebase gotcha to CLAUDE.md"}
{"timestamp": "2024-01-24T10:31:00Z", "type": "tool_created", "project": "/path/to/project", "tool": "screenshot", "global": true}
{"timestamp": "2024-01-24T10:31:30Z", "type": "skill_created", "project": "/path/to/project", "skill": "debugging", "summary": "Codified systematic debugging approach"}
{"timestamp": "2024-01-24T10:32:00Z", "type": "tool_rejected", "project": "/path/to/project", "tool": "db-cleanup", "description": "Tool to clean stale test databases"}
{"timestamp": "2024-01-24T10:32:30Z", "type": "skill_rejected", "project": "/path/to/project", "skill": "api-patterns", "description": "REST API design patterns"}
```

**Create the output directory and log file on first run** if they don't exist.

---

## Output Format

Always produce output so the user knows improve ran:

```
## Improve Summary

### Knowledge Capture (Project)
[What was added to project claude.md, or "No new project knowledge to capture"]

### Knowledge Capture (Global)
[What was added to ~/.claude/CLAUDE.md, or "No new global advice"]

### Tool & Skill Discovery
[Tool/skill suggestion + question, or "No tool or skill opportunities identified"]
```

If all three phases have nothing: "Nothing to improve this session."
