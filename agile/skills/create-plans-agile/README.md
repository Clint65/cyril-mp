# create-plans-agile

Agile project planning skill with Epic/Feature/User Story structure. Like `create-plans` but with Scrum-like methodology.

## Philosophy

**Agile structure without agile ceremony.**

You get:
- Epics for grouping related value
- Features for coherent functionality
- User stories with As a/I want/So that format
- Acceptance criteria with Given/When/Then
- Story point estimation
- Progress tracking

You don't get:
- Sprint planning meetings
- Daily standups
- Sprint retrospectives
- Velocity tracking ceremonies
- Burndown charts

Ship story by story. Track progress in artifacts. No overhead.

## Hierarchy

```
BRIEF.md          - What we're building (human vision)
    |
ROADMAP.md        - Epic overview (2-4 epics)
    |
EPIC.md           - Epic details + features (2-4 features)
    |
FEATURE.md        - Feature details + stories (2-5 stories)
    |
US-XXX.md         - User story (As a/I want/So that + AC)
    |
US-XXX-PLAN.md    - Executable plan (tasks Claude runs)
    |
US-XXX-SUMMARY.md - What shipped (outcome documentation)
```

## Directory Structure

```
.planning/
├── BRIEF.md
├── ROADMAP.md
├── ISSUES.md
├── E01-authentication/
│   ├── EPIC.md
│   └── F01-login/
│       ├── FEATURE.md
│       ├── US-001-basic-login/
│       │   ├── US-001.md
│       │   ├── US-001-PLAN.md
│       │   └── US-001-SUMMARY.md
│       └── US-002-remember-me/
│           └── ...
└── E02-dashboard/
    └── ...
```

## Commands

| Command | Purpose |
|---------|---------|
| `/create-plan-agile [project]` | Invoke skill for agile planning |
| `/plan-us <story-path>` | Create US-PLAN.md for a user story |
| `/run-us <plan-path>` | Execute a US-PLAN.md |

## User Story Format

```markdown
---
id: US-001
feature: F01
epic: E01
points: 3
status: not_started
---

# User Story: Basic Login

## Story

**As a** returning user
**I want** to log in with email and password
**So that** I can access my account

## Acceptance Criteria

### AC1: Valid login
- **Given** I'm on the login page with valid credentials
- **When** I click the login button
- **Then** I'm redirected to the dashboard

### AC2: Invalid password
- **Given** I enter an incorrect password
- **When** I click login
- **Then** I see an error message
```

## Story Points

| Points | Complexity | Time | Action |
|--------|------------|------|--------|
| 1 | Trivial | <1hr | Just do it |
| 2 | Small | 1-2hr | Standard |
| 3 | Medium | half-day | Standard |
| 5 | Larger | full day | Consider splitting |
| 8 | Large | 1-2 days | **Must split** |

## Quick Start

```bash
# 1. Start agile planning
/create-plan-agile my new project

# 2. Follow prompts to create:
#    - Brief
#    - Roadmap with epics
#    - Epic with features
#    - Feature with user stories

# 3. Plan a user story
/plan-us .planning/E01-auth/F01-login/US-001-basic-login/US-001.md

# 4. Execute the plan
/run-us .planning/E01-auth/F01-login/US-001-basic-login/US-001-PLAN.md
```

## vs create-plans

| Aspect | create-plans | create-plans-agile |
|--------|--------------|-------------------|
| Structure | Phases | Epics > Features > Stories |
| Story format | N/A | As a/I want/So that |
| Acceptance criteria | Verification | Given/When/Then |
| Estimation | Task count | Story points |
| Output | PLAN.md | US-PLAN.md |

Use **create-plans-agile** when:
- You prefer Scrum-like user stories
- You want story point tracking
- Project has clear user-facing features

Use **create-plans** when:
- You prefer phase-based planning
- Project is more technical/infrastructure focused
- Simpler structure is preferred

## File Reference

### References
- `agile-terminology.md` - Key terms and definitions
- `user-story-format.md` - How to write good stories
- `story-points.md` - Estimation guidelines
- `backlog-management.md` - Prioritization and ordering
- `checkpoints.md` - Human verification points
- `scope-estimation.md` - Quality-driven splitting

### Templates
- `brief.md` - Project vision
- `roadmap-agile.md` - Epic-based roadmap
- `epic.md` - Epic definition
- `feature.md` - Feature with story list
- `user-story.md` - Story with acceptance criteria
- `us-plan.md` - Executable story plan
- `us-summary.md` - Completion documentation

### Workflows
- `create-brief.md` - Project vision
- `create-roadmap-agile.md` - Epic structure
- `create-epic.md` - Epic with features
- `create-feature.md` - Feature with stories
- `create-user-story.md` - Story with AC
- `plan-user-story.md` - Create US-PLAN
- `execute-user-story.md` - Run US-PLAN
- `complete-user-story.md` - Mark story done
- `handoff.md` - Pause and preserve context
- `resume.md` - Continue from handoff
