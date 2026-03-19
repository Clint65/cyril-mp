# Workflow: Create User Story

<required_reading>
**Read these files NOW:**
1. templates/user-story.md
2. Current feature's FEATURE.md
3. references/scope-estimation.md (for story points)
</required_reading>

<purpose>
Create a well-formed user story with acceptance criteria and estimated
story points. Stories should be small enough to complete in one session.
</purpose>

<process>

<step name="select_feature">
```bash
cat .planning/EXX-name/FXX-name/FEATURE.md
ls -d .planning/EXX-name/FXX-name/US-*/ 2>/dev/null
```

Identify which feature and next story number.
</step>

<step name="gather_story">
Ask (conversationally):
1. Who is this for? (user type/persona)
2. What do they want to do?
3. Why do they want it? (value)
4. How will we know it works? (acceptance criteria)

Build the story collaboratively.
</step>

<step name="define_acceptance_criteria">
For each criterion, use Given/When/Then format:

"Let's define the acceptance criteria:

AC1: [Criterion name]
- Given: [initial context]
- When: [action taken]
- Then: [expected outcome]

AC2: [Criterion name]
- Given: [initial context]
- When: [action taken]
- Then: [expected outcome]

Does this capture the requirements? (yes / add more / adjust)"
</step>

<step name="estimate_points">
Present point estimate inline:

"Based on complexity, I estimate this at X points.

Story point guide:
- 1 = trivial (<1hr)
- 2 = small (1-2hr)
- 3 = medium (half-day)
- 5 = larger (full day)
- 8 = large (1-2 days) - consider splitting

Does X points feel right? (yes / adjust to Y)"

**If >5 points:** Recommend splitting into multiple stories.
</step>

<step name="write_story">
Create story folder and file:
```bash
mkdir -p .planning/EXX-name/FXX-name/US-XXX-name
```

Create `US-XXX.md` in the folder using templates/user-story.md.

Include:
- YAML frontmatter (id, feature, epic, points, status, priority)
- Story (As a / I want / So that)
- Acceptance criteria (Given/When/Then)
- Suggested tasks
- Technical notes
- Definition of done checklist
</step>

<step name="update_feature">
Update FEATURE.md story table with new story:
- Add story to table
- Update total points
</step>

<step name="offer_next">
```
Story created: US-XXX
Points: X
Feature F0X now has Y stories (ZZ total points)

What's next?
1. Create another story for this feature
2. Plan US-XXX for execution (/plan-us)
3. Move to next feature
4. Done for now
```
</step>

</process>

<invest_check>
Before finalizing, verify story is INVEST:

- **I**ndependent: Can be developed without other stories
- **N**egotiable: Details can be refined
- **V**aluable: Delivers value to user
- **E**stimable: Can be sized (has clear scope)
- **S**mall: Fits in one session (1-5 points)
- **T**estable: Clear pass/fail criteria (ACs)

If story fails INVEST, adjust before writing.
</invest_check>

<ac_guidelines>
**Good acceptance criteria:**
- Testable (can verify pass/fail)
- Independent (don't depend on other ACs)
- Focused (one behavior each)
- Complete (cover the story scope)

**Target:** 2-4 acceptance criteria per story
**If >5 ACs:** Story may be too large, consider splitting
</ac_guidelines>

<success_criteria>
User story is complete when:
- [ ] `US-XXX-name/US-XXX.md` folder and file exist
- [ ] Has As a/I want/So that format
- [ ] Has 2-4 acceptance criteria (Given/When/Then)
- [ ] Story points estimated (1-5, or split if larger)
- [ ] Passes INVEST criteria
- [ ] FEATURE.md updated with story
</success_criteria>
