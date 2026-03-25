---
name: create-plans-agile
description: Create Agile project plans with Epic/Feature/User Story structure optimized for solo agentic development. Use when planning projects, organizing a backlog, creating user stories, defining epics or features, or using Scrum-like methodology. Triggers on requests to plan a project, structure work into stories, estimate effort with story points, create a roadmap, or manage context handoffs. Also use when the user mentions "agile", "backlog", "sprint", "user story", "epic", "feature", or wants to break down a project into deliverable increments.
argument-hint: [what to plan]
---

<essential_principles>

<principle name="solo_developer_plus_claude">
You are planning for ONE person (the user) and ONE implementer (Claude).
No teams. No standups. No sprint ceremonies. No coordination overhead.
The user is the product owner. Claude is the builder.
Agile structure without agile ceremony.
</principle>

<principle name="stories_are_prompts">
US-PLAN.md is not a document that gets transformed into a prompt.
US-PLAN.md IS the prompt. It contains:
- Objective (story with As a/I want/So that)
- Context (@file references)
- Acceptance criteria (Given/When/Then)
- Tasks (type, files, action, verify, done, maps-to)
- Verification (overall checks)
- Success criteria (AC mapping)
- Output (US-SUMMARY.md specification)

When planning a user story, you are writing the prompt that will execute it.
</principle>

<principle name="mandatory_workflow">
**Plan and summary are REQUIRED for all stories.**

The formal workflow MUST be followed:
1. `/plan-us <story-path>` - Creates US-XXX-PLAN.md
2. `/run-us <plan-path>` - Executes plan, creates US-XXX-SUMMARY.md
3. Complete story - Updates tracking documents

**Do NOT:**
- Implement stories directly without a plan
- Mark stories complete without a summary
- Skip the planning step to "save time"

**Why mandatory:**
- Plans ensure quality through explicit task breakdown
- Plans enable AC verification
- Summaries document what was built
- Summaries enable progress tracking
- Skipping creates technical debt and tracking gaps
</principle>

<principle name="scope_control">
User story plans must complete within ~50% of context usage to maintain consistent quality.

**The quality degradation curve:**
- 0-30% context: Peak quality (comprehensive, thorough, no anxiety)
- 30-50% context: Good quality (engaged, manageable pressure)
- 50-70% context: Degrading quality (efficiency mode, compression)
- 70%+ context: Poor quality (rushed work)

**Solution:** Each US-PLAN contains 2-3 tasks maximum. Split larger stories.

**Story point guidance:**
- 1-3 points: Single US-PLAN
- 5 points: Consider splitting
- 8+ points: Must split into multiple stories

See: references/scope-estimation.md
</principle>

<principle name="human_checkpoints">
**Claude automates everything that has a CLI or API.** Checkpoints are for verification and decisions, not manual work.

**Checkpoint types:**
- `checkpoint:human-verify` - Human confirms Claude's automated work (visual checks, UI verification)
- `checkpoint:decision` - Human makes implementation choice (auth provider, architecture)

**Rarely needed:** `checkpoint:human-action` - Only for actions with no CLI/API

**Critical rule:** If Claude CAN do it via CLI/API/tool, Claude MUST do it.

See: references/checkpoints.md, references/cli-automation.md
</principle>

<principle name="deviation_rules">
Plans are guides, not straitjackets. During execution, deviations are handled automatically:

1. **Auto-fix bugs** - Broken behavior -> fix immediately, document in Summary
2. **Auto-add missing critical** - Security/correctness gaps -> add immediately, document
3. **Auto-fix blockers** - Can't proceed -> fix immediately, document
4. **Ask about architectural** - Major structural changes -> stop and ask user
5. **Log enhancements** - Nice-to-haves -> auto-log to ISSUES.md, continue

All deviations documented in US-SUMMARY.md.
</principle>

<principle name="agile_without_ceremony">
Structure without overhead:
- Epics for grouping related value
- Features for coherent functionality
- User stories for deliverable increments
- Story points for relative sizing

NO:
- Sprint planning meetings
- Daily standups
- Sprint retrospectives
- Velocity tracking ceremonies
- Burndown charts

Ship story by story. Track progress in ROADMAP.md and FEATURE.md.
</principle>

<principle name="context_awareness">
Monitor token usage via system warnings.

**At 25% remaining**: Mention context getting full
**At 15% remaining**: Pause, offer handoff
**At 10% remaining**: Auto-create handoff, stop

Never start large operations below 15% without user confirmation.
</principle>

<principle name="user_gates">
Never charge ahead at critical decision points. Use gates:
- **AskUserQuestion**: Structured choices (2-4 options)
- **Inline questions**: Simple confirmations
- **Decision gate loop**: "Ready, or ask more questions?"

Mandatory gates:
- Before writing US-PLAN.md (confirm task breakdown)
- After story point estimation
- On verification failures
- Before starting next story with previous issues

See: references/user-gates.md
</principle>

<principle name="git_versioning">
All planning artifacts are version controlled. Commit outcomes, not process.

- Check for repo on invocation, offer to initialize
- Commit only at: initialization, story completion, feature completion, handoff
- Intermediate artifacts (US-PLAN.md, EPIC.md) NOT committed separately
- Commit format: `feat(US-XXX): [story title]`

See: references/git-integration.md
</principle>

</essential_principles>

<context_scan>
**Run on every invocation** to understand current state:

```bash
# Check git status
git rev-parse --git-dir 2>/dev/null || echo "NO_GIT_REPO"

# Check for planning structure
ls -la .planning/ 2>/dev/null
ls -la .planning/E*/ 2>/dev/null

# Find any continue-here files
find .planning -name ".continue-here.md" -type f 2>/dev/null

# Check for existing artifacts
[ -f .planning/BRIEF.md ] && echo "BRIEF: exists"
[ -f .planning/ROADMAP.md ] && echo "ROADMAP: exists"
```

**If NO_GIT_REPO detected:**
Inline question: "No git repo found. Initialize one? (Recommended for version control)"
If yes: `git init`

**Present findings before intake question.**
</context_scan>

<domain_expertise>
If the user has domain-specific expertise skills in `~/.claude/skills/expertise/`, they can be loaded to enrich planning with domain knowledge.

```bash
ls ~/.claude/skills/expertise/ 2>/dev/null || echo "NO_DOMAIN_EXPERTISE"
```

If expertise found and relevant to the project, confirm with user before loading:
```
Found domain expertise: [list]. Load for planning? (Y / none)
```

If loaded, read the domain SKILL.md and use its principles during roadmap and story creation.
If no expertise directory exists, skip entirely.
</domain_expertise>

<intake>
Based on scan results, present context-aware options:

**If handoff found:**
```
Found handoff: .planning/EXX-name/.continue-here.md
[Summary of state from handoff]

1. Resume from handoff
2. Discard handoff, start fresh
3. Different action
```

**If agile structure exists:**
```
Project: [from BRIEF or directory]
Epic: E0X - [name] ([status])
Feature: F0X - [name] ([X]/[Y] stories complete)
Current story: US-XXX - [name]

What would you like to do?
1. Continue current story
2. Plan next story
3. Define more stories for current feature
4. Create handoff (stopping for now)
5. View progress
6. Something else
```

**If no planning structure:**
```
No planning structure found.

What would you like to do?
1. Start new project (create brief)
2. Create agile roadmap with epics
3. Jump straight to epic/feature/story definition
4. Get guidance on approach
```

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| "brief", "new project", "start", 1 (no structure) | `workflows/create-brief.md` |
| "roadmap", "epics", 2 (no structure) | `workflows/create-roadmap-agile.md` |
| "epic", "define epic" | `workflows/create-epic.md` |
| "feature", "define feature" | `workflows/create-feature.md` |
| "story", "user story", "define story" | `workflows/create-user-story.md` |
| "plan story", "plan us" | `workflows/plan-user-story.md` |
| "execute", "run", "build", "implement" | See routing logic below |
| "handoff", "pack up", "stopping" | `workflows/handoff.md` |
| "resume", "continue", 1 (has handoff) | `workflows/resume.md` |
| "complete", "done", "finish story" | `workflows/complete-user-story.md` |
| "guidance", "help" | `workflows/get-guidance.md` |

**Routing for "execute/run/build/implement":**
1. Check if US-XXX-PLAN.md exists for the story
2. **If plan exists:** Exit skill, instruct user to run `/run-us <plan-path>`
3. **If NO plan:** Redirect to `workflows/plan-user-story.md` first

```
No plan found for US-XXX.

Plan is required before implementation.
Creating plan now...
```

**Critical:**
- Story execution requires a plan (mandatory workflow)
- Use `/run-us` for execution (context efficiency)
- Do NOT implement stories without plans

**After reading the workflow, follow it exactly.**
</routing>

<hierarchy>
The agile planning hierarchy:

```
BRIEF.md              -> Human vision (you read this)
    |
ROADMAP.md            -> Epic structure (overview)
    |
EPIC.md               -> Epic definition + features
    |
FEATURE.md            -> Feature definition + stories
    |
US-XXX.md             -> User story (As a/I want/So that + AC)
    |
US-XXX-PLAN.md        -> THE PROMPT (Claude executes this)
    |
US-XXX-SUMMARY.md     -> Outcome (existence = story done)
```

**Rules:**
- Roadmap requires Brief (or prompts to create one)
- Epics can be created from roadmap
- Features belong to epics
- User stories belong to features
- US-PLAN.md IS the execution prompt
- US-SUMMARY.md existence marks story complete
- Each level can look UP for context
</hierarchy>

<output_structure>
All planning artifacts go in `.planning/`:

```
.planning/
├── BRIEF.md                           # Human vision
├── ROADMAP.md                         # Epic structure + tracking
├── ISSUES.md                          # Deferred enhancements log
├── E01-authentication/                # Epic folder
│   ├── EPIC.md                        # Epic definition
│   ├── F01-login/                     # Feature folder
│   │   ├── FEATURE.md                 # Feature definition
│   │   ├── US-001-basic-login/        # User story folder
│   │   │   ├── US-001.md              # User story definition
│   │   │   ├── US-001-PLAN.md         # Execution plan
│   │   │   └── US-001-SUMMARY.md      # Outcome
│   │   └── US-002-remember-me/
│   │       ├── US-002.md
│   │       ├── US-002-PLAN.md
│   │       └── US-002-SUMMARY.md
│   └── F02-registration/
│       ├── FEATURE.md
│       └── US-003-signup/
│           └── ...
└── E02-dashboard/
    └── ...
```

**Naming convention:**
- Epics: `EXX-name/` (e.g., E01-authentication/)
- Features: `FXX-name/` (e.g., F01-login/)
- Story folders: `US-XXX-name/` (e.g., US-001-basic-login/)
- Stories: `US-XXX.md` (e.g., US-001.md)
- Plans: `US-XXX-PLAN.md` (e.g., US-001-PLAN.md)
- Summaries: `US-XXX-SUMMARY.md` (e.g., US-001-SUMMARY.md)
</output_structure>

<reference_index>
All in `references/`:

**Structure:** scope-estimation.md
**Patterns:** context-management.md, checkpoints.md, cli-automation.md
**Process:** user-gates.md, git-integration.md
</reference_index>

<templates_index>
All in `templates/`:

| Template | Purpose |
|----------|---------|
| brief.md | Project vision document |
| roadmap-agile.md | Epic-based roadmap with progress |
| epic.md | Epic definition |
| feature.md | Feature definition + story list |
| user-story.md | User story with AC |
| us-plan.md | Executable story prompt (US-PLAN.md) |
| us-summary.md | Story outcome (US-SUMMARY.md) |
| continue-here.md | Context handoff format |
</templates_index>

<workflows_index>
All in `workflows/`:

| Workflow | Purpose |
|----------|---------|
| create-brief.md | Create project vision document |
| create-roadmap-agile.md | Define epics from brief |
| create-epic.md | Define epic with features |
| create-feature.md | Define feature with stories |
| create-user-story.md | Define user story with AC |
| plan-user-story.md | Create executable story prompt |
| execute-user-story.md | Run story prompt, create summary |
| complete-user-story.md | Mark story done, update tracking |
| handoff.md | Create context handoff for pausing |
| resume.md | Load handoff, restore context |
| get-guidance.md | Help decide planning approach |
</workflows_index>

<success_criteria>
Planning skill succeeds when:
- Context scan runs before intake
- Appropriate workflow selected based on state
- US-PLAN.md IS the executable prompt (not separate)
- Hierarchy is maintained (brief -> roadmap -> epic -> feature -> story)
- Handoffs preserve full context for resumption
- Context limits are respected (auto-handoff at 10%)
- Deviations handled automatically per embedded rules
- All work fully documented in US-SUMMARY.md
- Story execution uses /run-us command (not skill invocation)
</success_criteria>
