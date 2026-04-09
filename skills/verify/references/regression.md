# Regression Checking

Regression protection ensures new work doesn't break existing features. This is the most important part of verification.

## Check order

Every `/lgtm` call:

1. Run the project's existing test suite first (npm test, pytest, etc.)
2. Check `.ai-verify/regression/*.test-plan.md` if they exist
3. Only then check new task criteria

A regression failure blocks the entire verification — even if all new criteria pass.

## How to find existing features

On the first `/verify` call, scan for what already works:

- Existing test files → features they cover are regression candidates
- README/CLAUDE.md → described features
- `git log --oneline -20` → recent completed work
- Running `npm test` / `pytest` → what the test suite covers

Turn these into regression criteria (prefixed R1, R2, etc.).

## Saving to regression suite

When all criteria pass and the user confirms, save to `.ai-verify/regression/{feature-name}.test-plan.md`:

```markdown
---
plan: {feature-name}
status: implemented
created: {YYYY-MM-DD}
---

## {module}: {description}

### {module}_test_{behavior}
- acceptance: {what was verified}
- type: unit | integration | e2e
- file: {test file if exists}
- expected: {expected outcome}
```

## Reporting regressions

When a previously passing feature breaks:

```
⚠ REGRESSION: 1 existing feature broken

  ✗ R1. Test suite passes
    └ npm test → 3 failed (was 0)
    └ Failing: auth.test.ts, payment.test.ts

This must be fixed before new criteria can be evaluated.
```
