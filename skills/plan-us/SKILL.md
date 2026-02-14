---
name: plan-us
description: Create an execution plan (US-PLAN.md) for a specific user story
argument-hint: <story-path>
context: fork
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - mcp__context7__*
  - mcp__exa__*
---

Create an execution plan for the user story at $ARGUMENTS.

<objective>
Transform a user story into an executable US-PLAN.md that Claude can run directly.
Map acceptance criteria to specific implementation tasks.
</objective>

<process>

**1. Read the user story:**
```bash
cat $ARGUMENTS
```

Parse:
- Story format (As a/I want/So that)
- Acceptance criteria (Given/When/Then)
- Story points
- Technical notes
- Suggested tasks

**2. Check dependencies:**
```bash
# Check what stories are complete in this feature
ls $(dirname $ARGUMENTS)/US-*-SUMMARY.md 2>/dev/null
```

If this story depends on incomplete stories, warn the user.

**3. Research technical context:**

Identify libraries, frameworks, and APIs mentioned in the story or inferred from the project.

For each library/framework:
1. Use `mcp__context7__resolve-library-id` to find the Context7 library ID
2. Use `mcp__context7__query-docs` to fetch current documentation for the specific APIs needed
3. Use `mcp__exa__get_code_context_exa` for SDK patterns, integration examples, or APIs not covered by Context7

Include key findings (correct API signatures, current patterns, version-specific behavior) in the task specifications. This ensures the execution plan references up-to-date APIs, not stale training data.

Skip this step if the story is purely structural (file organization, config, documentation).

**4. Map acceptance criteria to tasks:**

For each AC, create 1+ implementation tasks:
- Each task should have `maps-to` attribute linking to AC
- Tasks should be specific and executable
- Target 2-3 tasks total (split story if more needed)

Log the mapping in your output:
```
Mapping acceptance criteria to tasks:

AC1: [criterion]
  -> Task 1: [implementation task]

AC2: [criterion]
  -> Task 2: [implementation task]

AC3: [criterion]
  -> Task 3: [implementation task]
```

**5. Scope check:**

If more than 3 tasks needed:
- Note the recommendation to split in the plan output
- Proceed with the plan as-is (user can adjust the file after review)

**6. Determine checkpoint placement:**

Add `checkpoint:human-verify` when:
- Story involves UI (visual verification needed)
- Story involves user flows (functional verification)
- At end of story (final acceptance)

No checkpoints when:
- Pure backend/API work
- All verification is programmatic

**7. Create US-PLAN.md:**

Write to same directory as user story:
- `$(dirname $ARGUMENTS)/US-XXX-PLAN.md`

Use this structure:
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
[Story title]

**As a** [from story]
**I want** [from story]
**So that** [from story]

Purpose: [contribution to feature]
Output: [what will be built]
</objective>

<execution_context>
@~/.claude/skills/create-plans-agile/workflows/execute-user-story.md
@~/.claude/skills/create-plans-agile/templates/us-summary.md
</execution_context>

<context>
@.planning/BRIEF.md
@[path to EPIC.md]
@[path to FEATURE.md]
@$ARGUMENTS
[relevant source files]
</context>

<acceptance_criteria>
AC1: [criterion]
AC2: [criterion]
AC3: [criterion]
</acceptance_criteria>

<technical_research>
[Key API signatures, patterns, and version-specific notes from Context7/Exa research.
Omit if story is non-technical.]
</technical_research>

<tasks>
[2-3 tasks with maps-to attributes. Reference technical_research findings in task instructions.]
</tasks>

<verification>
- [ ] All acceptance criteria met
- [ ] Tests pass
- [ ] No new errors
</verification>

<success_criteria>
- AC1: verified by [method]
- AC2: verified by [method]
- AC3: verified by [method]
</success_criteria>

<output>
Create US-XXX-SUMMARY.md in this directory.
</output>
```

**8. Report completion:**
```
Plan created: [path to US-PLAN.md]
Tasks: [N] (maps to [N] acceptance criteria)
Checkpoints: [None - autonomous | N checkpoints]

Review the plan before executing with /run-us [plan-path].
```

</process>

<success_criteria>
- US-PLAN.md created in same directory as user story
- 2-3 tasks defined (or story recommended for split)
- Every AC maps to at least one task
- Context references included
- Checkpoint placement decided
- Technical research done via Context7/Exa for code-related stories
</success_criteria>
