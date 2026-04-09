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

## What makes a complete criteria set

A task is NOT verified unless the criteria cover ALL of these:

1. **Functional**: the feature does what was asked (happy path)
2. **Edge cases**: None, zero, empty string, max-length, unicode, negative numbers
3. **Error handling**: DB down, API timeout, invalid response format, disk full
4. **Defensive**: upstream caller sends garbage — wrong types, missing required fields, extra unexpected fields, null where non-null expected
5. **Concurrency**: two requests at the same time for the same resource — does it 409 or 500?
6. **Data integrity**: after the operation, is the data in the DB correct? Not just "did it return 200" but "is the row actually right?"

If you skip any category, you're not verifying — you're hoping.

## Field rules

**check**: One concrete statement. "Login works" is vague. "POST /login with valid credentials returns 200 + JWT; the JWT decodes to the correct user_id" is verifiable.

**how**: Always prefer `test` over `inspection`. Code inspection is not evidence — it's opinion.

**evidence**: Test name + pass/fail + relevant assertion output. Empty evidence with `pass` status is not allowed.
