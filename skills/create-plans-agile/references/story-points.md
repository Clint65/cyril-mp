# Story Points Reference

This reference defines how to estimate and use story points in the create-plans-agile skill.

## What Are Story Points?

Story points are **relative effort estimates** for user stories. They measure complexity, uncertainty, and work required - not hours.

**Key principle:** Compare stories to each other, not to calendar time.

## The Fibonacci Scale

We use modified Fibonacci: 1, 2, 3, 5, 8

### 1 Point - Trivial
- Very well understood
- Minimal code changes
- < 1 hour of work
- No surprises expected

**Examples:**
- Fix a typo in UI text
- Add a simple CSS style
- Update a configuration value
- Add a log statement

### 2 Points - Small
- Well understood
- Straightforward implementation
- 1-2 hours of work
- Low risk

**Examples:**
- Add a new form field
- Create a simple component
- Add basic validation
- Implement a simple API endpoint

### 3 Points - Medium
- Understood but some complexity
- May require design decisions
- Half-day of work
- Some unknowns possible

**Examples:**
- Build a form with validation
- Create a list with filtering
- Implement authentication logic
- Add a new database table with API

### 5 Points - Larger
- Complex functionality
- Multiple files/components
- Full day of work
- Some uncertainty

**Examples:**
- Build a complete feature with UI
- Implement payment integration
- Create a dashboard with charts
- Add real-time functionality

### 8 Points - Large (Split This!)
- Too large for one story
- High complexity/uncertainty
- 1-2 days of work
- **Should be split into smaller stories**

**If you estimate 8 points:** Stop and split the story.

## Estimation Guidelines

### Relative Comparison

Compare stories to each other:

"Is this story more or less effort than that 3-point story we did?"

**Reference stories:**
- Pick a well-understood 2-point story as baseline
- Compare new stories to it
- "This feels about twice as complex" → 5 points

### What to Include

Story points should account for:

**Complexity:**
- How complicated is the logic?
- How many edge cases?
- How many integrations?

**Uncertainty:**
- How well do we understand this?
- Any unknowns or risks?
- New technology involved?

**Work Volume:**
- How many files touched?
- How much code written?
- How much testing needed?

### What NOT to Include

- Time pressure ("need it fast" doesn't change points)
- Who's doing it (points are consistent)
- Quality shortcuts (always assume proper quality)

## When to Split Stories

### Always Split If:

**Points > 5:**
- Any 8-point story must be split
- Even 5-point stories benefit from splitting

**Too Many ACs:**
- More than 5 acceptance criteria
- Split by grouping related ACs

**Multiple Domains:**
- Frontend AND backend AND database
- Consider splitting by layer

**High Uncertainty:**
- "We need to figure out X first"
- Split into research + implementation

### How to Split

**By Acceptance Criteria:**
```
Original: "User can manage settings" (8 points, 6 ACs)
Split:
- "User can view settings" (3 points, 2 ACs)
- "User can update settings" (3 points, 2 ACs)
- "User can reset settings" (2 points, 2 ACs)
```

**By User Journey:**
```
Original: "User completes checkout" (8 points)
Split:
- "User adds items to cart" (3 points)
- "User enters shipping info" (2 points)
- "User completes payment" (3 points)
```

**By Technical Layer:**
```
Original: "User profile feature" (8 points)
Split:
- "Profile API endpoints" (3 points)
- "Profile UI components" (3 points)
- "Profile validation/tests" (2 points)
```

## Quality vs Points

### The Connection

Points and quality are related through context management:

**Lower points (1-3):**
- Complete in peak quality zone
- Full attention to detail
- Comprehensive testing
- Clean code guaranteed

**Higher points (5):**
- Pushing toward context limits
- Quality may start to degrade
- Consider splitting anyway

**8+ points:**
- Will definitely hit degradation
- Quality WILL suffer
- Must split

### The Rule

**Better to have 3 two-point stories than 1 six-point story.**

Same total work, but:
- Consistent quality throughout
- Clear progress markers
- Easier to verify
- Surgical git commits

## Tracking Points

### In FEATURE.md

```markdown
## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-001 | Basic login | 3 | Complete | 2025-01-15 |
| US-002 | Remember me | 2 | In progress | - |
| US-003 | Password reset | 3 | Not started | - |

**Total:** 3 stories, 8 points (3/8 complete)
```

### In ROADMAP.md

```markdown
## Progress

| Epic | Features | Stories | Points | Status |
|------|----------|---------|--------|--------|
| E01: Auth | 2 | 3/5 | 8/15 | In progress |
```

## Common Mistakes

### Mistake: Time-Based Estimates
**Wrong:** "This will take 4 hours, so 4 points"
**Right:** "This is twice as complex as that 2-pointer, so 5 points"

### Mistake: Adjusting for Skill
**Wrong:** "I'm fast at this, so fewer points"
**Right:** Points are consistent regardless of who works on it

### Mistake: Padding Estimates
**Wrong:** "Add buffer points for safety"
**Right:** Estimate actual complexity, split if uncertain

### Mistake: Ignoring Uncertainty
**Wrong:** "It's simple, 2 points" (but never done before)
**Right:** "Uncertain, adding a point" or split into spike + implementation

## Summary

| Points | Complexity | Time | Action |
|--------|------------|------|--------|
| 1 | Trivial | <1hr | Just do it |
| 2 | Small | 1-2hr | Standard story |
| 3 | Medium | half-day | Standard story |
| 5 | Larger | full day | Consider splitting |
| 8 | Large | 1-2 days | **Must split** |

**The golden rule:** When in doubt, estimate higher and split.
