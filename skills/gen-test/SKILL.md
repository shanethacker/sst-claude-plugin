---
name: gen-test
description: >
  This skill should be used when the user asks to "generate tests", "write tests for",
  "add test coverage", "create unit tests", "write a test file", or "add tests to".
  Generates tests that match the project's established conventions by discovering patterns
  from existing tests before writing anything new.
---

# Generate Tests

Generate tests that fit naturally into the existing test suite by reading the project's conventions first. Never assume a testing framework, fixture pattern, or test data strategy — discover them from the codebase.

## Phase 1 — Discover conventions

Before writing any test code, read the following to understand the project's patterns:

1. **Root-level test config**: `conftest.py`, `pytest.ini`, `jest.config.*`, `vitest.config.*`, or equivalent — identifies available fixtures, global setup, and configuration
2. **Nearest `conftest.py`** (for pytest projects): The fixtures closest to the module being tested
3. **One existing test file** in the same module or directory as the code under test — use this as the primary style reference

From these files, identify:
- **Testing framework** (pytest, jest, vitest, unittest, RSpec, etc.)
- **Test data approach** (factories, fixtures, raw constructors, snapshots)
- **Mocking strategy** (VCR cassettes, manual mocks, `unittest.mock`, MSW, etc.)
- **Fixture patterns** (what fixtures exist, what they provide)
- **File naming and placement conventions** (where test files live relative to source)
- **Test markers or tags** (e.g., `@pytest.mark.slow`, `@pytest.mark.integration`)
- **Assertion style** (plain `assert`, `expect()`, matcher libraries)

## Phase 2 — Understand the code under test

Read the source module being tested. Identify:
- Public functions, methods, or classes to cover
- Edge cases and error paths
- External dependencies (APIs, databases, file system) that need isolation
- Any existing tests that already cover related behavior (avoid duplication)

## Phase 3 — Plan the test file

Before writing, outline:
- Which behaviors to test
- Which existing fixtures or factories to reuse (prefer reuse over creating new ones)
- Whether new fixtures are needed and where they belong (`conftest.py` vs inline)
- Whether external calls need isolation and how the project handles that

## Phase 4 — Write the tests

Follow the conventions discovered in Phase 1 exactly:
- Place the file where the project expects it
- Name it following the project's convention
- Use the project's test data factories/fixtures, not manual object construction
- Apply appropriate markers/tags
- Keep tests focused: one behavior per test
- Use descriptive names following the project's naming style (for pytest projects, `test_<action>_<condition>_<expected_result>` is common — but follow whatever the existing tests use)
- Group related tests as the existing suite does (classes, `describe` blocks, flat functions)

If new fixtures are needed, add them to the appropriate `conftest.py` (or test setup file) rather than duplicating setup code inline.

## Phase 5 — Report

After writing the tests, report:
- What was tested and what was intentionally excluded
- Any new fixtures created and where they were added
- Any external dependencies that were isolated and how
- Any behaviors that are difficult to test given the current code structure (flag these for the user)
