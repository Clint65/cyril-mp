# Continue-Here Template

Copy and fill this structure for `.planning/EXX-name/.continue-here.md`:

```yaml
---
epic: EXX-name
feature: FXX-name
story: US-XXX
task: 2
total_tasks: 3
status: in_progress
last_updated: 2025-01-15T14:30:00Z
---
```

```markdown
<current_state>
[Where exactly are we? What's the immediate context?]
</current_state>

<completed_work>
[What got done this session - be specific]

- Task 1: [name] - Done
- Task 2: [name] - In progress, [what's done on it]
</completed_work>

<remaining_work>
[What's left in this user story]

- Task 2: [name] - [what's left to do]
- Task 3: [name] - Not started
</remaining_work>

<acceptance_criteria_status>
[Which ACs are met, which remain]

- AC1: [criterion] - DONE
- AC2: [criterion] - IN PROGRESS
- AC3: [criterion] - NOT STARTED
</acceptance_criteria_status>

<decisions_made>
[Key decisions and why - so next session doesn't re-debate]

- Decided to use [X] because [reason]
- Chose [approach] over [alternative] because [reason]
</decisions_made>

<blockers>
[Anything stuck or waiting on external factors]

- [Blocker 1]: [status/workaround]
</blockers>

<context>
[Mental state, "vibe", anything that helps resume smoothly]

[What were you thinking about? What was the plan?
This is the "pick up exactly where you left off" context.]
</context>

<next_action>
[The very first thing to do when resuming]

Start with: [specific action]
</next_action>
```

<yaml_fields>
Required YAML frontmatter:

- `epic`: Epic directory name (e.g., `E01-authentication`)
- `feature`: Feature directory name (e.g., `F01-login`)
- `story`: User story ID (e.g., `US-001`)
- `task`: Current task number
- `total_tasks`: How many tasks in story plan
- `status`: `in_progress`, `blocked`, `almost_done`
- `last_updated`: ISO timestamp
</yaml_fields>

<guidelines>
- Be specific enough that a fresh Claude instance understands immediately
- Include WHY decisions were made, not just what
- Track acceptance criteria status explicitly
- The `<next_action>` should be actionable without reading anything else
- This file gets DELETED after resume - it's not permanent storage
</guidelines>
