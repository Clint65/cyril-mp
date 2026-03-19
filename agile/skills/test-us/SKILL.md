---
name: test-us
description: Create and run tests for a completed user story, fixing code if tests fail. Use when the user wants to test a story, verify acceptance criteria, generate test coverage for a story, or validate implementation. Triggers on "test story", "verify story", "add tests for story", or after story completion.
argument-hint: <story-path>
context: fork
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Agent
---

Create and run tests for the user story at $ARGUMENTS.

<objective>
Generate unit and integration tests for a completed user story.
Run tests and automatically fix code if tests fail.
Ensure story implementation meets acceptance criteria through testing.
</objective>

<process>

**1. Locate story files:**

Find summary and related files:
```bash
ls -la $(dirname $ARGUMENTS)/
```

Read the summary to understand what was implemented:
```bash
# Find and read summary
cat $(dirname $ARGUMENTS)/US-*-SUMMARY.md 2>/dev/null || cat $ARGUMENTS
```

Extract from summary:
- Story ID and title
- Acceptance criteria
- Files created/modified
- Key functionality implemented

**2. Analyze implementation:**

Read the source files mentioned in the summary to understand:
- Functions/methods to test
- Edge cases to cover
- Integration points
- Dependencies

**3. Detect test framework:**
```bash
# Check for existing test setup
ls package.json pyproject.toml setup.py Cargo.toml go.mod 2>/dev/null

# Check for test directories
ls -d test/ tests/ __tests__ spec/ 2>/dev/null

# Check for existing test patterns
find . -name "*test*" -o -name "*spec*" | head -20
```

Determine:
- Language/framework (Jest, pytest, Go testing, etc.)
- Test directory structure
- Naming conventions
- Existing test patterns to follow

**4. Plan test coverage:**

Present test plan to user:
```
════════════════════════════════════════
TEST PLAN: US-XXX
════════════════════════════════════════

Story: [title]
Files to test: [list]

Unit Tests:
- [function/method]: [what to test]
- [function/method]: [what to test]

Integration Tests:
- [flow]: [what to test]

Acceptance Criteria Coverage:
- AC1: [test approach]
- AC2: [test approach]
- AC3: [test approach]

Test framework: [detected framework]
Test location: [where tests will be created]
════════════════════════════════════════
```

Proceed to create the tests.

**5. Create test files:**

For each file to test, create corresponding test file:
- Follow existing naming conventions
- Import necessary dependencies
- Create test structure matching framework

Unit tests should cover:
- Happy path for each function
- Edge cases (empty, null, boundary values)
- Error handling paths

Integration tests should cover:
- End-to-end flows from acceptance criteria
- Component interactions
- API contracts (if applicable)

**6. Run tests:**
```bash
# Run appropriate test command based on framework
npm test        # Node.js
pytest          # Python
go test ./...   # Go
cargo test      # Rust
```

Capture output and analyze results.

**7. Handle test failures:**

If tests fail:
```
════════════════════════════════════════
TEST RESULTS: FAILURES DETECTED
════════════════════════════════════════

[N] tests passed, [M] tests failed

Failures:
1. [test name]: [error message]
2. [test name]: [error message]

Analyzing failures to determine fix...
════════════════════════════════════════
```

For each failure, determine if issue is in:
- **Test code**: Fix the test
- **Implementation code**: Fix the source

Apply fixes automatically:
1. Identify root cause
2. Make minimal fix
3. Re-run affected tests
4. Repeat until passing

**8. Iterate until green:**

Loop through fix-and-test cycle:
```
Attempt [N]: Fixing [issue description]
  - Modified: [file]
  - Change: [brief description]
  - Re-running tests...
```

Maximum 5 fix attempts. If still failing, report the remaining failures and stop:
```
Unable to auto-fix after 5 attempts.

Remaining failures:
- [test]: [issue]

Manual intervention required. Review the failing tests and source code.
```

**9. Report success:**
```
════════════════════════════════════════
TESTS PASSING: US-XXX
════════════════════════════════════════

Story: [title]

Test Results:
- Unit tests: [N] passing
- Integration tests: [N] passing
- Total: [N] tests

Coverage:
- AC1: Covered by [test names]
- AC2: Covered by [test names]
- AC3: Covered by [test names]

Test files created:
- [path/to/test1.test.ts]
- [path/to/test2.test.ts]

[If fixes were made:]
Code fixes applied:
- [file]: [brief fix description]

Next step: commit story with tests using /commit-us
════════════════════════════════════════
```

</process>

<auto_fix_rules>
When fixing code to make tests pass:

**Rule 1: Prefer implementation fixes**
- If test expectation matches AC, fix implementation
- If test expectation is wrong, fix test

**Rule 2: Minimal changes**
- Make smallest change that fixes the issue
- Don't refactor unrelated code
- Don't add features

**Rule 3: Preserve behavior**
- Fixes shouldn't break existing functionality
- Run full test suite after each fix

**Rule 4: Document fixes**
- Track all changes made
- Include in summary output
</auto_fix_rules>

<success_criteria>
- Test files created following project conventions
- All acceptance criteria have test coverage
- All tests passing (or user opted to skip)
- Code fixes documented if applied
- User informed of results
</success_criteria>
