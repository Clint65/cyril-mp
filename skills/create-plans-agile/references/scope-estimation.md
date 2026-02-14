# Scope Estimation & Quality-Driven Story Splitting

User stories must maintain consistent quality from first task to last. This requires understanding the **quality degradation curve** and splitting aggressively to stay in the peak quality zone.

## The Quality Degradation Curve

**Critical insight:** Claude doesn't degrade at arbitrary percentages - it degrades when it *perceives* context pressure and enters "completion mode."

```
Context Usage  |  Quality Level   |  Claude's Mental State
---------------------------------------------------------
0-30%          |  PEAK            |  "I can be thorough and comprehensive"
               |                  |  No anxiety, full detail, best work

30-50%         |  GOOD            |  "Still have room, maintaining quality"
               |                  |  Engaged, confident, solid work

50-70%         |  DEGRADING       |  "Getting tight, need to be efficient"
               |                  |  Efficiency mode, compression begins

70%+           |  POOR            |  "Running out, must finish quickly"
               |                  |  Self-lobotomization, rushed, minimal
```

**The 40-50% inflection point:**

This is where quality breaks. Claude sees context mounting and thinks "I'd better conserve now or I won't finish." Result: The classic mid-execution statement "I'll complete the remaining tasks more concisely" = quality crash.

**The fundamental rule:** Stop BEFORE quality degrades, not at context limit.

## Target: 50% Context Maximum

**User story plans should complete within ~50% of context usage.**

Why 50% not 80%?
- Huge safety buffer
- No context anxiety possible
- Quality maintained from start to finish
- Room for unexpected complexity
- Space for iteration and fixes

**If you target 80%, you're planning for failure.** By the time you hit 80%, you've already spent 40% in degradation mode.

## The 2-3 Task Rule for User Stories

**Each US-PLAN.md should contain 2-3 tasks maximum.**

Why this number?

**Task 1 (0-15% context):**
- Fresh context
- Peak quality
- Comprehensive implementation
- Full testing
- Complete documentation

**Task 2 (15-35% context):**
- Still in peak zone
- Quality maintained
- Buffer feels safe
- No anxiety

**Task 3 (35-50% context):**
- Beginning to feel pressure
- Quality still good but managing it
- Natural stopping point
- Better to commit here

**Task 4+ (50%+ context):**
- DEGRADATION ZONE
- "I'll do this concisely" appears
- Quality crashes
- Should have split before this

## When to Split User Stories

### Always Split If:

**1. More than 3 acceptance criteria**
- Each AC typically maps to 1+ tasks
- 4+ ACs = 4+ tasks = degradation risk
- Split into multiple stories

**2. Story estimated at >5 points**
- 8-point stories are too large
- Split by acceptance criteria grouping
- Create US-001a, US-001b if needed

**3. Multiple technical domains**
```
Bad (1 story):
- Backend API changes
- Database schema updates
- Frontend components
- Integration tests
Total: 4 domains, 1 story -> guaranteed degradation

Good (3 stories):
- US-001: Backend + Database (2 tasks)
- US-002: Frontend components (2 tasks)
- US-003: Integration + tests (2 tasks)
Total: 3 stories, consistent quality
```

**4. Any task with >5 file modifications**
- Large tasks burn context fast
- Split by file groups or logical units

### Consider Splitting If:

**1. Estimated >5 files modified total**
- Context from reading existing code
- Context from diffs
- Adds up faster than expected

**2. Complex domains (auth, payments, data modeling)**
- These require careful thinking
- Burns more context per task
- Split more aggressively

**3. Any uncertainty about approach**
- "Figure out X" phase separate from "implement X" phase
- Don't mix exploration and implementation

## Story Point Guidelines

**1 point:** Trivial change, <1 hour, well-understood
**2 points:** Small change, 1-2 hours, straightforward
**3 points:** Medium change, half-day, some complexity
**5 points:** Larger change, full day, multiple files/components
**8 points:** Large change, 1-2 days - **MUST SPLIT**

If estimated >5 points, strongly consider splitting.
If estimated >8 points, always split.

## Mapping Acceptance Criteria to Tasks

**Good pattern:**
```
AC1: User can enter email
  -> Task 1: Create email input component

AC2: User can enter password
AC3: Password shows/hides on toggle
  -> Task 2: Create password input with toggle

AC4: Login button submits form
  -> Task 3: Implement form submission
```

3 tasks, maps to 4 ACs, within budget.

**Bad pattern:**
```
AC1-AC4: All in one task
  -> Task 1: Build entire login form

Result: One massive task that degrades quality
```

## Autonomous vs Interactive Story Plans

**Critical optimization:** Plans without checkpoints don't need main context.

### Autonomous Plans (No Checkpoints)
- Contains only `type="auto"` tasks
- No user interaction needed
- **Execute via subagent with fresh 200k context**
- Impossible to degrade (always starts at 0%)
- Creates US-SUMMARY, commits, reports back

### Interactive Plans (Has Checkpoints)
- Contains `checkpoint:human-verify` or `checkpoint:decision` tasks
- Requires user interaction
- Must execute in main context
- Still target 50% context (2-3 tasks)

## Summary

**The principle:** Aggressive atomicity. More stories, smaller scope, consistent quality.

**Story size rules:**
- 2-3 tasks per US-PLAN
- 2-5 acceptance criteria per story
- 1-5 story points (split if >5)
- Target 50% context usage

**The rule:** If in doubt, split. Quality over consolidation. Always.
