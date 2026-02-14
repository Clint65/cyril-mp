# Agile Planning - Claude Code Plugin Marketplace

Agile project planning for Claude Code with Epic/Feature/User Story structure. Scrum-like methodology without the ceremony, optimized for solo agentic development.

## Installation

### Add the marketplace

```bash
/plugin marketplace add Clint65/agile-skill
```

### Install the plugin

```bash
/plugin install agile-planning@agile-skill
```

## Skills included

| Skill | Command | Description |
|-------|---------|-------------|
| create-plan-agile | `/create-plan-agile [project]` | Create agile plans with epics, features, and user stories |
| plan-us | `/plan-us <story-path>` | Create an execution plan (US-PLAN.md) for a user story |
| run-us | `/run-us <plan-path>` | Execute a US-PLAN.md implementation plan |
| test-us | `/test-us <story-path>` | Create and run tests for a completed user story |
| commit-us | `/commit-us <story-path>` | Commit a completed user story with proper git format |
| qa-report | `/qa-report [scope]` | Generate a QA report for user stories coverage |

## Philosophy

**Agile structure without agile ceremony.**

You get:
- Epics for grouping related value
- Features for coherent functionality
- User stories with As a / I want / So that format
- Acceptance criteria with Given / When / Then
- Story point estimation
- Progress tracking

You don't get:
- Sprint planning meetings
- Daily standups
- Sprint retrospectives
- Velocity tracking ceremonies

Ship story by story. Track progress in artifacts. No overhead.

## Workflow

```
/create-plan-agile my project    # 1. Plan: brief -> roadmap -> epics -> features -> stories
/plan-us <story-path>            # 2. Create execution plan for a story
/run-us <plan-path>              # 3. Execute the plan
/test-us <story-path>            # 4. Test the implementation
/commit-us <story-path>          # 5. Commit with proper format
/qa-report all                   # 6. Generate QA report
```

## Planning hierarchy

```
BRIEF.md          - What we're building (human vision)
ROADMAP.md        - Epic overview (2-4 epics)
EPIC.md           - Epic details + features (2-4 features)
FEATURE.md        - Feature details + stories (2-5 stories)
US-XXX.md         - User story (As a/I want/So that + AC)
US-XXX-PLAN.md    - Executable plan (tasks Claude runs)
US-XXX-SUMMARY.md - What shipped (outcome documentation)
```

## License

MIT
