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

## Phase 1c: Token Usage Analysis

After knowledge capture, analyze token usage to identify optimization opportunities.

### Process

1. **Get conversation ID** - Find the most recent conversation file:
   ```python
   import os
   from pathlib import Path

   project_path = Path.cwd()
   # Normalize path: /Users/foo/bar -> -Users-foo-bar
   normalized = str(project_path).replace('/', '-')
   if normalized.startswith('-'):
       normalized = normalized[1:]

   project_dir = Path.home() / '.claude/projects' / normalized
   if project_dir.exists():
       conversation_files = sorted(project_dir.glob('*.jsonl'),
                                  key=lambda f: f.stat().st_mtime,
                                  reverse=True)
       if conversation_files:
           conversation_id = conversation_files[0].stem  # UUID without .jsonl
   ```

2. **Run analysis** (only if conversation_id found):
   ```bash
   ~/bin/claude-cost analyze <conversation_id> --since-last --mark --top 15
   ~/bin/claude-cost bash <conversation_id> --since-last --top 10
   ```

3. **Identify high-cost operations** (anything >5000 tokens):
   - Test runs with verbose output
   - Linting with full output
   - Code searches (Grep/Glob) that could use language server
   - Bash commands with unnecessary verbosity
   - Large file reads that could be filtered

### Optimization Patterns

For each high-cost operation, consider creating a wrapper:

**Tests with verbose output** → Create quiet wrapper:
- Success: "✓ Tests passed (N passed)"
- Failure: Show only failing tests
- Example: See `~/src/claude-cost/run` command_test()

**Linting with full output** → Create quiet wrapper:
- Success: "✓ Linting passed"
- Failure: Show only violations
- Example: See `~/src/claude-cost/run` command_lint()

**Code search heavy** → Suggest language server or narrow search patterns

**Bash commands** → Create description-focused wrappers that capture output and summarize

### Creation Bias

For token optimization opportunities, **bias toward automatic creation**:
- Don't ask "Would you like me to create...?"
- Just create the wrapper tool/skill
- Report what was created afterward
- This is "aggressive improvement" mode for fast compounding optimization

Only skip creation if:
- The wrapper already exists
- The operation was already marked as "rejected" in the improve log
- There's genuine ambiguity about how to optimize it

### Quiet Test Runner Template

When creating a test wrapper, use this pattern:

```python
#!/usr/bin/env python3
import subprocess
import sys

result = subprocess.run(
    ['pytest', '-q', '--tb=short', '--disable-warnings'] + sys.argv[1:],
    capture_output=True,
    text=True
)

if result.returncode == 0:
    passed = result.stdout.count('passed')
    print(f'✓ Tests passed ({passed} passed)' if passed > 0 else '✓ Tests passed')
else:
    print('✗ Tests failed')
    print(result.stdout)
    if result.stderr:
        print(result.stderr, file=sys.stderr)

sys.exit(result.returncode)
```

### Output

Only report if optimization tools/skills were created. Don't report if no high-cost operations found.

---

## Phase 2: Tool & Skill Discovery

**Bias toward creating. A small, imperfect tool is better than no tool.**

Run through this checklist actively:

1. Did I repeat a multi-step process that could be a single command?
2. Did I write boilerplate that could be templated?
3. Did I discover a technique or pattern worth codifying as a reusable skill?
4. Did I do something manually that could be automated?
5. Did I have to look something up that a skill could encode as institutional knowledge?
6. Did I use tokens on verbose output that could be quieted? (from Phase 1c analysis)
7. Did I run the same command multiple times with noisy output?

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
{"timestamp": "2024-01-24T10:33:00Z", "type": "token_optimization", "project": "/path/to/project", "tool": "quiet-test", "saved_tokens_estimate": 5000}
```

**Create the output directory and log file on first run** if they don't exist.

---

## Output Format

At start: "Running improve..."

At end, only report positive actions:
- If project knowledge added: "✓ Added project knowledge: [brief summary]"
- If global advice added: "✓ Added global advice: [brief summary]"
- If tool/skill created: "✓ Created [tool/skill name]: [brief purpose]"

Do NOT report:
- "No new project knowledge to capture"
- "No new global advice"
- "No tool or skill opportunities identified"
- "Nothing to improve this session"

Silence is success. Only speak when something was created or improved.
