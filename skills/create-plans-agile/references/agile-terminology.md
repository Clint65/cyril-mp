# Agile Terminology Reference

This reference defines the key agile/scrum terms as used in the create-plans-agile skill.

## Core Artifacts

### Epic
A large body of work that delivers distinct business value. Contains multiple features.

**Characteristics:**
- 1-4 weeks of work (solo + Claude)
- Can be released independently
- Has clear value proposition
- Contains 2-4 features

**Example:** "E01: Authentication" - enables users to securely access the application.

### Feature
A coherent unit of functionality within an epic. Contains multiple user stories.

**Characteristics:**
- Can be demonstrated independently
- Contains 2-5 user stories
- 8-20 story points total
- Delivers specific capability

**Example:** "F01: Login" - allows users to authenticate with email and password.

### User Story
A description of a feature from an end-user perspective. The atomic unit of work.

**Format:** As a [user type], I want [capability], so that [benefit]

**Characteristics:**
- Independent, valuable, testable
- 1-5 story points
- Has acceptance criteria
- Completes in one session

**Example:** "US-001: As a returning user, I want to stay logged in, so that I don't have to re-enter credentials."

### Acceptance Criteria (AC)
Conditions that must be satisfied for a user story to be considered complete.

**Format:** Given [context], When [action], Then [outcome]

**Characteristics:**
- Testable (pass/fail)
- Independent of each other
- 2-4 per story
- Maps to tasks

**Example:**
```
Given I check "Remember me" during login
When I close and reopen the browser
Then I am still logged in
```

### Story Points
Relative effort estimate for user stories.

**Scale:**
- 1 point: Trivial (<1 hour)
- 2 points: Small (1-2 hours)
- 3 points: Medium (half-day)
- 5 points: Larger (full day)
- 8 points: Large - should split

**Key principle:** Relative comparison, not absolute time.

## Planning Artifacts

### BRIEF.md
Human-focused vision document. Captures what we're building and why.

### ROADMAP.md
Epic-level overview with progress tracking. Shows all epics and their status.

### EPIC.md
Epic definition with features list. Lives in `epics/EXX-name/` directory.

### FEATURE.md
Feature definition with stories list. Lives in `features/FXX-name/` directory.

### US-XXX.md
User story definition with acceptance criteria. The "what" to build.

### US-XXX-PLAN.md
Executable plan with tasks. The "how" to build. Claude executes this directly.

### US-XXX-SUMMARY.md
Completion documentation. Records what was built, deviations, outcomes.

## Status Values

### Story Status
- `not_started` - Story defined, not yet planned
- `planned` - US-PLAN.md created
- `in_progress` - Currently being executed
- `done` - US-SUMMARY.md exists

### Feature Status
- `Not started` - No stories complete
- `In progress` - Some stories complete
- `Complete` - All stories complete

### Epic Status
- `Not Started` - No features complete
- `In Progress` - Some features complete
- `Complete` - All features complete

## Priority Values (MoSCoW)

- `must` - Critical for current release
- `should` - Important but not critical
- `could` - Desirable if time permits
- `wont` - Not in current scope

## Key Principles

### INVEST Criteria
Good user stories are:
- **I**ndependent - Can be developed alone
- **N**egotiable - Details can be refined
- **V**aluable - Delivers user value
- **E**stimable - Can be sized
- **S**mall - Fits in one session
- **T**estable - Clear pass/fail criteria

### Definition of Done
Standard completion checklist:
- All acceptance criteria met
- Code reviewed (self-review for solo)
- Tests written and passing
- No new bugs introduced
- Documentation updated (if applicable)

### Vertical Slicing
Stories should deliver end-to-end functionality, not horizontal layers.

**Good:** "User can log in with email" (touches UI, API, database)
**Bad:** "Create user database schema" (only database layer)

## What This Skill Does NOT Include

This is agile structure without agile ceremony:

- No sprint planning meetings
- No daily standups
- No sprint retrospectives
- No velocity tracking
- No burndown charts
- No team ceremonies

Ship story by story. Track progress in artifacts. No overhead.
