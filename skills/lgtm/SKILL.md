---
name: lgtm
description: "Looks Good To Me — write tests that find real bugs. Two modes: verify a task is done (acceptance tests), or hunt bugs in recent commits (adversarial tests). Reads git diffs, writes real test code, runs it, reports what breaks. Use after implementing a feature, before committing, or to audit recent changes."
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, Agent
---

# /lgtm

Find real bugs by **writing and running real tests**.

Two modes:
- **Verify**: "I built feature X — is it done?" → write acceptance tests
- **Hunt**: "What's broken in recent changes?" → read diffs, write adversarial tests

Both modes produce the same output: tests that either pass or fail, with concrete evidence.

## What you can write

- Test files: `tests/`, `__tests__/`, files matching `*.test.*`, `*.spec.*`, `*_test.*`
- Verification artifacts: `.ai-verify/` directory

Never write source code, docs, or config. Report failures — don't fix them.

## Mode 1: Verify (task acceptance)

When the user says `/lgtm` during or after a task:

1. Read conversation to understand what was built
2. Generate acceptance criteria
3. Write tests for each criterion — prefer integration tests, minimize mocks
4. Run the tests, report results
5. Save criteria to `.ai-verify/criteria.md`

## Mode 2: Hunt (bug finding)

When the user says `/lgtm` with "scan", "hunt", "check commits", or similar:

1. Run `git log --oneline -15` to find recent changes
2. For each interesting commit, run `git diff <commit>^..<commit>` to read the diff
3. Read the FULL source files that were changed (not just the diff — context matters)
4. Look for bug patterns (see checklist below)
5. Write **adversarial tests** that try to break the code
6. Run the tests — a failing test = confirmed bug

### The adversarial testing mindset

Acceptance tests ask "does it work?" Adversarial tests ask "how does it break?"

```python
# Acceptance test — confirms the happy path
def test_create_binding_201():
    r = client.post("/bindings", json=valid_payload)
    assert r.status_code == 201

# Adversarial test — finds the TOCTOU race condition
def test_create_binding_concurrent_race():
    """Two concurrent requests for same channel_id — does the second get
    409 (correct) or 500 (unhandled unique constraint)?"""
    # First request sees no existing row
    # Second request ALSO sees no existing row (race)
    # Both try INSERT — one gets unique violation
    monkeypatch.setattr(module, "async_execute", fake_that_raises_unique_violation)
    r = client.post("/bindings", json=same_payload)
    assert r.status_code in (409, 200), f"Got {r.status_code} — unhandled race"
```

The adversarial test is harder to write but catches real production bugs.

## Bug pattern checklist

When reading a diff, actively look for these. Each one maps to a specific test you can write.

### 1. None/null fallback traps

```python
# LOOKS SAFE:
count = result.count or 0

# BUG: if result.count is None (not 0), this silently becomes 0.
# Everything downstream that depends on count is wrong.
# Test: mock result.count = None, verify the code still works.
```

This is the #1 most common silent bug. Any `x or default` where x could be None/0/False/"" is suspicious.

### 2. Check-then-act (TOCTOU)

```python
existing = await db.select(...).eq("id", x)
if not existing.data:
    await db.insert(...)  # ← Another request can insert between SELECT and INSERT
```

Test: make the SELECT return empty, then make the INSERT raise a unique constraint exception. The code should handle it gracefully (409 or retry), not 500.

### 3. Error message info leaks

```python
except Exception as e:
    raise HTTPException(status_code=500, detail=f"DB error: {e}")
```

Test: make the DB raise an exception containing table names and schema info. Verify the API response exposes this to the caller.

### 4. String-based error detection

```python
if "23505" in str(err) or "unique" in str(err).lower():
```

Test: pass an error with the same meaning but different wording (e.g., "conflict: row already exists"). The detection fails, the error propagates as 500.

### 5. Function argument arity mismatch (JS)

```javascript
function helper(a, b, c) { ... }
helper(a, b, c, d, e);  // JS silently ignores d and e
```

Test: grep for function definitions and their call sites, count arguments. A mismatch suggests missing logic.

### 6. Cache staleness after writes

```javascript
await api.updateBinding(id, newWebhook);
cache.invalidate(id);  // Only invalidates THIS instance's cache
```

Test: verify that after an update, a subsequent read returns fresh data (not cached). If using multi-instance, the cache on other instances is stale for TTL duration.

### 7. Missing encodeURIComponent

```javascript
const url = `/api/users?id=${userId}`;  // userId could contain & or =
```

Test: grep for template literals in URL construction, check if parameters are encoded. Compare against other call sites in the same file that DO encode.

## Test quality standards

**Real calls, not mocks:**

```python
# BAD — mocks the thing you're testing
@patch('app.services.create_user')
def test_register(mock_create):
    mock_create.return_value = {"id": "123"}
    ...

# GOOD — calls real code through the API
def test_register():
    r = client.post("/register", json={"email": "a@b.com", "password": "Str0ng!"})
    assert r.status_code == 201
    assert r.json()["id"]  # real response
```

**Mocks ONLY for:**
- External HTTP APIs (Stripe, Discord, etc.)
- Databases that need credentials not in test env
- Simulating specific failure modes (DB returning None, unique violations)

**Match existing test patterns:** Read the project's existing test files first. Reuse their fixtures, test clients, and helper functions.

## Report format

```
LGTM Bug Hunt: saddlepoint-ai/backend (last 10 commits)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Scanned: 7 commits, 12 changed files
Tests written: tests/api/test_lgtm_hunt.py (8 tests)

BUGS FOUND:
  ✗ crash-loop count=None → detection disabled (HIGH)
    └ test_crash_loop_count_none FAILED
      result.count or 0 → always 0 when count is None
      Fix: use len(result.data) as fallback

  ✗ concurrent create_definition → unhandled 500 (MEDIUM)
    └ test_definition_concurrent_race FAILED
      unique constraint exception not caught

  ✗ DB error leaks table names in response (MEDIUM)
    └ test_db_error_info_leak PASSED (confirms leak)

VERIFIED OK:
  ✓ team reactivation preserves new config
  ✓ binding idempotent refresh returns 200
  ✓ ownership check runs before webhook creation

Summary: 3 bugs found, 3 verified OK, 0 unchecked
```

## Criteria and regression

See `references/criteria-format.md` for criteria file format.
See `references/regression.md` for regression suite management.
