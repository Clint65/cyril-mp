# Workflow: Create Handoff

<required_reading>
**Read these files NOW:**
1. templates/continue-here.md
</required_reading>

<purpose>
Create a context handoff file when pausing work. This preserves full context
so a fresh Claude session can pick up exactly where you left off.

**Handoff is a parking lot, not a journal.** Create when leaving, delete when returning.
</purpose>

<when_to_create>
- User says "pack it up", "stopping for now", "save my place"
- Context window at 15% or below (offer to create)
- Context window at 10% (auto-create)
- Switching to different project
</when_to_create>

<process>

<step name="identify_location">
Determine current position in agile structure:

```bash
# Find current epic/feature/story
ls -lt .planning/E*/F*/US-*/US-*-PLAN.md 2>/dev/null | head -1
```

Handoff goes in the current epic directory.
</step>

<step name="gather_context">
Collect everything needed for seamless resumption:

1. **Current position**: Which epic, feature, story, task
2. **Work completed**: What's done this session
3. **Work remaining**: What's left in current story
4. **AC status**: Which acceptance criteria are met
5. **Decisions made**: Why things were done this way
6. **Blockers/issues**: Anything stuck
7. **Mental context**: The "vibe" - what you were thinking
</step>

<step name="write_handoff">
Use template from `templates/continue-here.md`.

Write to `.planning/EXX-name/.continue-here.md`:

```yaml
---
epic: EXX-name
feature: FXX-name
story: US-XXX
task: 2
total_tasks: 3
status: in_progress
last_updated: [ISO timestamp]
---
```

Then markdown body with:
- current_state
- completed_work
- remaining_work
- acceptance_criteria_status
- decisions_made
- blockers
- context
- next_action
</step>

<step name="git_commit_wip">
Commit handoff as WIP:

```bash
git add .planning/
git commit -m "$(cat <<'EOF'
wip: US-XXX paused at task [X]/[Y]

Current: [task name]
[If blocked:] Blocked: [reason]
EOF
)"
```

Confirm: "Committed: wip: US-XXX paused at task [X]/[Y]"
</step>

<step name="handoff_confirmation">
Require acknowledgment:

"Handoff created: .planning/[EXX-name]/.continue-here.md

Current state:
- Epic: [EXX-name]
- Feature: [FXX-name]
- Story: US-XXX ([X] of [Y] tasks complete)
- AC Status: [X]/[Y] met
- Committed as WIP

To resume: Invoke this skill in a new session.

Confirmed?"

Wait for acknowledgment before ending.
</step>

</process>

<context_trigger>
**Auto-handoff at 10% context:**

When system warning shows ~20k tokens remaining:
1. Complete current atomic operation (don't leave broken state)
2. Create handoff automatically
3. Tell user: "Context limit reached. Handoff created at [location]."
4. Stop working - don't start new tasks

**Warning at 15%:**
"Context getting low (~30k remaining). Create handoff now or push through?"
</context_trigger>

<handoff_lifecycle>
```
Working           -> No handoff exists
"Pack it up"      -> CREATE .continue-here.md
[Session ends]
[New session]
"Resume"          -> READ handoff, then DELETE it
Working           -> No handoff (context is fresh)
Story complete    -> Ensure no stale handoff exists
```

Handoff is temporary. If it persists after resuming, it's stale.
</handoff_lifecycle>

<success_criteria>
Handoff is complete when:
- [ ] .continue-here.md exists in current epic directory
- [ ] YAML frontmatter has epic, feature, story, task, status, timestamp
- [ ] Body has: AC status, completed work, remaining work, decisions, context
- [ ] WIP committed to git
- [ ] User knows how to resume
</success_criteria>
