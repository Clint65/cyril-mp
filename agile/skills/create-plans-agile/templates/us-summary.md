# User Story Summary Template

Copy for `.planning/EXX-name/FXX-name/US-XXX-name/US-XXX-SUMMARY.md`:

```markdown
# US-XXX: [Story Title] Summary

**[Substantive one-liner - what shipped, not "story complete"]**

## Story Completed

**As a** [from story]
**I want** [from story]
**So that** [from story]

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | [brief] | Done | [verification method] |
| AC2 | [brief] | Done | [verification method] |
| AC3 | [brief] | Done | [verification method] |

## Accomplishments

- [Key outcome 1 - what was built]
- [Key outcome 2 - what value delivered]
- [Key outcome 3 - notable implementation detail]

## Files Created/Modified

- `path/to/file.ts` - [What this file does/what changed]
- `path/to/another.ts` - [Description]
- `path/to/test.ts` - [Test coverage added]

## Decisions Made

[Key implementation decisions and rationale]

- **[Decision]:** [What was decided and why]
- **[Approach]:** [Which approach taken and why]

## Deviations from Plan

[None - story executed as planned]

OR

### Auto-fixed Issues
- [Issue found]: [How it was fixed]
- Logged per deviation rule #[N]

### Deferred Enhancements
- [Enhancement]: Logged to ISSUES.md for future consideration
- Per deviation rule #5

## Definition of Done Checklist

- [x] All acceptance criteria met
- [x] Code self-reviewed
- [x] Tests written and passing
- [ ] Documentation updated (N/A for this story)

## Story Metrics

**Estimated:** X points
**Actual Effort:** [X points / As estimated / More than estimated - reason]

## Next Story

Ready for: US-XXX+1 "[next story title]"
OR: "Feature F0X complete - ready for next feature"
OR: "Epic E0X complete - ready for milestone"

---
*Story: US-XXX*
*Feature: F0X - [Name]*
*Epic: E0X - [Name]*
*Completed: YYYY-MM-DD*
```

<summary_guidelines>
**One-liner:**
- Substantive: "Added email/password login with validation"
- Not generic: "Story complete" or "Implemented user story"

**AC Status:**
- Every AC must be listed
- Status: Done, Partial, Skipped (with reason)
- Verification: How it was verified (test, manual, build)

**Accomplishments:**
- What was built (concrete)
- What value delivered (user-facing)
- Notable implementation details
</summary_guidelines>

<deviation_tracking>
Track all deviations from the original US-PLAN:

**Auto-fixed (Rules 1-3):**
- Bug found and fixed
- Missing critical functionality added
- Blocker resolved

**Deferred (Rule 5):**
- Enhancement ideas logged to ISSUES.md
- Scope creep prevented
- Future improvements captured

**Architectural (Rule 4):**
- Major change requested
- User decision documented
- Rationale recorded
</deviation_tracking>

<metrics>
Compare estimated vs actual:
- **As estimated:** Story took expected effort
- **Under:** Story was simpler than expected (good)
- **Over:** Story was more complex - capture why for future estimation

This data improves future story point estimation.
</metrics>
