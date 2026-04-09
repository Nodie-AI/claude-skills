---
name: lgtm
description: "Looks Good To Me — write tests to verify the current task is done. Generates acceptance criteria, writes real test code (unit + integration), runs them, catches regressions, produces structured verification reports. Use after implementing a feature, before committing, or anytime you want an honest progress check."
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, Agent
---

# /lgtm

Verify the current task by **writing and running real tests**.

The core loop: criteria → write tests → run tests → report. Not: criteria → read code → guess.

## What you can write

- Test files: `tests/`, `__tests__/`, files matching `*.test.*`, `*.spec.*`, `*_test.*`
- Verification artifacts: `.ai-verify/` directory (criteria, reports, regression plans)

Do not write to source code, docs, or config. If you need to change source code to fix a failing test, report the failure — the main agent or user fixes it.

## Workflow

### First call — define criteria, write tests

1. Read conversation to understand the task
2. Generate acceptance criteria — each one maps to a real test
3. **Write test code** that verifies each criterion:
   - Match the project's existing test framework and patterns (look at existing test files first)
   - Prefer integration tests over unit tests — call real functions, hit real endpoints
   - Minimize mocks. Only mock external services (HTTP APIs, databases that need credentials). Never mock the code under test.
   - Each test should be runnable independently
4. Run the tests
5. Save criteria to `.ai-verify/criteria.md`, show results

### Subsequent calls — run and report

1. Load criteria from `.ai-verify/criteria.md`
2. **Regression first**: run existing test suite, check `.ai-verify/regression/` plans
3. Run the tests you wrote for current criteria
4. Report what passes, fails, remains untested
5. If a test fails, report the failure with enough detail to fix it

### When all pass — save to regression

1. Offer to save criteria + test files to `.ai-verify/regression/`
2. The test files persist — next time any agent works on this codebase, these tests run automatically

## Test quality standards

**Prefer real calls over mocks:**

```python
# BAD — mocks everything, proves nothing
@patch('app.services.user_service.create_user')
def test_register(mock_create):
    mock_create.return_value = {"id": "123"}
    result = register({"email": "a@b.com"})
    assert result.status == 201

# GOOD — calls real code, catches real bugs
def test_register():
    result = client.post("/register", json={"email": "a@b.com", "password": "Str0ng!"})
    assert result.status_code == 201
    assert "id" in result.json()
    # Verify side effect: user actually exists in DB
    user = db.query(User).filter_by(email="a@b.com").first()
    assert user is not None
    assert user.password != "Str0ng!"  # should be hashed
```

**When mocks ARE appropriate:**
- External HTTP APIs (Stripe, Discord, etc.)
- Services requiring credentials not available in test env
- Time-dependent behavior (freeze time instead of waiting)

**Test patterns to use:**
- Find the project's existing test files, match their style (fixtures, client setup, DB setup)
- If the project has a test client (FastAPI TestClient, supertest, etc.), use it
- If the project has fixtures or factories, reuse them
- Test both happy path AND error cases (invalid input, 404, 409, 500)
- Test concurrent scenarios if the code does check-then-act patterns

## Scanning new/changed code

When checking criteria against a diff or recent commit:

1. Read the diff (`git diff` or `git show`)
2. For each changed function, look for:
   - Check-then-act without locking (TOCTOU race conditions)
   - Exception handling that swallows errors or leaks info
   - URL/query construction without encoding
   - Cache invalidation gaps (write happens but cache not cleared)
3. Write tests that specifically target these patterns
4. A test that catches a real bug is worth 10 tests that confirm happy paths

## Example session

User built a registration feature, runs `/lgtm`:

```
Verification: User registration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Writing tests...
  Created tests/test_registration_verify.py (4 tests)

REGRESSION:
  ✓ Existing test suite passes (pytest → 42 passed)

NEW CRITERIA:
  ✓ POST /register returns 201 with user object
    └ test_register_success PASSED (23ms)
  ✓ Duplicate email returns 409
    └ test_register_duplicate_email PASSED (15ms)
  ✗ Password stored as bcrypt hash
    └ test_password_hashed FAILED
      AssertionError: user.password == 'plaintext123'
      Expected: bcrypt hash (starts with $2b$)
  ○ Verification email sent
    └ test_verification_email SKIPPED (no email service in test env)

Progress: ●●✗○ 2/4

Fix needed:
  Password is stored as plaintext at src/models/user.py:12
  Expected: bcrypt.hash(password) before save
```

## Criteria file format

See `references/criteria-format.md` for the full spec.

## Regression

See `references/regression.md` for how regression checking and the persistent test suite work.
