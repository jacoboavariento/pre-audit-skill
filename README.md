# pre-audit-skill

Run a full security pre-audit on this repository. Performs AI code summary, AI security audit, npm audit, Semgrep OWASP Top 10 scan, and Slither analysis on Solidity files. Saves a markdown report. Use when asked to audit, review security, or run pre-audit. Accepts optional arguments: a branch/tag to diff against (diff mode), or "-f [branch/tag]" for full codebase mode.

## Requirements
* semgrep
* slither
* scc


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
