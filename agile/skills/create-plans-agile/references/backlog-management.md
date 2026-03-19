# Backlog Management Reference

This reference defines how to organize and prioritize work in the create-plans-agile skill.

## The Backlog Hierarchy

```
ROADMAP.md (Epic backlog)
    |
    └── EPIC.md (Feature backlog)
            |
            └── FEATURE.md (Story backlog)
```

Each level has its own "backlog" - ordered list of items to complete.

## Priority System (MoSCoW)

### Must Have
- Critical for the release
- Cannot ship without these
- Core functionality
- No workarounds exist

**In YAML:** `priority: must`

### Should Have
- Important but not critical
- Can ship without (with pain)
- Expected by users
- Workarounds possible

**In YAML:** `priority: should`

### Could Have
- Desirable if time permits
- Nice-to-have features
- Polish items
- User delights

**In YAML:** `priority: could`

### Won't Have (This Time)
- Explicitly out of scope
- Deferred to future
- Documented but not planned
- Prevents scope creep

**In YAML:** `priority: wont`

## Story Ordering

### Within a Feature

Order stories for optimal flow:

**1. Foundation First**
- Setup, configuration, base components
- Stories other stories depend on
- Infrastructure before features

**2. Happy Path Next**
- Core functionality
- Main user journey
- Primary use case

**3. Variations**
- Edge cases
- Alternative flows
- Error handling

**4. Polish Last**
- Loading states
- Accessibility
- Performance
- UX refinements

### Example Ordering

```markdown
## User Stories (Login Feature)

| ID | Story | Points | Priority | Dependencies |
|----|-------|--------|----------|--------------|
| US-001 | Basic login form | 3 | must | none |
| US-002 | Form validation | 2 | must | US-001 |
| US-003 | Error messages | 2 | must | US-002 |
| US-004 | Remember me | 2 | should | US-001 |
| US-005 | Loading states | 1 | could | US-003 |
```

**Order rationale:**
1. US-001 first - creates foundation
2. US-002 next - adds validation
3. US-003 - handles errors
4. US-004 - enhancement after basics work
5. US-005 - polish after core complete

## Dependency Management

### Tracking Dependencies

In FEATURE.md:

```markdown
## Dependencies

**Story Dependencies:**
- US-002 requires US-001 (form must exist to validate)
- US-003 requires US-002 (validation before error display)

**External Dependencies:**
- Auth API endpoint must be deployed
- Design tokens must be finalized
```

### Handling Blockers

When a story is blocked:

1. **Mark it clearly** - Note in status
2. **Skip and continue** - Work on unblocked stories
3. **Track the blocker** - Document what's needed
4. **Return when clear** - Pick it up when unblocked

```markdown
| ID | Story | Points | Status | Notes |
|----|-------|--------|--------|-------|
| US-004 | OAuth login | 3 | Blocked | Waiting on Google API keys |
```

## Refinement

### When to Refine

Refine stories when:
- About to start planning a feature
- Story is next in queue
- Requirements have changed
- Uncertainty needs resolution

### What to Refine

**Add detail to:**
- Acceptance criteria (Given/When/Then)
- Technical notes
- Dependencies
- Story points (re-estimate if needed)

**Remove:**
- Ambiguity
- Assumptions
- Uncertainty
- Scope creep

### Refinement Checklist

Before planning a story:
- [ ] Acceptance criteria are clear
- [ ] Story points estimated
- [ ] Dependencies identified
- [ ] Technical approach understood
- [ ] INVEST criteria met

## Managing Scope

### Preventing Scope Creep

**In BRIEF.md:**
- "Out of Scope" section
- Clear boundaries

**In EPIC.md:**
- "Out of Scope" section
- What this epic doesn't include

**In User Stories:**
- Focused acceptance criteria
- Clear "done" definition

### When Scope Changes

**If new requirement discovered during story:**
1. Is it critical for this story? → Add to current story
2. Is it enhancement? → Log to ISSUES.md
3. Is it new story? → Add to feature backlog

**The deviation rules handle this:**
- Rules 1-3: Fix immediately (bugs, blockers, critical)
- Rule 4: Ask (architectural changes)
- Rule 5: Defer (enhancements)

## Progress Tracking

### Story Level

In FEATURE.md:
```markdown
| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-001 | Basic login | 3 | Complete | 2025-01-15 |
| US-002 | Validation | 2 | In progress | - |
| US-003 | Errors | 2 | Not started | - |

**Progress:** 1/3 stories (3/7 points)
```

### Feature Level

In EPIC.md:
```markdown
| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F01 | Login | 1/3 | 3/7 | In progress |
| F02 | Register | 0/2 | 0/5 | Not started |

**Progress:** 1/5 stories (3/12 points)
```

### Epic Level

In ROADMAP.md:
```markdown
| Epic | Features | Stories | Points | Status |
|------|----------|---------|--------|--------|
| E01: Auth | 1/2 | 1/5 | 3/12 | In progress |
| E02: Dashboard | 0/3 | 0/8 | 0/20 | Not started |

**Progress:** 1/13 stories (3/32 points)
```

## ISSUES.md - The Enhancement Log

### Purpose

Capture ideas and enhancements without derailing current work.

### When to Log

- Enhancement discovered during story execution (deviation rule 5)
- User feedback received
- Technical debt identified
- Future feature ideas

### Format

```markdown
# Issues & Enhancements

## Enhancements

### [ENH-001] Add keyboard shortcuts
**Source:** Discovered during US-003
**Priority:** could
**Notes:** Would improve power user experience

### [ENH-002] Dark mode support
**Source:** User feedback
**Priority:** should
**Notes:** Multiple users requested

## Technical Debt

### [TECH-001] Refactor auth module
**Source:** Code review during US-002
**Priority:** should
**Notes:** Current structure won't scale

## Bugs (Non-Critical)

### [BUG-001] Layout shift on slow network
**Source:** Testing US-004
**Priority:** could
**Notes:** Minor visual issue, not blocking
```

### Graduating Issues to Stories

When ready to address:
1. Create user story from issue
2. Add to appropriate feature
3. Mark issue as "converted to US-XXX"

## Summary

**Ordering principles:**
- Must before Should before Could
- Dependencies before dependents
- Foundation before features
- Happy path before edge cases

**Tracking principles:**
- Progress flows UP (story → feature → epic → roadmap)
- Context flows DOWN (roadmap → epic → feature → story)
- Update tracking on every completion
- Log enhancements, don't derail
