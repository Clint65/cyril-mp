# Workflow: Complete User Story

<required_reading>
**Read these files NOW:**
1. The US-SUMMARY.md that was created
2. Current FEATURE.md
3. Current EPIC.md
4. ROADMAP.md
</required_reading>

<purpose>
Mark a user story as complete and update all tracking documents.
This workflow handles the bookkeeping after story execution.
</purpose>

<mandatory_workflow>
**Plan and summary are REQUIRED.** Stories must follow the formal workflow:

1. `/plan-us <story-path>` - Creates US-XXX-PLAN.md
2. `/run-us <plan-path>` - Executes plan, creates US-XXX-SUMMARY.md
3. This workflow - Updates tracking documents

Do NOT mark stories complete without plan and summary.
</mandatory_workflow>

<process>

<step name="verify_summary_exists">
```bash
cat .planning/EXX-name/FXX-name/US-XXX-name/US-XXX-SUMMARY.md
```

**If no summary exists:**
```
════════════════════════════════════════
ERROR: No summary found
════════════════════════════════════════

Story US-XXX has not been properly implemented.

Required workflow:
1. First: /plan-us <story-path>
2. Then: /run-us <plan-path>
3. Finally: Complete story (this workflow)

Please run the planning workflow first.
════════════════════════════════════════
```

**STOP** - Do not proceed without summary.

**If summary exists:** Continue to update tracking.
</step>

<step name="update_feature">
Read and update FEATURE.md:

```bash
cat .planning/EXX-name/FXX-name/FEATURE.md
```

Update the story table:
- Change status from "In progress" to "Complete"
- Add completion date
- Update totals

```markdown
## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-001 | [title] | 3 | Complete | 2025-01-15 |  <- Updated
| US-002 | [title] | 2 | Not started | - |
```
</step>

<step name="check_feature_complete">
If all stories in feature are complete:

1. Update FEATURE.md status to "Complete"
2. Check off feature-level acceptance criteria
3. Update EPIC.md feature status
</step>

<step name="update_epic">
Read and update EPIC.md:

```bash
cat .planning/EXX-name/EPIC.md
```

Update the feature table:
- Update story completion counts
- Update point counts
- If feature complete, mark it

```markdown
## Features

| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F01 | Login | 2/3 | 5/8 | In progress |  <- Updated counts
```
</step>

<step name="check_epic_complete">
If all features in epic are complete:

1. Update EPIC.md status to "Complete"
2. Add completion date
3. Update ROADMAP.md epic status
4. Consider milestone commit
</step>

<step name="update_roadmap">
Update ROADMAP.md progress table:

```bash
cat .planning/ROADMAP.md
```

Update:
```markdown
## Progress

| Epic | Features | Stories | Points | Status | Completed |
|------|----------|---------|--------|--------|-----------|
| E01: Auth | 2 | 3/5 | 8/15 | In progress | - |  <- Updated
```
</step>

<step name="report_status">
```
Story US-XXX marked complete.

Progress:
- Feature F0X: [X]/[Y] stories ([XX]/[YY] points)
- Epic E0X: [X]/[Y] features
- Overall: [X]/[Y] stories complete

[If feature complete:]
Feature F0X is now complete!

[If epic complete:]
Epic E0X is now complete! Consider creating a milestone.

What's next?
1. Plan next story
2. View overall progress
3. Done for now
```
</step>

</process>

<milestone_check>
When an epic completes, offer milestone creation:

```
Epic E0X: [Name] is complete!

All features delivered:
- F01: [name] (X stories)
- F02: [name] (X stories)

Create a milestone marker? (yes / no)
```

If yes: Run complete-milestone workflow (or equivalent)
</milestone_check>

<success_criteria>
Story completion is recorded when:
- [ ] US-SUMMARY.md exists (mandatory)
- [ ] FEATURE.md story table updated
- [ ] EPIC.md feature table updated
- [ ] ROADMAP.md progress table updated
- [ ] User informed of current progress
</success_criteria>
