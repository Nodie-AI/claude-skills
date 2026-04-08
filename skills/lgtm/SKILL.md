---
name: lgtm
description: "Looks Good To Me — verify whether the current task is actually done. Generates acceptance criteria, checks them by running tests and inspecting code, catches regressions in existing features, reports what passes, fails, or remains untested. Use after implementing a feature, before committing, or anytime you want an honest progress check."
allowed-tools: Read, Grep, Glob, Bash, Agent
---

# /lgtm

Answer one question: **is the current task actually done?**

This skill only reads code and runs tests. It never writes source code, docs, or config. If something fails, it reports the problem — the main agent or the user fixes it. Verification that also fixes defeats the purpose: it hides the gap between "agent says done" and "actually done."

The only files this skill creates are inside `.ai-verify/` (criteria, reports, regression plans).

## Workflow

### First call — generate criteria

1. Read conversation to understand the task
2. Scan the project for existing features that could regress (check README, CLAUDE.md, git log, existing tests)
3. Generate concrete acceptance criteria — each one must be checkable with a test, a command, or file inspection
4. Save to `.ai-verify/criteria.md`
5. Show the criteria list with 0/N progress

### Subsequent calls — check progress

1. Load `.ai-verify/criteria.md`
2. **Regression first**: run existing test suite (`npm test`, `pytest`, etc.) and check `.ai-verify/regression/` plans. If anything regressed, report it before checking new criteria.
3. **Then new criteria**: check each one using the most concrete method available — run a test, run a command, or inspect code (in that priority order)
4. Update status in criteria.md
5. Show progress report

### When all pass — save to regression

Offer to save passing criteria to `.ai-verify/regression/{feature}.test-plan.md` so future changes can't silently break this feature.

## What counts as evidence

A criterion passes only with concrete proof:

- Test output: `"37 passed, 0 failed"` — strong
- Command output: `curl localhost:3000 → 200` — strong
- Build succeeds: `tsc --noEmit → exit 0` — strong
- File/function exists: `grep -q "function handleAuth"` — acceptable for structural checks
- "I read the code and it looks right" — **not acceptable**, mark as `unchecked`
- Visual/UI checks — mark as `unchecked (needs user confirmation)`

Be honest. `unchecked` is better than a false `pass`.

## Example session

User has been building a registration feature. They run `/verify`.

First call output:

```
Verification: User registration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

REGRESSION:
  ✓ Existing test suite passes (npm test → 42 passed)
  ✓ Build clean (tsc --noEmit → exit 0)

NEW CRITERIA:
  ✓ POST /register returns 201
    └ tests/auth.test.ts → test_register_success passes
  ✗ Passwords hashed with bcrypt
    └ src/models/user.ts:12 — stores plaintext password
  ○ Verification email sent
    └ unchecked — needs manual test (no email service in test env)

Progress: ●●○ 2/3 (1 unchecked)

---
1 item needs fixing:

**Passwords hashed with bcrypt** — FAIL
  Current: `user.password = req.body.password` at src/models/user.ts:12
  Expected: hash with bcrypt before storing
```

Second call (after fix):

```
Verification: User registration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

REGRESSION: 2/2 ✓
NEW CRITERIA: 2/3 ✓ (1 unchecked)

  ✓ POST /register returns 201
  ✓ Passwords hashed with bcrypt
    └ src/models/user.ts:12 — now uses bcrypt.hash()
  ○ Verification email sent (needs user confirmation)

All checkable criteria pass. Save to regression suite? (y/n)
```

## Criteria file format

See `references/criteria-format.md` for the full format spec.

## Regression

See `references/regression.md` for how regression checking and the persistent test suite work.
