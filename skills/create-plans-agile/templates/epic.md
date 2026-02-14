# Epic Template

Copy for `.planning/EXX-name/EPIC.md`:

```markdown
# Epic: [Epic Name]

**Epic ID:** E0X
**Status:** Not Started | In Progress | Complete
**Created:** YYYY-MM-DD

## Value Proposition

**As a** [user type]
**I want** [capability at epic level]
**So that** [business outcome]

## Success Metrics

How we know this epic succeeded:
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]

## Features

| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F01 | [Name] | X | XX | Not started |
| F02 | [Name] | X | XX | Not started |

**Total:** X features, X stories, XX points

## Dependencies

- **Requires:** [Other epics/external dependencies]
- **Enables:** [What this epic enables]

## Out of Scope

- [What this epic does NOT include]
- [Deferred to future epic]

## Technical Notes

[Any architectural considerations or constraints]
```

<guidelines>
**Good epics:**
- Deliver distinct business value
- Can be released independently
- Take 1-4 weeks to complete (solo + Claude)
- Have clear success metrics

**Epic naming:**
- `E01-authentication` - clear, lowercase, hyphenated
- `E02-dashboard` - sequential numbering
- `E03-notifications` - descriptive name

**Feature count:**
- 2-4 features per epic
- Each feature is coherent unit of functionality
</guidelines>

<status_updates>
Update EPIC.md when:
- Feature started: Update feature status
- Feature completed: Check off feature, update counts
- Epic completed: Update status, add completion date

```markdown
## Features

| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F01 | Login | 3/3 | 8/8 | Complete |
| F02 | Registration | 2/2 | 5/5 | Complete |

**Total:** 2 features, 5 stories, 13 points - COMPLETE
```
</status_updates>
