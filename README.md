![](banner.jpg)

# Improve

A meta-skill for Claude Code that enables continuous improvement across coding sessions. It captures project knowledge and identifies opportunities for tooling automation.

## Purpose

The Improve skill serves two functions:

1. **Knowledge Capture** — Preserves insights about a project (structure, conventions, gotchas) in the project's `claude.md` file so future sessions start with context instead of rediscovering it.

2. **Tool Discovery** — Identifies repetitive, tedious, or error-prone workflows that could be automated with a new tool or skill.

## Installation

No installation required. The skill is located at `~/.claude/skills/improve/` and is invoked directly within Claude Code.

## Usage

Run the improve skill by typing:

```
/improve
```

### When to Run

Run `/improve` automatically in these situations:

- **After every git commit**
- **Before clearing context** (before `/clear` or context window reset)
- **After completing any significant multi-step task**

Do not run after trivial interactions like answering a quick question or reading a single file.

### Prerequisites

Before running:

- All tests must pass
- Git status should be clean

## Examples

### After Completing a Feature

```
user: [completes implementing user authentication]
assistant: [commits the changes]
assistant: /improve

## Improve Summary

### Knowledge Capture
Added to project claude.md: Authentication uses JWT tokens stored in httpOnly cookies. 
Auth middleware located at src/middleware/auth.ts.

### Tool Discovery
No tool opportunities identified.
```

### When a Tool Opportunity is Found

```
user: [finishes a session involving repeated manual screenshot verification]
assistant: /improve

## Improve Summary

### Knowledge Capture
No new knowledge to capture.

### Tool Discovery
Identified repetitive screenshot capture for UI verification. 
Would you like me to create a screenshot automation tool?
```

### Nothing to Improve

```
assistant: /improve

## Improve Summary

Nothing to improve this session.
```

## Output

The skill always produces output so you know it ran:

```
## Improve Summary

### Knowledge Capture
[What was added, or "No new knowledge to capture"]

### Tool Discovery
[Tool suggestion + question, or "No tool opportunities identified"]
```

## License

This project is licensed under [CC BY-NC 4.0](https://darren-static.waft.dev) - free to use and modify, but no commercial use without permission.
