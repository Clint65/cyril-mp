---
name: plan-us
description: Create an execution plan (US-PLAN.md) for a specific user story. Use when the user wants to plan a story, create a task breakdown for a user story, or prepare a story for implementation. Triggers on "plan story", "prepare story", "break down story", or when a user story exists but has no US-PLAN.md yet.
argument-hint: <story-path>
context: fork
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
  - mcp__context7__*
  - mcp__exa__*
---

Create an execution plan for the user story at $ARGUMENTS.

<objective>
Transform a user story into an executable US-PLAN.md that Claude can run directly.
Explore the codebase, design the architecture, then map acceptance criteria to specific tasks.
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

**3. Explore the codebase:**

Launch 2 `code-explorer` agents in parallel to understand the existing code relevant to this story. Each agent targets a different angle:

Agent 1 prompt example:
> "Analyze the codebase for features similar to this user story: [story summary]. Trace their implementation, identify patterns, and list the 5-10 key files to read. Story acceptance criteria: [AC list]"

Agent 2 prompt example:
> "Map the architecture and abstractions in [relevant area]. Trace the code flows, identify entry points, dependencies, and integration points relevant to: [story summary]"

After agents return, read the key files they identified to build deep understanding.

Skip this step if the story is purely structural (file organization, config, documentation) or if the project has no existing code yet.

**4. Design the architecture:**

Launch 1 `code-architect` agent with the exploration results:

> "Design the implementation for this user story based on the codebase analysis. Story: [story]. Acceptance criteria: [ACs]. Exploration findings: [summary of explorer results]. Propose a concrete implementation blueprint with files to create/modify, components, and build sequence."

Review the architecture and present to the user:
```
Architecture proposal:

Approach: [one-line summary]
Rationale: [why this approach]
Components: [N] files to create/modify

Implementation sequence:
1. [Task] (maps to AC1)
2. [Task] (maps to AC2)
3. [Task] (maps to AC3)

Does this approach work? (yes / adjust)
```

Wait for user confirmation before proceeding.

Skip this step for trivial stories (1 story point, single file change).

**5. Research technical context:**

Identify libraries, frameworks, and APIs mentioned in the story or discovered during exploration.

For each library/framework:
1. Use `mcp__context7__resolve-library-id` to find the Context7 library ID
2. Use `mcp__context7__query-docs` to fetch current documentation for the specific APIs needed
3. Use `mcp__exa__get_code_context_exa` for SDK patterns, integration examples, or APIs not covered by Context7

Include key findings (correct API signatures, current patterns, version-specific behavior) in the task specifications.

Skip this step if the story is purely structural (file organization, config, documentation).

**6. Map acceptance criteria to tasks:**

Using the architecture blueprint, create implementation tasks:
- Each task should have `maps-to` attribute linking to AC
- Tasks should be specific, executable, and reference concrete files from the architecture
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

**7. Scope check:**

If more than 3 tasks needed:
- Note the recommendation to split in the plan output
- Proceed with the plan as-is (user can adjust the file after review)

**8. Determine checkpoint placement:**

Add `checkpoint:human-verify` when:
- Story involves UI (visual verification needed)
- Story involves user flows (functional verification)
- At end of story (final acceptance)

No checkpoints when:
- Pure backend/API work
- All verification is programmatic

**9. Create US-PLAN.md:**

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
Execute this plan with: /run-us [path-to-this-plan]
The run-us skill handles execution protocol, deviation rules, and summary creation.
</execution_context>

<codebase_analysis>
[Summary of code-explorer findings: key patterns, relevant files, existing code to reuse.
Omit if no existing codebase.]
</codebase_analysis>

<architecture>
[Summary of code-architect blueprint: chosen approach, components, rationale.
Omit for trivial stories.]
</architecture>

<context>
@.planning/BRIEF.md
@[path to EPIC.md]
@[path to FEATURE.md]
@$ARGUMENTS
[relevant source files identified by exploration]
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
[2-3 tasks with maps-to attributes. Reference architecture blueprint and technical_research in task instructions.]
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

**10. Report completion:**
```
Plan created: [path to US-PLAN.md]
Tasks: [N] (maps to [N] acceptance criteria)
Checkpoints: [None - autonomous | N checkpoints]
Codebase explored: [yes/no]
Architecture designed: [yes/no]

Review the plan before executing with /run-us [plan-path].
```

</process>

<success_criteria>
- US-PLAN.md created in same directory as user story
- 2-3 tasks defined (or story recommended for split)
- Every AC maps to at least one task
- Codebase explored via code-explorer agents (unless no existing code)
- Architecture designed via code-architect agent (unless trivial story)
- Context references included
- Checkpoint placement decided
- Technical research done via Context7/Exa for code-related stories
</success_criteria>
