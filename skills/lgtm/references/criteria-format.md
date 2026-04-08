# Criteria File Format

Save to `.ai-verify/criteria.md` in the project root.

```markdown
---
task: {short task description}
created: {YYYY-MM-DD}
status: in_progress | done
---

## Regression

### R1. {existing feature}
- check: {what to verify}
- how: test | command | inspection
- status: pass | fail | unchecked
- evidence: {proof}

## Criteria

### 1. {criterion}
- check: {what to verify — one concrete statement}
- how: test | command | file-exists | inspection
- status: pass | fail | unchecked
- evidence: {file:line, test name, or command output}
```

## Field rules

**check**: A single verifiable statement. "Login works" is too vague. "POST /login with valid credentials returns 200 and a JWT token" is concrete.

**how**: Determines verification method.
- `test` — run a specific test
- `command` — run a command and check output
- `file-exists` — check file or function exists (weakest)
- `inspection` — read code (use only when no better option exists)

**status**: Only set to `pass` with concrete evidence. If uncertain, use `unchecked`.

**evidence**: The proof. Include file path + line number, test name, or exact command output. Empty evidence with `pass` status is not allowed.
