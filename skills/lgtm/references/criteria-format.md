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

### 1. {functional requirement}
- check: {concrete verifiable statement}
- how: test
- status: pass | fail | unchecked
- evidence: {test name + result}

### 2. {edge case / boundary}
- check: {what happens with None, empty, duplicate, max-length, etc.}
- how: test
- status: pass | fail | unchecked
- evidence:

### 3. {error handling}
- check: {what happens when DB fails, API 500s, input is garbage}
- how: test
- status: pass | fail | unchecked
- evidence:

### 4. {defensive — upstream sends bad data}
- check: {what if the caller sends None, wrong type, extra fields, missing fields}
- how: test
- status: pass | fail | unchecked
- evidence:

### 5. {concurrency / race condition}
- check: {what if two requests hit this at the same time}
- how: test
- status: pass | fail | unchecked
- evidence:
```

## What makes criteria complete

One principle: **if a reasonable production scenario could break this code, there should be a test for it.**

Ask yourself: "what inputs, states, and timing conditions could this code encounter in production?" Then write tests for those. Not just the ones the developer intended — the ones that will actually happen.

Some areas people consistently miss:
- What if a value that's "always there" is None?
- What if two requests hit the same resource at the same time?
- What if the upstream caller sends something slightly wrong?
- What does the error response actually contain — does it leak internals?
- After the operation, is the data correct, or just the status code?

This is not an exhaustive list. Think about what's specific to the code you're verifying.

## Field rules

**check**: One concrete statement. "Login works" is vague. "POST /login with valid credentials returns 200 + JWT; the JWT decodes to the correct user_id" is verifiable.

**how**: Always prefer `test` over `inspection`. Code inspection is not evidence — it's opinion.

**evidence**: Test name + pass/fail + relevant assertion output. Empty evidence with `pass` status is not allowed.
