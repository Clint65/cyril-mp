# Workflow: Execute User Story

<required_reading>
**Read these files when executing a US-PLAN.md:**
1. The US-PLAN.md being executed
2. templates/us-summary.md
3. references/checkpoints.md (if plan has checkpoints)
</required_reading>

<purpose>
Execute a US-PLAN.md file, implementing all tasks and creating US-SUMMARY.md.
This workflow is invoked by /run-us command, not directly.
</purpose>

<execution_protocol>

<step name="load_plan">
Read the US-PLAN.md file completely.

Parse:
- objective (what we're building)
- context (files to reference)
- acceptance_criteria (what we're verifying)
- tasks (what to execute)
- verification (overall checks)
- success_criteria (AC mapping)
</step>

<step name="read_context">
Load all @file references from the context section:
- BRIEF.md
- EPIC.md
- FEATURE.md
- User story file
- Source files mentioned

This establishes implementation context.
</step>

<step name="execute_tasks">
For each task in sequence:

**For type="auto" tasks:**
1. Announce: "Task [N]: [name]"
2. Read files mentioned in `<files>`
3. Execute `<action>` instructions
4. Run `<verify>` checks
5. Confirm `<done>` criteria met
6. Continue to next task

**For type="checkpoint:human-verify" tasks:**
1. Display checkpoint clearly:
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
2. **STOP and wait for user response**
3. If issues reported: fix them, re-verify
4. If approved: continue to next task

**For type="checkpoint:decision" tasks:**
1. Display decision with options
2. **STOP and wait for user choice**
3. Record decision
4. Continue based on choice
</step>

<step name="deviation_handling">
During execution, apply deviation rules automatically:

**Rule 1: Auto-fix bugs**
- Found broken behavior -> fix immediately
- Document in summary: "Fixed: [issue] - [resolution]"

**Rule 2: Auto-add missing critical**
- Security gap found -> add fix immediately
- Document in summary: "Added: [what] - [why critical]"

**Rule 3: Auto-fix blockers**
- Can't proceed -> fix the blocker
- Document in summary: "Unblocked: [issue] - [resolution]"

**Rule 4: Ask about architectural**
- Major structural change needed -> STOP
- Ask user: "This requires architectural change: [description]. Proceed?"
- Document decision in summary

**Rule 5: Log enhancements**
- Nice-to-have found -> log to ISSUES.md
- Continue without implementing
- Document in summary: "Deferred: [enhancement] -> ISSUES.md"
</step>

<step name="quality_review">
After all tasks complete, launch a `code-reviewer` agent to review the changes:

The agent reviews `git diff` against acceptance criteria and project conventions.
Present findings to user with verdict (APPROVE / APPROVE WITH FIXES / REQUEST CHANGES).
User decides: fix now, fix later, or skip.

**Skip if:** non-code story (documentation, config only) or no unstaged changes.
</step>

<step name="verify_acceptance_criteria">
After review (or after implementation if review was skipped), verify each acceptance criterion:

```
Acceptance Criteria Verification:

AC1: [criterion]
  Status: PASS
  Verified by: [method - test, manual, build]

AC2: [criterion]
  Status: PASS
  Verified by: [method]

AC3: [criterion]
  Status: PASS
  Verified by: [method]

All acceptance criteria met.
```

If any AC fails: Report failure, ask how to proceed.
</step>

<step name="create_summary">
Create US-XXX-SUMMARY.md using templates/us-summary.md.

Include:
- Substantive one-liner (what shipped)
- Story reference (As a/I want/So that)
- AC status table (all DONE with verification method)
- Accomplishments (key outcomes)
- Files created/modified
- Decisions made
- Deviations from plan (if any)
- Definition of done checklist
- Story metrics (estimated vs actual)
- Next story pointer
</step>

<step name="update_tracking">
Update tracking files:

1. **FEATURE.md**: Mark story complete, update counts
2. **EPIC.md**: Update feature progress (if feature complete)
3. **ROADMAP.md**: Update story counts
</step>

<step name="offer_next_actions">
After implementation, present options:

```
════════════════════════════════════════
STORY IMPLEMENTED: US-XXX
════════════════════════════════════════

[Substantive one-liner]

Acceptance Criteria: [X]/[X] met
Files: [N] created/modified
Deviations: [None | X logged]

Summary: .planning/EXX-name/FXX-name/US-XXX-name/US-XXX-SUMMARY.md

Changes are NOT committed yet.

What's next?
1. Create & run tests for this story (/test-us)
2. Commit this story (/commit-us)
3. Plan next story
4. Done for now
════════════════════════════════════════
```

**Do NOT auto-commit.** User decides when to commit via `/commit-us`.
</step>

</execution_protocol>

<context_monitoring>
Monitor context usage during execution:

**At 25% remaining**: Note in output
**At 15% remaining**: "Context low. Complete current task, then create handoff."
**At 10% remaining**: Auto-create handoff, stop execution
</context_monitoring>

<success_criteria>
Execution is complete when:
- [ ] All tasks executed
- [ ] All checkpoints passed
- [ ] All acceptance criteria verified
- [ ] US-SUMMARY.md created
- [ ] Tracking files updated
- [ ] User offered test/commit options
</success_criteria>
