---
name: verify
description: "Looks Good To Me — write thorough tests that verify work is done and catch bugs. Reads code and git diffs, writes real test code covering happy paths + edge cases + error handling + concurrency, runs tests, reports results. Use after implementing a feature, before committing, or to verify recent changes."
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, Agent
---

# /verify

Write and run tests that prove the work is done and behaves correctly.

A passing test means the code does what it should. A failing test means you found a bug. But be careful: **a test that passes because the code "doesn't crash" is not the same as a test that passes because the code "does the right thing."** Always assert on the expected business outcome, not just the absence of errors.

For example: if a crash-loop detector returns `count=0` because a field was None, the code didn't crash — but the feature is silently disabled. The test should assert `count >= 3` (the real expected value), not just `status_code == 200`.

## What you can write

- Test files: `tests/`, `__tests__/`, `*.test.*`, `*.spec.*`, `*_test.*`
- Verification artifacts: `.ai-verify/`

Never write source code, docs, or config. Report failures — don't fix them.

## Workflow

1. **Understand what changed**: read the conversation, or run `git log` + `git diff` to see recent changes
2. **Read the full source files** that were changed — not just diffs, you need context
3. **Read existing test files** to match the project's patterns, fixtures, and test client setup
4. **Write tests** that cover:
   - The stated requirements (does the feature work?)
   - Edge cases (what happens with None, empty, duplicate, concurrent?)
   - Error paths (what happens when the DB fails, the API returns 500, the input is malformed?)
   - The specific bug patterns listed below
5. **Run the tests**
6. **Report results** — passing tests are verified requirements, failing tests are confirmed bugs
7. Save to `.ai-verify/criteria.md` and offer to persist as regression tests

## How to write tests that actually find bugs

Every test you write should make one of these two statements:
- "This requirement is met" (test passes)
- "This is a bug" (test fails)

If you're writing a test and you already know it will pass, ask yourself: am I testing anything meaningful, or just rewriting the implementation as an assertion?

**Test real code, not mocks:**

```python
# This proves nothing — you're testing your own mock
@patch('app.services.create_user')
def test_register(mock_create):
    mock_create.return_value = {"id": "123"}
    result = register({"email": "a@b.com"})
    assert result.status == 201

# This catches real bugs — calling through the actual stack
def test_register():
    r = client.post("/register", json={"email": "a@b.com", "password": "Str0ng!"})
    assert r.status_code == 201
    assert r.json()["id"]
```

Mocks only for: external HTTP APIs, services needing unavailable credentials, simulating specific failure modes.

## Bug patterns to test for

When reading changed code, look for these and write tests that target them. Each pattern has caused real production bugs.

### None/null fallbacks

`count = result.count or 0` — if `result.count` is None (not zero), this silently becomes 0. Everything downstream is wrong. Test by mocking the return value as None.

### Check-then-act without locking

SELECT to check existence → INSERT if not found. Two concurrent requests both see "not found", both INSERT, one gets unique constraint violation that's unhandled → 500. Test by making the INSERT raise a constraint exception.

### Error message leaks

`raise HTTPException(detail=f"DB error: {e}")` — raw exception text in the response leaks table names, schema, connection info. Test by raising an exception with sensitive info and checking the response body.

### Fragile string matching

`if "unique" in str(err).lower()` — breaks when the error message format changes. Test by passing an error with equivalent meaning but different wording.

### Argument count mismatch (JS)

Function defined with 4 params, called with 5 args — JS silently drops extras. The 5th arg was probably meant to be used. Verify by reading call sites.

### Cache not invalidated across instances

Write updates the DB but only clears local cache. Other instances serve stale data for TTL duration. Test by verifying reads after writes return fresh data.

### Missing URL encoding

`/api?id=${userId}` without `encodeURIComponent` — breaks if userId contains `&`, `=`, `#`. Check for inconsistency: if some call sites encode and others don't, the non-encoding ones are bugs.

### Silent business logic failure

Code runs without errors, returns 200, but the feature doesn't actually work. Example: `count = result.count or 0` returns 0 when count is None — the API responds 200 with `is_crash_loop: false` even though there were 5 crashes. The code is "working" but the feature is disabled. **Always verify the returned values make business sense, not just that the status code is 2xx.**

## Report format

```
LGTM: feature-name
━━━━━━━━━━━━━━━━━━

Tests written: tests/test_lgtm_feature.py (8 tests)
Existing suite: pytest → 42 passed

Results:
  ✓ POST /register returns 201              test_register_success PASSED
  ✓ Duplicate email returns 409             test_register_dup PASSED
  ✗ Password stored as hash                 test_password_hash FAILED
    └ user.password == 'plaintext' — not hashed
  ✗ Concurrent create returns 409           test_concurrent_race FAILED
    └ unhandled unique constraint → 500
  ✓ Invalid email returns 422               test_register_invalid PASSED

5 tests: 3 passed, 2 failed (2 bugs found)
```

## Criteria and regression

See `references/criteria-format.md` and `references/regression.md`.
