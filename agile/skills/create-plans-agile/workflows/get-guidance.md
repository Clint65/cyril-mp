# Workflow: Get Guidance

<purpose>
Help user understand the agile planning approach and decide how to proceed.
</purpose>

<common_questions>

<question name="when_to_use_agile">
**Q: When should I use this agile skill vs regular create-plans?**

Use **create-plans-agile** when:
- You want Scrum-like user story structure
- You prefer As a/I want/So that format
- You want story point estimation
- You like Given/When/Then acceptance criteria
- Project has clear user-facing features

Use **create-plans** (regular) when:
- You prefer phase-based planning
- Project is more technical/infrastructure focused
- You don't need story point tracking
- Simpler structure is preferred
</question>

<question name="story_vs_task">
**Q: What's the difference between a user story and a task?**

**User Story (US-XXX.md):**
- User-facing value description
- As a [user] I want [capability] so that [benefit]
- Has acceptance criteria (Given/When/Then)
- Estimated in story points (1-5)
- Lives in its own US-XXX-name/ folder within FEATURE

**Task (in US-PLAN.md):**
- Implementation step
- Technical action Claude executes
- Has files, action, verify, done
- 2-3 tasks per story plan
- Maps to acceptance criteria
</question>

<question name="how_to_split">
**Q: How do I know when to split a story?**

Split when:
- More than 5 story points estimated
- More than 4 acceptance criteria
- More than 3 tasks needed in plan
- Multiple technical domains involved
- Story feels "too big"

Split by:
- Acceptance criteria grouping
- User journey stages
- Technical layers (backend/frontend)
- Happy path vs edge cases
</question>

<question name="hierarchy_explained">
**Q: Can you explain the hierarchy?**

```
BRIEF.md          - What we're building (human vision)
    |
ROADMAP.md        - Epic overview (2-4 epics)
    |
EPIC.md           - Epic details + features (2-4 features)
    |
FEATURE.md        - Feature details + stories (2-5 stories)
    |
US-XXX.md         - User story definition (As a/I want/So that + AC)
    |
US-XXX-PLAN.md    - Executable plan (tasks Claude runs)
    |
US-XXX-SUMMARY.md - What shipped (outcome documentation)
```

Each level builds on the previous. Look UP for context, look DOWN for status.
</question>

<question name="epic_vs_feature">
**Q: What's the difference between an epic and a feature?**

**Epic:**
- Large body of work (1-4 weeks)
- Delivers distinct business value
- Can be released independently
- Contains 2-4 features
- Example: "Authentication Epic"

**Feature:**
- Coherent functionality unit
- Can be demonstrated independently
- Contains 2-5 user stories
- 8-20 story points total
- Example: "Login Feature" (within Auth Epic)
</question>

<question name="story_points">
**Q: How do story points work?**

Story points are relative effort estimates:
- **1 point:** Trivial (<1 hour)
- **2 points:** Small (1-2 hours)
- **3 points:** Medium (half-day)
- **5 points:** Larger (full day)
- **8 points:** Large (1-2 days) - split this

Key principles:
- Relative, not absolute time
- Compare to other stories
- Include complexity, not just time
- If >5, consider splitting
</question>

<question name="acceptance_criteria">
**Q: How do I write good acceptance criteria?**

Use Given/When/Then format:

```
### AC1: Valid login
- **Given** I'm on the login page with valid credentials
- **When** I click the login button
- **Then** I'm redirected to the dashboard
```

Good ACs are:
- Testable (can verify pass/fail)
- Independent (don't depend on each other)
- Focused (one behavior each)
- Complete (cover the story scope)

Target 2-4 ACs per story.
</question>

</common_questions>

<getting_started>
**New to this skill? Here's how to start:**

1. **Create a brief** - Define what you're building
   `/create-plan-agile` -> select "Start new project"

2. **Create roadmap with epics** - Define major value deliverables
   Select "Create agile roadmap"

3. **Define first epic** - Break into features
   Select "Define Epic 1"

4. **Define first feature** - Break into user stories
   Select "Define Feature F01"

5. **Create user stories** - Define As a/I want/So that + AC
   Select "Define user stories"

6. **Plan first story** - Create executable plan
   `/plan-us .planning/E01-name/F01-name/US-001-name/US-001.md`

7. **Execute story** - Build it
   `/run-us .planning/E01-name/F01-name/US-001-name/US-001-PLAN.md`

8. **Repeat** - Plan and execute remaining stories
</getting_started>

<process>
When user asks for guidance:

1. Understand their question
2. Find relevant section above
3. Provide clear, direct answer
4. Offer to help with next step

If question not covered: Use general agile/scrum knowledge while staying
consistent with this skill's philosophy (solo dev + Claude, no ceremonies).
</process>
