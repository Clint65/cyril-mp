# User Gates Reference

User gates prevent Claude from charging ahead at critical decision points.

## Question Types

### AskUserQuestion Tool
Use for **structured choices** (2-4 options):
- Selecting from distinct approaches
- Domain/type selection
- When user needs to see options to decide

Examples:
- "What type of project?" (macos-app / iphone-app / web-app / other)
- "Research confidence is low. How to proceed?" (dig deeper / proceed anyway / pause)
- "Multiple valid approaches exist:" (Option A / Option B / Option C)

### Inline Questions
Use for **simple confirmations**:
- Yes/no decisions
- "Does this look right?"
- "Ready to proceed?"

Examples:
- "Here's the task breakdown: [list]. Does this look right?"
- "Proceed with this approach?"
- "I'll initialize a git repo. OK?"

## Decision Gate Loop

After gathering context, ALWAYS offer:

```
Ready to [action], or would you like me to ask more questions?

1. Proceed - I have enough context
2. Ask more questions - There are details to clarify
3. Let me add context - I want to provide additional information
```

Loop continues until user selects "Proceed".

## Mandatory Gate Points

| Location | Gate Type | Trigger |
|----------|-----------|---------|
| plan-user-story | Inline | Confirm task breakdown |
| plan-user-story | AskUserQuestion | Multiple valid approaches |
| plan-user-story | AskUserQuestion | Decision gate before writing |
| create-user-story | AskUserQuestion | Story point estimation confirmation |
| execute-user-story | Inline | Verification failure |
| execute-user-story | Inline | Acceptance criteria review before proceeding |
| execute-user-story | AskUserQuestion | Previous story had issues |
| create-epic | Inline | Confirm feature breakdown |
| create-roadmap-agile | Inline | Confirm epic structure |
| create-roadmap-agile | AskUserQuestion | Decision gate before writing |
| handoff | Inline | Handoff acknowledgment |

## Good vs Bad Gating

### Good
- Gate before writing artifacts (not after)
- Gate when genuinely ambiguous
- Gate when issues affect next steps
- Quick inline for simple confirmations

### Bad
- Asking obvious choices ("Should I save the file?")
- Multiple gates for same decision
- AskUserQuestion for yes/no
- Gates after the fact
