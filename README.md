# pre-audit-skill


## Usage:
Drop it into your repo at `.claude/skills/pre-audit/SKILL.md` and you're good to go.
How to use it in Claude Code:
```
/pre-audit                  # diff mode, auto-detect base branch
/pre-audit main             # diff mode against main
/pre-audit v2.0.0           # diff mode against a tag
/pre-audit -f               # full codebase audit
/pre-audit -f develop       # full audit of the develop branch
```
