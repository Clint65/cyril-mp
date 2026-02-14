# Workflow: Create Feature

<required_reading>
**Read these files NOW:**
1. templates/feature.md
2. templates/user-story.md
3. Current epic's EPIC.md
</required_reading>

<purpose>
Define a feature with its user stories. This workflow creates the FEATURE.md
and optionally the initial user stories.
</purpose>

<process>

<step name="select_feature">
```bash
cat .planning/EXX-name/EPIC.md
ls .planning/EXX-name/F*/ 2>/dev/null
```

Identify which feature to define.
</step>

<step name="gather_feature_context">
For this feature, understand:
- What functionality does it provide?
- Who uses it?
- What's the happy path?
- What are the edge cases?
</step>

<step name="identify_stories">
Break feature into 2-5 user stories.

Good stories:
- Independent (can be built alone)
- Valuable (delivers user value)
- Small (1-5 story points)
- Testable (clear acceptance criteria)

Present:
"For Feature [X]: [Name], I'd suggest these user stories:

US-001: [Story title] (~X points)
  As a [user], I want [capability] so that [benefit]

US-002: [Story title] (~X points)
  As a [user], I want [capability] so that [benefit]

US-003: [Story title] (~X points)
  As a [user], I want [capability] so that [benefit]

Does this breakdown work? (yes / adjust / add more detail)"
</step>

<step name="write_feature">
Create `.planning/EXX-name/FXX-name/FEATURE.md` using templates/feature.md.

Include:
- Feature ID and status
- Description (2-3 sentences)
- User story table
- Feature-level acceptance criteria
- Technical notes
- Dependencies
</step>

<step name="write_stories_option">
Ask: "Create the user story files now? (yes / no - I'll define them later)"

If yes: For each story, create `US-XXX-name.md` using templates/user-story.md
- Include As a/I want/So that
- Add acceptance criteria (Given/When/Then)
- Estimate story points
- Add suggested tasks
</step>

<step name="offer_next">
```
Feature created: .planning/EXX-name/FXX-name/FEATURE.md
Stories identified: [X] stories, [XX] total points

What's next?
1. Define user stories in detail
2. Plan first story (US-001)
3. Move to next feature
4. Done for now
```
</step>

</process>

<story_ordering>
Order stories for optimal flow:

1. **Foundation stories first** - Setup, configuration, base components
2. **Happy path next** - Core functionality
3. **Variations** - Edge cases, alternatives
4. **Polish last** - Error handling, loading states, accessibility
</story_ordering>

<success_criteria>
Feature is complete when:
- [ ] `.planning/EXX-name/FXX-name/FEATURE.md` exists
- [ ] 2-5 user stories identified
- [ ] Story table has titles and point estimates
- [ ] Feature-level acceptance criteria defined
- [ ] User knows how to proceed to stories
</success_criteria>
