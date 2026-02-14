# Workflow: Create Epic

<required_reading>
**Read these files NOW:**
1. templates/epic.md
2. templates/feature.md
3. `.planning/ROADMAP.md`
</required_reading>

<purpose>
Define an epic with its features. This workflow creates the EPIC.md and
sets up the feature structure.
</purpose>

<process>

<step name="select_epic">
```bash
cat .planning/ROADMAP.md
ls .planning/E*/ 2>/dev/null
```

Identify which epic to define (usually the first undefined one).
</step>

<step name="gather_epic_context">
For this epic, understand:
- What business value does it deliver?
- What are the major features needed?
- What depends on this epic?
</step>

<step name="identify_features">
Break epic into 2-4 features.

Good features:
- Coherent unit of functionality
- Can be demonstrated independently
- Contains 2-5 user stories

Present:
"For Epic [X]: [Name], I'd suggest these features:

F01: [Feature name] - [what it does]
F02: [Feature name] - [what it does]
F03: [Feature name] - [what it does]

Does this breakdown work? (yes / adjust)"
</step>

<step name="create_structure">
Create feature directories:
```bash
mkdir -p .planning/EXX-name/F01-feature-name
mkdir -p .planning/EXX-name/F02-feature-name
```
</step>

<step name="write_epic">
Create `.planning/EXX-name/EPIC.md` using templates/epic.md.

Include:
- Epic ID and status
- Value proposition (As a / I want / So that)
- Success metrics
- Feature table
- Dependencies
- Out of scope
</step>

<step name="offer_next">
```
Epic created: .planning/EXX-name/EPIC.md
Features identified: F01, F02, F03

What's next?
1. Define Feature F01 with user stories
2. Define all features first, stories later
3. Done for now
```
</step>

</process>

<feature_guidelines>
**Good feature count:** 2-4 features per epic

**Feature naming:**
- F01-login - sequential within epic
- F02-registration - clear, descriptive
- Lowercase, hyphenated

**Feature size:**
- 2-5 user stories per feature
- 8-20 story points total
- Can be demonstrated independently
</feature_guidelines>

<success_criteria>
Epic is complete when:
- [ ] `.planning/EXX-name/EPIC.md` exists
- [ ] 2-4 features identified
- [ ] Feature directories created
- [ ] User knows how to proceed to features
</success_criteria>
