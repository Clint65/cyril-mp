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

<story_format_guidelines>
**As a / I want / So that:**
- Be specific about the user type (not just "user")
- Action should be concrete and achievable
- Benefit should explain WHY this matters

**Good:**
```
As a returning user
I want to stay logged in across browser sessions
So that I don't have to re-enter my credentials every visit
```

**Bad:**
```
As a user
I want better login
So that it's easier
```
</story_format_guidelines>

<acceptance_criteria_guidelines>
**Given/When/Then:**
- Given: The precondition (state before action)
- When: The action/trigger
- Then: The expected outcome (verifiable)

**Good AC:**
```
### AC1: Valid login
- **Given** I'm on the login page with valid credentials
- **When** I click the login button
- **Then** I'm redirected to the dashboard
```

**Bad AC:**
```
- Login should work
- User should see dashboard
```

**Keep ACs:**
- Testable (can verify pass/fail)
- Independent (don't depend on other ACs)
- Focused (one behavior each)
</acceptance_criteria_guidelines>

<story_points_guide>
**1 point:** Trivial change, <1 hour, well-understood
**2 points:** Small change, 1-2 hours, straightforward
**3 points:** Medium change, half-day, some complexity
**5 points:** Larger change, full day, multiple files/components
**8 points:** Large change, 1-2 days - **consider splitting**

If estimated >5 points, strongly consider splitting the story.
If estimated >8 points, always split into multiple stories.
</story_points_guide>

<invest_criteria>
Good user stories are **INVEST**:
- **I**ndependent: Can be developed without other stories
- **N**egotiable: Details can be refined
- **V**aluable: Delivers value to user
- **E**stimable: Can be sized
- **S**mall: Fits in one session
- **T**estable: Clear pass/fail criteria
</invest_criteria>
