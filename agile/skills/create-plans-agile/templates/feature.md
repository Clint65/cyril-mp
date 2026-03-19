# Feature Template

Copy for `.planning/EXX-name/FXX-name/FEATURE.md`:

```markdown
# Feature: [Feature Name]

**Feature ID:** F0X (within E0X)
**Epic:** E0X - [Epic Name]
**Status:** Not Started | In Progress | Complete

## Description

[2-3 sentences describing what this feature does and why it matters]

## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-001 | [Brief story title] | 3 | Not started | - |
| US-002 | [Brief story title] | 2 | Not started | - |
| US-003 | [Brief story title] | 5 | Not started | - |

**Total:** X stories, XX points

## Acceptance Criteria (Feature Level)

Feature is complete when:
- [ ] [High-level criterion 1]
- [ ] [High-level criterion 2]
- [ ] All user stories pass their individual acceptance criteria

## Technical Notes

[Any architectural considerations for this feature]

## Dependencies

- **Requires:** [Other features/stories that must be complete first]
- **Enables:** [What completing this feature enables]
```

<guidelines>
**Good features:**
- Coherent unit of functionality
- Can be demonstrated independently
- Contains 2-5 user stories
- 8-20 story points total

**Feature naming:**
- `F01-login` - sequential within epic
- `F02-registration` - clear, descriptive
- `F03-password-reset` - lowercase, hyphenated

**Story ordering:**
- Dependencies first
- Foundation -> Enhancement pattern
- Test/polish stories last
</guidelines>

<status_updates>
Update FEATURE.md when:
- Story started: Update story status to "In progress"
- Story completed: Update status to "Complete", add date
- Feature completed: Update feature status, check off feature-level ACs

```markdown
## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-001 | Basic login | 3 | Complete | 2025-01-15 |
| US-002 | Remember me | 2 | Complete | 2025-01-16 |
| US-003 | Error handling | 2 | In progress | - |

**Total:** 3 stories, 7 points (5/7 complete)
```
</status_updates>

<story_ordering_tips>
**Order stories for:**
1. Dependencies - foundational stories first
2. Risk reduction - risky/uncertain stories earlier
3. Value delivery - high-value stories first
4. Testability - stories that enable testing earlier

**Example ordering:**
1. US-001: Basic login form (foundation)
2. US-002: Form validation (builds on #1)
3. US-003: Error messages (builds on #2)
4. US-004: Remember me checkbox (enhancement)
5. US-005: Loading states (polish)
</story_ordering_tips>
