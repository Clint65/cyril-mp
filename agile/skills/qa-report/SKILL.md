---
name: qa-report
description: Execute all tests and generate a QA report for user stories coverage and results. Use when the user wants a quality assessment, test coverage overview, story completion status, or to identify testing gaps across stories. Triggers on "QA report", "test coverage", "quality check", "story status overview", or when auditing project health.
argument-hint: [scope: epic (E01), feature (F01), or "all"]
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

Generate a comprehensive QA report for $ARGUMENTS user stories. If no argument provided, default scope is "all".

<objective>
Act as a QA Engineer to:
1. Inventory all user stories and their test coverage
2. Execute all existing tests
3. Generate a detailed QA report identifying gaps and results
</objective>

<process>

**1. Scan planning structure:**
```bash
# Find all user stories
find .planning -name "US-*.md" -not -name "*-PLAN.md" -not -name "*-SUMMARY.md" 2>/dev/null | sort

# Find all summaries (completed stories)
find .planning -name "US-*-SUMMARY.md" 2>/dev/null | sort

# Find all plans
find .planning -name "US-*-PLAN.md" 2>/dev/null | sort
```

Apply scope filter if provided:
- `E01` → filter to `.planning/E01-*/`
- `F01` → filter to `.planning/E*-*/F01-*/`
- `all` → no filter

**2. Build story inventory:**

For each user story found:
```bash
# Read story to extract:
# - Story ID (US-XXX)
# - Story title
# - Acceptance criteria count
cat [story-path]
```

Create inventory:
| Story | Title | Has Plan | Has Summary | Status |
|-------|-------|----------|-------------|--------|
| US-001 | ... | Yes/No | Yes/No | Complete/In Progress/Not Started |

**3. Detect test framework:**
```bash
# Check for test setup
ls package.json pyproject.toml setup.py Cargo.toml go.mod 2>/dev/null

# Find test directories
ls -d test/ tests/ __tests__ spec/ src/**/__tests__ 2>/dev/null

# Count existing test files
find . -name "*.test.*" -o -name "*.spec.*" -o -name "test_*.py" -o -name "*_test.go" 2>/dev/null | wc -l
```

**4. Map tests to user stories:**

Search for test files that reference story IDs:
```bash
# Find tests mentioning story IDs
grep -r "US-[0-9]" tests/ __tests__ spec/ 2>/dev/null || true

# Find test files in same directories as source files from summaries
# Parse SUMMARY files for "Files Created/Modified" section
```

Build test mapping:
| Story | Test Files | Test Count | Coverage |
|-------|------------|------------|----------|
| US-001 | auth.test.ts | 5 | Partial |
| US-002 | - | 0 | None |

**5. Execute all tests:**
```bash
# Run test suite with coverage if available
npm test -- --coverage 2>&1 || \
pytest -v --cov 2>&1 || \
go test -v -cover ./... 2>&1 || \
cargo test 2>&1

# Capture exit code
echo "Exit code: $?"
```

Parse results:
- Total tests run
- Passed / Failed / Skipped
- Coverage percentage (if available)
- Individual test results

**6. Analyze acceptance criteria coverage:**

For each completed story (has SUMMARY):
1. Read the story's acceptance criteria
2. Read the SUMMARY's AC status
3. Check if tests exist for each AC
4. Determine coverage level:
   - **Full**: All AC have tests
   - **Partial**: Some AC have tests
   - **None**: No tests for any AC

**7. Identify gaps:**

**Missing tests:**
- Stories with no test files
- AC without corresponding tests
- Edge cases not covered

**Incomplete workflow:**
- Stories without plans
- Stories without summaries
- Stories marked complete but no tests

**Test quality issues:**
- Failing tests
- Skipped tests
- Low coverage areas

**8. Generate QA report:**

Write to `.planning/QA-REPORT.md`:

```markdown
# QA Report

**Generated:** [date]
**Scope:** [scope or "All Stories"]
**QA Engineer:** Claude

## Executive Summary

| Metric | Value |
|--------|-------|
| Total Stories | X |
| Stories Completed | X |
| Stories with Tests | X |
| Test Coverage | X% |
| Tests Passing | X/Y |

**Overall Status:** [PASS / NEEDS ATTENTION / CRITICAL]

## Test Execution Results

**Test Suite:** [framework]
**Run Date:** [timestamp]

| Status | Count |
|--------|-------|
| Passed | X |
| Failed | X |
| Skipped | X |
| Total | X |

[If failures:]
### Failed Tests

| Test | File | Error |
|------|------|-------|
| [name] | [file:line] | [brief error] |

## Story Coverage Detail

### Epic: E01 - [Name]

#### Feature: F01 - [Name]

| Story | Title | Status | Tests | AC Coverage | Result |
|-------|-------|--------|-------|-------------|--------|
| US-001 | [title] | Complete | 5 | 3/3 (100%) | PASS |
| US-002 | [title] | Complete | 2 | 1/3 (33%) | PARTIAL |
| US-003 | [title] | Complete | 0 | 0/2 (0%) | NO TESTS |
| US-004 | [title] | In Progress | - | - | N/A |

[Repeat for each feature/epic]

## Coverage Gaps

### Stories Missing Tests

| Story | Title | Acceptance Criteria | Priority |
|-------|-------|---------------------|----------|
| US-002 | [title] | AC2, AC3 missing | High |
| US-003 | [title] | All AC missing | Critical |

### Recommended Test Actions

1. **Critical:** [Story] - [What tests to add]
2. **High:** [Story] - [What tests to add]
3. **Medium:** [Story] - [What tests to add]

## Workflow Compliance

| Check | Status |
|-------|--------|
| All stories have plans | [X/Y] |
| All stories have summaries | [X/Y] |
| Mandatory workflow followed | [Yes/No] |

[If issues:]
### Workflow Issues

| Story | Issue | Action Required |
|-------|-------|-----------------|
| US-XXX | No plan | Run /plan-us |
| US-YYY | No summary | Run /run-us |

## Recommendations

1. [Priority recommendation]
2. [Next recommendation]
3. [...]

---
*Report generated by /qa-report command*
*Next QA review recommended: [date + 1 week]*
```

**9. Report completion:**
```
════════════════════════════════════════
QA REPORT GENERATED
════════════════════════════════════════

Report: .planning/QA-REPORT.md

Summary:
- Stories: [X] total, [Y] complete
- Tests: [N] passing, [M] failing
- Coverage: [X]%

[If issues found:]
Action Required:
- [N] stories need tests
- [M] tests failing
- [P] workflow issues

Review: .planning/QA-REPORT.md
Fix gaps with /test-us for stories needing coverage.
════════════════════════════════════════
```

</process>

<coverage_levels>
Define coverage based on AC verification:

- **100%**: All AC have at least one test
- **75-99%**: Most AC covered, minor gaps
- **50-74%**: Partial coverage, significant gaps
- **25-49%**: Low coverage, needs attention
- **0-24%**: Critical - minimal or no tests
</coverage_levels>

<priority_rules>
Assign test priority:

- **Critical**: Completed story with 0 tests
- **High**: Completed story with <50% AC coverage
- **Medium**: Completed story with 50-99% AC coverage
- **Low**: In-progress story (tests can wait)
</priority_rules>

<success_criteria>
- All stories inventoried with status
- All existing tests executed
- Test results captured and parsed
- Coverage gaps identified with priorities
- Workflow compliance checked
- QA-REPORT.md generated
- Actionable recommendations provided
</success_criteria>
