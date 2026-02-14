# Agile Roadmap Template

Copy and fill for `.planning/ROADMAP.md`:

```markdown
# Roadmap: [Project Name]

## Overview

[One paragraph describing the product vision and journey]

## Epics

- [ ] **E01: [Epic Name]** - [One-line value proposition]
- [ ] **E02: [Epic Name]** - [One-line value proposition]
- [ ] **E03: [Epic Name]** - [One-line value proposition]

## Epic Details

### E01: [Epic Name]

**Value:** [Business/user value this epic delivers]
**Depends on:** Nothing (first epic) | E0X
**Features:** [Number of features]
**Stories:** [Total story count or "TBD"]

Features:
- [ ] F01: [Feature name] ([X] stories, [XX] pts)
- [ ] F02: [Feature name] ([X] stories, [XX] pts)

### E02: [Epic Name]

**Value:** [Business/user value]
**Depends on:** E01
**Features:** [Number]
**Stories:** [Total or "TBD"]

Features:
- [ ] F01: [Feature name] ([X] stories, [XX] pts)

## Progress

| Epic | Features | Stories | Points | Status | Completed |
|------|----------|---------|--------|--------|-----------|
| E01: [Name] | 2 | 0/5 | 0/15 | Not started | - |
| E02: [Name] | 1 | 0/3 | 0/8 | Not started | - |

**Total:** [X] epics, [Y] features, [Z] stories, [W] points
```

<guidelines>
- 2-4 epics for MVP
- Each epic delivers distinct business value
- Stories estimated in points (1, 2, 3, 5, 8)
- Progress tracked at story level
- Update progress table as stories complete
</guidelines>

<epic_sizing>
**Good epic size:**
- 2-4 features
- 5-15 user stories total
- 1-4 weeks of work (solo + Claude)
- Delivers demonstrable value

**If epic too large:** Split by user journey or technical domain
**If epic too small:** Combine with related functionality
</epic_sizing>

<progress_tracking>
Update ROADMAP.md when:
- Story completed: Update story counts
- Feature completed: Check off feature, update counts
- Epic completed: Check off epic, mark completed date

```markdown
| Epic | Features | Stories | Points | Status | Completed |
|------|----------|---------|--------|--------|-----------|
| E01: Auth | 2 | 5/5 | 15/15 | Complete | 2025-01-15 |
| E02: Dashboard | 1 | 2/3 | 5/8 | In progress | - |
```
</progress_tracking>
