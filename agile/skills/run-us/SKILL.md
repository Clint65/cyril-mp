---
name: run-us
description: Execute a US-PLAN.md file (user story implementation plan). Use when the user wants to implement, build, execute, or run a planned user story. Triggers on "run story", "execute plan", "implement story", "build story", or when a US-PLAN.md exists and the user wants to start coding.
argument-hint: <plan-path>
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
  - mcp__context7__*
  - mcp__exa__*
---

Execute the user story plan at $ARGUMENTS using **intelligent segmentation** for optimal quality.

<objective>
Run a US-PLAN.md file, implementing all tasks and creating US-SUMMARY.md.
Maintain consistent quality through intelligent execution strategies.
</objective>

<process>

**1. Verify plan exists and is unexecuted:**
```bash
cat $ARGUMENTS
```

Check if corresponding SUMMARY exists:
```bash
ls $(dirname $ARGUMENTS)/$(basename $ARGUMENTS -PLAN.md)-SUMMARY.md 2>/dev/null
```

If SUMMARY exists: "This story appears complete. Re-run anyway? (yes / no)"
If plan doesn't exist: Error and exit.

**2. Parse plan and determine execution strategy:**

Read the plan and extract:
- `<objective>` section
- `<execution_context>` files to load
- `<context>` files to reference
- `<acceptance_criteria>` to verify
- `<tasks>` to execute
- `<verification>` checklist
- `<success_criteria>` AC mapping

Analyze checkpoint structure:
```bash
grep "type=\"checkpoint" $ARGUMENTS
```

**3. Determine routing strategy:**

**A. Fully Autonomous (no checkpoints):**
- All tasks are `type="auto"`
- Can execute in subagent with fresh context
- Spawn Agent to execute entire plan
- Agent creates SUMMARY, reports back

**B. Segmented Execution (verify-only checkpoints):**
- Has `checkpoint:human-verify` but no decisions that affect later tasks
- Execute segments between checkpoints via subagent
- Return to main for verification
- Continue after approval

**C. Decision-Dependent (has decision checkpoints):**
- Has `checkpoint:decision` that affects implementation
- Execute entirely in main context
- Decisions inform subsequent tasks

**4. Execute based on strategy:**

**For Strategy A (Fully Autonomous):**
```
This plan has no checkpoints - executing autonomously.

Spawning subagent to implement: [story title]
```
- Use Agent tool to spawn execution
- Subagent runs the plan tasks and reports back (does NOT create SUMMARY)
- After subagent returns, continue in main context for: code review (step 7), AC verification (step 8), and SUMMARY creation (step 9)

**For Strategy B (Segmented):**
For each segment between checkpoints:
1. Execute auto tasks
2. Hit checkpoint - display verification request
3. Wait for user approval
4. Continue to next segment

**For Strategy C (Decision-Dependent):**
Execute tasks sequentially in main context:
1. Announce each task
2. Load files, execute action
3. Run verification
4. At checkpoints: stop and wait
5. Continue based on responses

**5. Task execution protocol:**

For each `type="auto"` task:
```
════════════════════════════════════════
Task [N]/[Total]: [Task name]
════════════════════════════════════════

Files: [files from task]
Action: [executing...]
```

**Before writing code**, look up current documentation:
1. Check the plan's `<technical_research>` section for pre-fetched API references
2. If the task uses libraries/frameworks not covered by the research, or if more detail is needed:
   - Use `mcp__context7__resolve-library-id` + `mcp__context7__query-docs` for library documentation
   - Use `mcp__exa__get_code_context_exa` for SDK patterns, integration examples, or recent API changes
3. Use the fetched documentation to write code with correct, current API signatures

Skip this lookup for non-code tasks (config changes, file moves, documentation).

**Then execute:**
- Read specified files
- Execute action instructions using up-to-date API references
- Run verify command
- Confirm done criteria

For `type="checkpoint:human-verify"`:
```
════════════════════════════════════════
CHECKPOINT: Verification Required
════════════════════════════════════════

Task [N] of [Total]: [Name]

I built: [what-built content]

How to verify:
[how-to-verify content]

[resume-signal content]
════════════════════════════════════════
```
**STOP and wait for response.**

**6. Apply deviation rules during execution:**

- **Rule 1:** Bug found -> fix immediately, document
- **Rule 2:** Missing critical -> add immediately, document
- **Rule 3:** Blocker -> fix immediately, document
- **Rule 4:** Architectural change needed -> STOP, ask user
- **Rule 5:** Enhancement idea -> log to ISSUES.md, continue

**7. Quality review (code-reviewer agent):**

After all tasks are implemented, launch a `code-reviewer` agent to review the changes:

> "Review the unstaged changes (`git diff`) for this user story implementation. Acceptance criteria to verify: [list ACs from plan]. Check for bugs, code quality, project conventions, and whether the implementation satisfies each AC. Only report issues with confidence >= 75."

Present the review findings to the user:
```
════════════════════════════════════════
CODE REVIEW: [story title]
════════════════════════════════════════

Verdict: [APPROVE | APPROVE WITH FIXES | REQUEST CHANGES]

[Critical issues if any]
[Important issues if any]

AC alignment:
- AC1: [PASS/FAIL]
- AC2: [PASS/FAIL]

What do you want to do?
1. Fix issues now
2. Fix later (proceed to commit)
3. Skip review findings
════════════════════════════════════════
```

If the user chooses to fix: apply the fixes, then re-run the reviewer if needed.

Skip this step if:
- The story is non-code (documentation, config only)
- There are no unstaged changes (nothing to review)

**8. Verify acceptance criteria:**

After review (or after implementation if review was skipped):
```
Acceptance Criteria Verification:

AC1: [criterion]
  Status: PASS | FAIL
  Verified by: [method]

AC2: [criterion]
  Status: PASS | FAIL
  Verified by: [method]

[All criteria status...]
```

If any FAIL: Report and ask how to proceed.

**9. Create US-SUMMARY.md:**

Write to same directory as plan:
```markdown
# US-XXX: [Story Title] Summary

**[Substantive one-liner]**

## Story Completed

**As a** [from story]
**I want** [from story]
**So that** [from story]

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | [brief] | Done | [method] |
| AC2 | [brief] | Done | [method] |

## Accomplishments

- [Key outcome 1]
- [Key outcome 2]

## Files Created/Modified

- `path/to/file.ts` - [description]

## Deviations from Plan

[None | list deviations per rules]

## Code Review

[Review verdict: APPROVE / APPROVE WITH FIXES / Skipped]
[Issues found and resolved, if any]

---
*Story: US-XXX*
*Completed: [date]*
```

**10. Update tracking files:**

Update FEATURE.md - mark story complete:
```bash
# Read and update feature file
cat $(dirname $(dirname $ARGUMENTS))/FEATURE.md
```

Update counts in EPIC.md and ROADMAP.md if needed.

**11. Report completion:**
```
════════════════════════════════════════
STORY IMPLEMENTED: US-XXX
════════════════════════════════════════

[Substantive one-liner]

Acceptance Criteria: [X]/[X] met
Files: [N] created/modified
Deviations: [None | X logged]

Summary: [path to SUMMARY]

Changes are NOT committed yet.

What's next?
1. Create & run tests for this story (/test-us)
2. Commit this story (/commit-us)
3. Plan next story
4. Done for now
════════════════════════════════════════
```

**Do NOT auto-commit.** User decides when to commit via `/commit-us`.

</process>

<context_monitoring>
Monitor context usage during execution:
- **25% remaining:** Note in output
- **15% remaining:** "Context low. Completing current task, then creating handoff."
- **10% remaining:** Auto-create handoff, stop execution
</context_monitoring>

<success_criteria>
- All tasks executed
- All checkpoints passed
- Code review completed via code-reviewer agent (unless non-code story)
- All acceptance criteria verified
- US-SUMMARY.md created
- Tracking files updated
- User offered test/commit options
- Code tasks used Context7/Exa for current API references
</success_criteria>
