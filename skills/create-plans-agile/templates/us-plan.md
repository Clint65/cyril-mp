# User Story Plan Template

Copy for `.planning/EXX-name/FXX-name/US-XXX-name/US-XXX-PLAN.md`:

```yaml
---
story: US-XXX
feature: F0X
epic: E0X
type: execute
---
```

```xml
<objective>
[Story title from user story]

**As a** [from story]
**I want** [from story]
**So that** [from story]

Purpose: [How this story contributes to the feature/epic]
Output: [What will be built - files, components, functionality]
</objective>

<execution_context>
Execute this plan with: /run-us [path-to-this-plan]
The run-us skill handles execution protocol, deviation rules, and summary creation.
</execution_context>

<context>
@.planning/BRIEF.md
@.planning/EXX-name/EPIC.md
@.planning/EXX-name/FXX-name/FEATURE.md
@.planning/EXX-name/FXX-name/US-XXX-name/US-XXX.md
[Relevant source files:]
@src/path/to/relevant.ts
</context>

<acceptance_criteria>
<!-- Copy from user story for reference during execution -->
AC1: [criterion - Given/When/Then summary]
AC2: [criterion]
AC3: [criterion]
</acceptance_criteria>

<tasks>

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

<task type="auto" maps-to="AC2">
  <name>Task 2: [Action-oriented name]</name>
  <files>path/to/file.ext</files>
  <action>[Specific implementation]</action>
  <verify>[Verification command]</verify>
  <done>[Maps to AC2]</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[What Claude built that needs verification]</what-built>
  <how-to-verify>
    1. [Step 1 - specific action]
    2. [Step 2 - what to look for]
    3. [Step 3 - expected behavior]
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>

</tasks>

<verification>
Before declaring story complete:
- [ ] All acceptance criteria met
- [ ] Tests pass (if applicable)
- [ ] No new errors introduced
- [ ] Code follows project conventions
</verification>

<success_criteria>
Story is complete when:
- AC1: [brief] - verified by [method]
- AC2: [brief] - verified by [method]
- AC3: [brief] - verified by [method]
- All tests pass
- No regressions introduced
</success_criteria>

<output>
After completion, create US-XXX-SUMMARY.md in this directory using templates/us-summary.md format.
</output>
```

<task_guidelines>
**Task structure:**
- `type`: "auto" (Claude executes) or "checkpoint:*" (requires human)
- `maps-to`: Which acceptance criterion this task satisfies
- `name`: Action-oriented (verb + object)
- `files`: Which files will be touched
- `action`: Specific implementation instructions
- `verify`: How to check it worked
- `done`: Completion criteria (should match AC)

**Task count:**
- 2-3 tasks maximum per US-PLAN
- Each task maps to 1+ acceptance criteria
- If more tasks needed, split the user story
</task_guidelines>

<ac_mapping>
Every acceptance criterion should map to at least one task.
Tasks can map to multiple ACs if they satisfy multiple criteria.

```xml
<task type="auto" maps-to="AC1,AC2">
  <!-- This task satisfies both AC1 and AC2 -->
</task>
```

Track mapping in success_criteria to ensure coverage.
</ac_mapping>

<checkpoint_placement>
Place checkpoints:
- After UI work (visual verification needed)
- After integration work (functional verification needed)
- At end of story (final acceptance verification)

**Don't checkpoint:**
- After every task (checkpoint fatigue)
- For things Claude can verify programmatically
</checkpoint_placement>
