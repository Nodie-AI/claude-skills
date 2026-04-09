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
- check: {concrete verifiable statement about expected behavior}
- how: test
- status: pass | fail | unchecked
- evidence: {test name + result}

### 2. {criterion}
- check: ...
- how: test
- status: ...
- evidence:
```

## What makes criteria complete

**If a reasonable production scenario could break this code, there should be a test for it.** Not just the scenarios the developer intended — the ones that will actually happen.

Areas people consistently miss — make sure you've at least considered these:

- **Edge values**: None where you expect a value, empty string, zero, negative, unicode, max-length
- **Error paths**: DB down, API timeout, malformed response, disk full — does the code degrade gracefully or leak internals?
- **Upstream garbage**: the caller sends wrong types, missing required fields, extra fields, null where non-null expected
- **Concurrency**: two requests hit the same resource at the same time — 409 or unhandled 500?
- **Data integrity**: after the operation, is the data in the DB actually correct? Not just "returned 200" but "the row has the right values"
- **Silent failures**: code runs without errors, returns 200, but the feature doesn't actually work (e.g., `count or 0` returns 0 when count is None — no crash, but the feature is disabled)

This is not exhaustive. Think about what's specific to the code you're verifying — its dependencies, its callers, the data it touches.

## Field rules

**check**: One concrete statement. "Login works" is vague. "POST /login with valid credentials returns 200 + JWT; the JWT decodes to the correct user_id" is verifiable.

**how**: Always prefer `test` over `inspection`. Code inspection is not evidence — it's opinion.

**evidence**: Test name + pass/fail + relevant assertion output. Empty evidence with `pass` status is not allowed.
