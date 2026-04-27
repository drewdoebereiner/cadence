# Write Unit Tests

## Overview

Discover test framework and patterns from the codebase first, then write tests that match existing conventions. Never assume the stack -- read it.

## Step 1: Discover What Needs Testing

```bash
git diff HEAD~1  # for committed changes
git diff         # for unstaged changes
git diff --staged  # for staged changes
```

Build a list of changed files and functions. Focus on new behavior, not refactored internals.

## Step 2: Discover the Test Setup

Look for configuration files:
- Python: `pytest.ini`, `pyproject.toml`, `conftest.py`, `setup.cfg`
- JS/TS: `jest.config.*`, `vitest.config.*`, `package.json` (test script)

Read 1-2 existing test files in the same area as the changed code. Note:
- Naming conventions (`test_foo.py` vs `foo.test.ts`)
- Fixture patterns (pytest fixtures, Jest `beforeEach`, factories)
- Assertion style (`assert x == y` vs `expect(x).toBe(y)`)
- How external dependencies are handled (mocks, fakes, test containers)

## Step 3: Identify Coverage Gaps

For each changed function or class, check for:
- Happy path (valid input produces expected output)
- Each distinct error path
- Auth and permission failures if applicable
- External dependency failures (network down, DB unavailable)
- Boundary values (empty list, zero, max int, None/null)
- Invariants the code claims to maintain

## Step 4: Write Tests That Match Existing Patterns

- Match the naming scheme exactly
- Use the same fixture and factory approach as existing tests
- Match the assertion library and style
- Group tests logically -- one describe/class per unit under test
- Test behavior, not implementation details

## Coverage Checklist

- [ ] Valid input -- expected output
- [ ] Each error path exercised
- [ ] Auth failures tested if applicable
- [ ] External dependency failures simulated
- [ ] Boundary values covered (empty, zero, null, max)

## Running Tests

```bash
# Python
venv/bin/pytest path/to/test_file.py -v

# JS/TS
npm test -- path/to/test.file.ts
# or
npx vitest run path/to/test.file.ts
```

Verify new tests pass and no existing tests regress.

## Common Mistakes

| Mistake | Correct behavior |
|---|---|
| Assuming pytest when it might be unittest or Jest | Read config files first |
| Mocking everything | Match the project's existing approach to mocks |
| Testing implementation details | Test observable behavior and return values |
| Using system python | Use `venv/bin/python` and `venv/bin/pytest` |
| Writing tests before reading existing ones | Always read 1-2 existing tests first |
