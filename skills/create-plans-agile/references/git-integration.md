# Git Integration Reference

## Core Principle

**Commit outcomes, not process.**

The git log should read like a changelog of what shipped, not a diary of planning activity.

## Commit Points

| Event | Commit? | Why |
|-------|---------|-----|
| BRIEF + ROADMAP created | YES | Project initialization |
| EPIC.md created | NO | Intermediate - commit with first story completion |
| FEATURE.md created | NO | Intermediate |
| US-XXX.md (user story) created | NO | Intermediate |
| US-XXX-PLAN.md created | NO | Intermediate - commit with completion |
| **User story completed** | YES | Actual code shipped |
| Feature completed | YES | Milestone within epic |
| Epic completed | YES | Major milestone |
| Handoff created | YES | WIP state preserved |

## Git Check on Invocation

```bash
git rev-parse --git-dir 2>/dev/null || echo "NO_GIT_REPO"
```

If NO_GIT_REPO:
- Inline: "No git repo found. Initialize one? (Recommended for version control)"
- If yes: `git init`

## Commit Message Formats

### 1. Project Initialization (brief + roadmap together)

```
docs: initialize [project-name] with agile structure ([N] epics)

[One-liner from BRIEF.md]

Epics:
- E01: [epic-name]: [value proposition]
- E02: [epic-name]: [value proposition]
```

What to commit:
```bash
git add .planning/
git commit
```

### 2. User Story Completion

```
feat(US-XXX): [story title]

As a [user], I want [capability] so that [benefit]

Acceptance Criteria:
- [x] AC1: [criterion]
- [x] AC2: [criterion]

Files: [key files modified]
```

What to commit:
```bash
git add .planning/EXX-name/FXX-name/US-XXX-name/  # Story folder with PLAN + SUMMARY
git add src/                                       # Actual code created
git commit
```

### 3. Feature Completion

```
feat(F0X): [feature name] complete

Stories completed:
- US-001: [title]
- US-002: [title]
- US-003: [title]

Total: [X] points delivered
```

### 4. Epic Completion

```
feat(E0X): [epic name] complete

[Value proposition achieved]

Features:
- F01: [name] (X stories)
- F02: [name] (X stories)

Total: [X] stories, [XX] points
```

### 5. Handoff (WIP)

```
wip: US-XXX paused at task [X]/[Y]

Current: [task name]
[If blocked:] Blocked: [reason]
```

What to commit:
```bash
git add .planning/
git commit
```

## Example Clean Git Log

```
a]7f2d1 feat(US-005): password reset flow
b]3e9c4 feat(US-004): remember me checkbox
c]8a1b2 feat(US-003): logout functionality
d]5c3d7 feat(F01): login feature complete
e]2f4a8 feat(US-002): login form validation
f]1b3c5 feat(US-001): basic login with email/password
g]9d2e6 docs: initialize auth-app with agile structure (3 epics)
```

## What NOT To Commit Separately

- EPIC.md creation (wait for first story completion)
- FEATURE.md creation (intermediate)
- US-XXX.md creation (intermediate)
- US-XXX-PLAN.md creation (wait for story completion)
- Minor planning tweaks
- "Fixed typo in user story"

These create noise. Commit outcomes, not process.
