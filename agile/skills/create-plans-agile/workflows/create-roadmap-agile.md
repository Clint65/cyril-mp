# Workflow: Create Agile Roadmap

<required_reading>
**Read these files NOW:**
1. templates/roadmap-agile.md
2. templates/epic.md
3. Read `.planning/BRIEF.md` if it exists
</required_reading>

<purpose>
Define the epic structure for the project. Epics are large bodies of work
that deliver business value. Each epic contains features, and features
contain user stories.
</purpose>

<process>

<step name="check_brief">
```bash
cat .planning/BRIEF.md 2>/dev/null || echo "No brief found"
```

If no brief: Ask to create one first or gather quick context.
</step>

<step name="identify_epics">
Based on brief, identify 2-4 epics.

Good epics:
- Deliver distinct business value
- Can be released independently
- Take 1-4 weeks to complete (for solo + Claude)

Example patterns:
- Authentication -> Core Features -> Polish -> Launch
- Infrastructure -> MVP -> Enhancement -> Scale
- Foundation -> Primary Value -> Secondary Value -> Quality
</step>

<step name="confirm_epics">
Present inline:

"Here's how I'd structure the epics:

E01: [Name] - [Value proposition]
E02: [Name] - [Value proposition]
E03: [Name] - [Value proposition]

Each epic will contain features, and features will contain user stories.

Does this feel right? (yes / adjust)"
</step>

<step name="create_structure">
For each epic:
```bash
mkdir -p .planning/E0X-name
```
</step>

<step name="write_roadmap">
Write to `.planning/ROADMAP.md` using templates/roadmap-agile.md format.

Include:
- Overview paragraph
- Epic list with value propositions
- Epic details (value, dependencies, feature count)
- Progress tracking table
</step>

<step name="git_commit">
```bash
git add .planning/
git commit -m "$(cat <<'EOF'
docs: initialize [project] with agile structure ([N] epics)

[One-liner from BRIEF.md]

Epics:
- E01: [name] - [value]
- E02: [name] - [value]
EOF
)"
```

Confirm: "Committed: docs: initialize [project] with agile structure"
</step>

<step name="offer_next">
```
Project initialized with Agile structure:
- Brief: .planning/BRIEF.md
- Roadmap: .planning/ROADMAP.md (X epics)
- Epic directories created
- Committed

What's next?
1. Define Epic 1 in detail (features + stories)
2. Review/adjust epics
3. Done for now
```
</step>

</process>

<anti_patterns>
- Don't create too many epics (2-4 for MVP)
- Don't add sprint timelines
- Don't include team ceremonies
- Don't add velocity tracking
- Don't create burndown charts

Keep it simple: Epics -> Features -> Stories
</anti_patterns>

<success_criteria>
Roadmap is complete when:
- [ ] `.planning/ROADMAP.md` exists
- [ ] 2-4 epics defined
- [ ] Epic directories created
- [ ] Brief + Roadmap committed together
- [ ] User knows how to proceed
</success_criteria>
