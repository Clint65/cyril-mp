# Workflow: Test User Story

<purpose>
Create and run unit/integration tests for a completed user story.
Automatically fix code or tests until all tests pass.
Invoked by /test-us command.
</purpose>

<test_protocol>

<step name="analyze_story">
Gather implementation details:

1. Read story summary for:
   - Files created/modified
   - Acceptance criteria
   - Key functionality

2. Read source files to understand:
   - Public APIs to test
   - Edge cases
   - Dependencies
   - Error paths
</step>

<step name="detect_framework">
Identify project test setup:

**JavaScript/TypeScript:**
- Jest: `jest.config.*`, `__tests__/`
- Vitest: `vitest.config.*`
- Mocha: `mocha.opts`, `.mocharc.*`

**Python:**
- pytest: `pytest.ini`, `conftest.py`, `tests/`
- unittest: `test_*.py` pattern

**Go:**
- Built-in: `*_test.go` files

**Rust:**
- Built-in: `#[test]` annotations, `tests/`

Follow existing patterns exactly.
</step>

<step name="design_tests">
Map acceptance criteria to tests:

For each AC:
- Identify testable assertion
- Determine test type (unit/integration)
- Plan test data

Test categories:
1. **Unit tests**: Individual functions, isolated
2. **Integration tests**: Component interactions
3. **AC verification**: End-to-end AC validation

Present plan before creating.
</step>

<step name="create_tests">
Create test files following project conventions:

**Structure:**
```
describe('[Component/Module]', () => {
  describe('[function/method]', () => {
    it('should [expected behavior]', () => {
      // Arrange
      // Act
      // Assert
    });
  });
});
```

**Coverage targets:**
- Happy path (normal operation)
- Edge cases (empty, null, boundaries)
- Error handling (throws, rejects)
- AC validation (Given/When/Then)
</step>

<step name="run_tests">
Execute test suite:

```bash
# Framework-specific command
npm test -- --coverage    # Jest
pytest -v --cov           # pytest
go test -v ./...          # Go
cargo test                # Rust
```

Capture:
- Pass/fail counts
- Error messages
- Stack traces
- Coverage report (if available)
</step>

<step name="fix_failures">
For each failing test:

**1. Diagnose:**
- Is expectation correct per AC?
- Is implementation wrong?
- Is test setup wrong?

**2. Categorize:**
- Test bug: Fix test
- Implementation bug: Fix source
- Missing feature: Flag for review

**3. Fix:**
- Make minimal change
- Re-run single test
- Verify fix doesn't break others

**4. Iterate:**
- Maximum 5 attempts per failure
- Escalate if unable to fix
</step>

<step name="verify_coverage">
After all tests pass:

Map tests to acceptance criteria:
```
AC Coverage Verification:

AC1: [criterion]
  - test: [test name] - COVERED

AC2: [criterion]
  - test: [test name] - COVERED

AC3: [criterion]
  - test: [test name] - COVERED
```

If AC lacks coverage, add additional test.
</step>

<step name="report_results">
Final output:

```
════════════════════════════════════════
TESTS COMPLETE: US-XXX
════════════════════════════════════════

Results: [N] passing, 0 failing

Test Files:
- [path]: [N] tests
- [path]: [N] tests

AC Coverage: [N]/[N] criteria tested

Fixes Applied: [None | list]

Ready to commit with /commit-us
════════════════════════════════════════
```
</step>

</test_protocol>

<fix_priority>
When tests fail, fix in this order:

1. **Test setup errors** (imports, mocks, fixtures)
2. **Test logic errors** (wrong assertions, bad test data)
3. **Implementation bugs** (actual code issues)
4. **Missing implementation** (untested paths)

Always prefer fixing implementation over changing test expectations,
unless test expectation contradicts acceptance criteria.
</fix_priority>

<success_criteria>
- Test files created following project conventions
- All acceptance criteria have test coverage
- All tests passing
- Any code fixes documented
- Ready for commit
</success_criteria>
