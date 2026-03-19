---
name: commit-us
description: Commit a completed user story with proper Angular Conventional Commits git message format. Use when the user wants to commit a story, save story progress to git, or create a structured commit for a user story. Triggers on "commit story", "save story", or after testing a story.
argument-hint: <story-path>
disable-model-invocation: true
allowed-tools:
  - Read
  - Bash
  - Glob
---

Commit the user story at $ARGUMENTS with a properly formatted git message.

<objective>
Create a well-structured git commit for a completed user story.
Ensures story changes are committed with proper format and traceability.
</objective>

<process>

**1. Locate story files:**

Determine if path is story, plan, or summary:
```bash
# Check what was provided and find related files
ls -la $(dirname $ARGUMENTS)/
```

Find the summary file (contains what was actually implemented):
```bash
# If summary provided directly
cat $ARGUMENTS

# If story or plan provided, find summary
ls $(dirname $ARGUMENTS)/US-*-SUMMARY.md 2>/dev/null
```

If no summary exists:
```
No summary found for this story. Cannot commit without a completed story.
Run /run-us to implement the story first, then retry /commit-us.
```
Stop execution and report the error.

**2. Parse story information:**

From the summary, extract:
- Story ID (US-XXX)
- Story title
- User story format (As a/I want/So that)
- Acceptance criteria and their status
- Files created/modified
- Deviations (if any)

**Determine commit type** (Angular Conventional Commits for semantic-release):

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability (triggers **minor** release) |
| `fix` | Bug fix (triggers **patch** release) |
| `refactor` | Code restructuring, no behavior change (no release) |
| `perf` | Performance improvement (triggers **patch** release) |
| `test` | Adding or updating tests only (no release) |
| `docs` | Documentation only (no release) |
| `style` | Formatting, whitespace, no code change (no release) |
| `build` | Build system or dependencies (no release) |
| `ci` | CI configuration (no release) |
| `chore` | Maintenance, no production code change (no release) |

**Determine scope:** Use the most specific module or area affected (e.g., `auth`, `api`, `ui`, `db`). If the story spans multiple areas, use the primary one. Include the story ID in the body, not the scope.

If the story introduces a breaking change, add `!` after the scope: `feat(auth)!: remove legacy login`

**3. Check git status:**
```bash
git status --porcelain
```

If no changes staged or unstaged:
```
No changes to commit for this story.
The story may have already been committed.

Check: git log --oneline -5
```

**4. Stage relevant files:**
```bash
# Stage planning files
git add .planning/

# Stage source files (based on summary's "Files Created/Modified" section)
# Parse files from summary and stage them
```

Present staged changes:
```
Files to be committed:

Planning:
- .planning/epics/EXX/features/FXX/US-XXX-SUMMARY.md
- .planning/epics/EXX/features/FXX/US-XXX-PLAN.md (if exists)
- .planning/epics/EXX/features/FXX/FEATURE.md (updated)

Source:
- [files from summary]
```

Proceed to create the commit.

**5. Create commit:**

Format: `<type>(<scope>): <subject>` (Angular Conventional Commits)

```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <short imperative description, max 72 chars>

US-XXX: [Story title]
As a [user type], I want [capability] so that [benefit]

Acceptance Criteria:
- [x] AC1: [brief description]
- [x] AC2: [brief description]
- [x] AC3: [brief description]

[BREAKING CHANGE: <description> — only if applicable]

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

Rules:
- Subject line: imperative mood, no period, max 72 chars
- Body: blank line after subject, wrap at 100 chars
- `BREAKING CHANGE:` footer triggers **major** release — use only when API/behavior changes break consumers
- Scope is the affected module (e.g., `auth`, `api`), NOT the story ID

**6. Confirm commit:**
```bash
git log -1 --oneline
```

**7. Report success:**
```
════════════════════════════════════════
STORY COMMITTED: US-XXX
════════════════════════════════════════

Commit: [hash]
Story: [title]

Files committed: [N]
Branch: [current branch]

Push with: git push
════════════════════════════════════════
```

</process>

<success_criteria>
- Story summary located and parsed
- Commit type correctly determined (feat/fix/refactor/etc.)
- Scope reflects affected module, not story ID
- Subject line: imperative, max 72 chars, no period
- Body includes story reference and acceptance criteria
- BREAKING CHANGE footer only when warranted
- Commit message passes semantic-release Angular preset validation
- Commit hash reported
</success_criteria>
