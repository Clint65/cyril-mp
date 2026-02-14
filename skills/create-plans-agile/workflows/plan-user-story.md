# Workflow: Plan User Story

<required_reading>
**Read these files NOW:**
1. templates/us-plan.md
2. references/scope-estimation.md
3. references/checkpoints.md
4. The user story file to plan
</required_reading>

<purpose>
Create an executable US-PLAN.md for a user story. This transforms the story's
acceptance criteria into specific tasks Claude can execute.
</purpose>

<process>

<step name="read_story">
```bash
cat .planning/EXX-name/FXX-name/US-XXX-name/US-XXX.md
```

Parse:
- Story (As a/I want/So that)
- Acceptance criteria (Given/When/Then)
- Suggested tasks
- Story points
- Technical notes
</step>

<step name="check_dependencies">
Are there other stories this depends on?
```bash
ls .planning/EXX-name/FXX-name/US-*/US-*-SUMMARY.md 2>/dev/null
```

If dependencies not complete, warn user.
</step>

<step name="map_ac_to_tasks">
For each acceptance criterion, create 1+ tasks.

**Task mapping rules:**
- Each AC should have at least one task that satisfies it
- Task `<done>` criteria should match AC's Then clause
- 2-3 tasks total per plan (split story if more needed)

Present:
```
Mapping acceptance criteria to tasks:

AC1: [criterion]
  -> Task 1: [implementation task]

AC2: [criterion]
  -> Task 2: [implementation task]

AC3: [criterion]
  -> Task 3: [implementation task] + checkpoint:human-verify

This plan has 3 tasks targeting ~50% context.
Does this mapping look right? (yes / adjust)
```
</step>

<step name="scope_check">
If >3 tasks needed:
```
This story requires >3 tasks. Recommend splitting into:

US-XXX-a: [first part] (AC1, AC2)
US-XXX-b: [second part] (AC3, AC4)

Split the story, or proceed with larger plan?
```

Strongly recommend splitting.
</step>

<step name="determine_checkpoints">
Decide checkpoint placement:

**Add checkpoint:human-verify when:**
- Story involves UI (visual verification needed)
- Story involves user flows (functional verification)
- End of story (final acceptance)

**No checkpoints when:**
- Pure backend/API work
- All verification is programmatic
- Story is fully autonomous

```
Checkpoint plan:
- Tasks 1-2: Autonomous (auto)
- Task 3: checkpoint:human-verify (verify complete story)

Or: No checkpoints (fully autonomous, can run in subagent)

Does this checkpoint plan work? (yes / adjust)
```
</step>

<step name="write_plan">
Create `US-XXX-PLAN.md` using templates/us-plan.md.

Include:
- YAML frontmatter (story, feature, epic, type: execute)
- objective (story with As a/I want/So that)
- execution_context (workflow + template references)
- context (@file references)
- acceptance_criteria (copy from story)
- tasks (with maps-to attribute)
- verification (checklist)
- success_criteria (AC mapping)
- output (US-SUMMARY.md)
</step>

<step name="offer_next">
```
Plan created: US-XXX-PLAN.md
Tasks: X (maps to X acceptance criteria)
Checkpoints: [None - autonomous | X checkpoints]

What's next?
1. Execute this plan (/run-us path)
2. Review/adjust tasks
3. Done for now
```
</step>

</process>

<task_structure>
Each task should have:

```xml
<task type="auto" maps-to="AC1">
  <name>Task 1: [Action-oriented name]</name>
  <files>path/to/file.ext</files>
  <action>
    [Specific implementation instructions]
    - What to create/modify
    - Which patterns to follow
    - What to avoid
  </action>
  <verify>[Command or check to verify]</verify>
  <done>[Completion criteria - maps to AC1]</done>
</task>
```

**The `maps-to` attribute** links task to acceptance criteria.
This ensures every AC is covered by at least one task.
</task_structure>

<autonomous_vs_interactive>
**Autonomous plans (no checkpoints):**
- Execute via subagent with fresh context
- Best for backend, API, pure code work
- Impossible to degrade (fresh context)

**Interactive plans (has checkpoints):**
- Execute in main context
- Required for UI, visual, user flow verification
- Still target 2-3 tasks
</autonomous_vs_interactive>

<success_criteria>
Plan is complete when:
- [ ] `US-XXX-PLAN.md` exists
- [ ] 2-3 tasks defined (or story split)
- [ ] Every AC maps to at least one task
- [ ] Checkpoint placement decided
- [ ] Context references included
- [ ] User knows how to execute
</success_criteria>
