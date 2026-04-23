---
name: pre-audit
description: >
  Run a full security pre-audit on this repository. Performs AI code summary,
  AI security audit, npm audit, Semgrep OWASP Top 10 scan, and Slither analysis
  on Solidity files. Saves a markdown report. Use when asked to audit, review
  security, or run pre-audit. Accepts optional arguments: a branch/tag to diff
  against (diff mode), or "-f [branch/tag]" for full codebase mode.
argument-hint: "[-f] [branch-or-tag]"
disable-model-invocation: true
allowed-tools: >
  Bash(git *) Bash(npm audit*) Bash(semgrep *) Bash(slither *)
  Bash(find *) Bash(wc *) Bash(scc *) Bash(date *) Bash(tee *)
---

# Pre-Audit Security Skill

You are acting as a senior security auditor. Follow this playbook precisely to
produce a comprehensive security pre-audit report.

## 1 — Parse arguments

Arguments passed to this skill: `$ARGUMENTS`

- If arguments start with `-f`, run in **FULL MODE** (entire codebase).
  The remainder (if any) is a branch/tag to check out first.
- Otherwise, run in **DIFF MODE** comparing HEAD against the supplied
  branch/tag (or auto-detect one if none supplied).

## 2 — Gather context

### DIFF MODE

Run the following to collect diff data:

```bash
git rev-parse --short HEAD
git rev-parse --abbrev-ref @{-1} 2>/dev/null || echo ""
git rev-parse --verify origin/main >/dev/null 2>&1 && echo "origin/main" || \
  git rev-parse --verify main >/dev/null 2>&1 && echo "main" || \
  git rev-parse --verify origin/master >/dev/null 2>&1 && echo "origin/master" || \
  echo "HEAD^"
```

Then, using the detected or supplied reference (`$ARGUMENTS` if provided):

```bash
git diff <REF> HEAD --stat
git diff <REF> HEAD --name-only
git diff <REF> HEAD
```

Count lines added / removed from the diff output.

### FULL MODE

```bash
find . -type f \( -name "*.ts" -o -name "*.js" -o -name "*.tsx" -o -name "*.jsx" \
  -o -name "*.py" -o -name "*.sol" -o -name "*.go" -o -name "*.java" \
  -o -name "*.rs" -o -name "*.rb" -o -name "*.php" \) \
  ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/dist/*" ! -path "*/build/*" \
  | head -100
```

If `scc` is available: run `scc --exclude-dir=node_modules,dist,build,.git,vendor,out --no-cocomo`

Show first 30 lines of the 5 most important files to understand the codebase.

Truncate total code context to ~6,000 characters before sending to AI analysis.

## 3 — AI Code Summary

Based on the gathered diff or codebase snapshot, produce a markdown summary:

### For DIFF MODE:

```markdown
## 🚀 Key Changes
1. **[Feature/Fix]**: description
...

## 📁 Files Modified
- `file.ext`: what changed (one line)
...

## 💡 Summary
One paragraph on the overall purpose and impact.
```

### For FULL MODE:

```markdown
## 🏗️ Project Overview
What this project does.

## 🚀 Key Features
1. **[Feature]**: description
...

## 📁 Project Structure
- `folder/`: what it contains
...

## 💡 Summary
One paragraph on purpose, architecture, and tech stack.
```

## 4 — AI Security Audit

Analyse the same diff or codebase snapshot and produce:

```markdown
## 🔴 High-Impact Security Findings
- **File**: `path/to/file`
  **Issue**: description
  **Risk**: OWASP / CWE / blockchain category
  **Severity**: Critical | High | Medium
  **Recommendation**: short fix

(If none found: "No significant security issues detected.")

## 🟡 Potential Concerns
Patterns or practices to improve.

## Summary
One paragraph on overall security posture.
```

Focus on: injection flaws, auth issues, reentrancy, access control,
sensitive data exposure, insecure dependencies, hardcoded secrets.
Ignore whitespace/comment-only changes.

## 5 — npm audit

If `package.json` exists in the repo root:

```bash
npm audit 2>&1 | head -60
```

Otherwise note: `(npm audit skipped — no package.json found)`

## 6 — Semgrep OWASP Top 10

If `semgrep` is installed:

```bash
semgrep --config=p/owasp-top-ten . \
  --output owasp-semgrep-report.txt \
  --quiet 2>/dev/null || true
cat owasp-semgrep-report.txt 2>/dev/null || echo "(no findings)"
```

Otherwise note: `(semgrep skipped — run: pip install semgrep)`

## 7 — Slither (Solidity only)

Collect `.sol` files:
- DIFF MODE: only changed `.sol` files from the diff
- FULL MODE: `find . -name "*.sol" ! -path "*/node_modules/*"`

If any `.sol` files exist and `slither` is installed, for each file run:

```bash
slither <file> --print human-summary 2>/dev/null | head -20
```

Otherwise note: `(slither skipped — no .sol files or run: pip install slither-analyzer)`

## 8 — Build the report

Save the complete report to `pre-audit-report-<YYYY-MM-DD-HHmmss>.md` using
the structure below. Use `date +%Y-%m-%d-%H%M%S` for the timestamp.

```markdown
# [Full Codebase Security Audit Report | Pre-Pull Code Analysis Report]

## Date
<ISO 8601 timestamp>

## Mode
- **[Full Audit | Diff Audit]**
- Reference: <branch/tag or "n/a">
- Commit: <short hash>

## 📊 Lines of Code / Diff Stats
<scc output or diff line stats>

## [🏗️ Code Summary | 📋 Changes Summary]
<section 3 output>

## 🔒 Security Audit
<section 4 output>

## 📦 npm audit
\`\`\`
<npm audit output or skip note>
\`\`\`

## 🛡️ Semgrep – OWASP Top 10
\`\`\`
<semgrep output or skip note>
\`\`\`

## ⚡ Slither – Solidity Audit
<slither output per file, or skip note>
```

After saving, tell the user: `Report saved → pre-audit-report-<timestamp>.md`

## Notes

- If the diff is larger than ~6,000 characters, truncate it and note
  `[TRUNCATED — showing first 6,000 characters]`.
- If a tool is not installed, include a skip note with the install command;
  do not abort the entire audit.
- Commit `.claude/skills/pre-audit/` to version control to share this skill
  with your team.
