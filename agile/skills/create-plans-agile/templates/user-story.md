# User Story Template

Copy for `.planning/EXX-name/FXX-name/US-XXX-name/US-XXX.md`:

```yaml
---
id: US-XXX
feature: F0X
epic: E0X
points: X
status: not_started | planned | in_progress | done
priority: must | should | could | wont
---
```

```markdown
# User Story: [Brief Title]

## Story

**As a** [specific user type/persona]
**I want** [specific capability/action]
**So that** [specific benefit/value]

## Acceptance Criteria

Given/When/Then format:

### AC1: [Criterion name]
- **Given** [initial context]
- **When** [action taken]
- **Then** [expected outcome]

### AC2: [Criterion name]
- **Given** [initial context]
- **When** [action taken]
- **Then** [expected outcome]

### AC3: [Criterion name]
- **Given** [initial context]
- **When** [action taken]
- **Then** [expected outcome]

## Tasks

Suggested implementation tasks (refined during planning):

1. [ ] [Task 1 - what needs to be built]
2. [ ] [Task 2]
3. [ ] [Task 3]

## Technical Notes

[Implementation considerations, existing code to reference, patterns to follow]

## Definition of Done

- [ ] All acceptance criteria met
- [ ] Code reviewed (self-review for solo)
- [ ] Tests written and passing
- [ ] No new bugs introduced
- [ ] Documentation updated (if applicable)
```

<guidelines_reference>
For story format examples, AC quality guidelines, story point scale, and INVEST criteria,
see `references/user-story-format.md`.
</guidelines_reference>
