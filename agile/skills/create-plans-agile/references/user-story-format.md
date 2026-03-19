# User Story Format Reference

This reference defines how to write well-formed user stories with acceptance criteria.

## Story Format

### As a / I want / So that

```
As a [specific user type]
I want [specific capability]
So that [specific benefit]
```

**Each part matters:**
- **As a** - WHO benefits (be specific, not just "user")
- **I want** - WHAT capability (concrete action)
- **So that** - WHY it matters (user value)

### Good Examples

```
As a returning user
I want to stay logged in across browser sessions
So that I don't have to re-enter my credentials every visit

As a content creator
I want to save drafts automatically
So that I don't lose work if my browser crashes

As an admin user
I want to export user data to CSV
So that I can analyze usage patterns in Excel
```

### Bad Examples

```
As a user                          <- Too vague
I want better login               <- Not specific
So that it's easier               <- No clear value

As a user                          <- Who?
I want authentication             <- What does that mean?
So that it works                  <- Why?
```

## Acceptance Criteria Format

### Given / When / Then

```
### AC1: [Criterion name]
- **Given** [initial context/precondition]
- **When** [action/trigger]
- **Then** [expected outcome]
```

**Each part:**
- **Given** - Starting state, preconditions
- **When** - Action that triggers behavior
- **Then** - Observable, verifiable outcome

### Good Examples

```
### AC1: Valid login redirects to dashboard
- **Given** I am on the login page with valid credentials entered
- **When** I click the "Login" button
- **Then** I am redirected to the dashboard page

### AC2: Invalid password shows error
- **Given** I am on the login page with incorrect password
- **When** I click the "Login" button
- **Then** I see an error message "Invalid email or password"
- **And** I remain on the login page

### AC3: Empty form shows validation
- **Given** I am on the login page with empty fields
- **When** I click the "Login" button
- **Then** I see validation messages on required fields
```

### Bad Examples

```
- Login should work                <- Not testable
- User sees error                  <- When? What error?
- Form validates                   <- What validation?
```

## INVEST Criteria

Before finalizing a story, verify it meets INVEST:

### Independent
- Can be developed without other stories completing first
- No blocking dependencies (or they're minimal and explicit)
- Can be tested in isolation

**Check:** "Can we build this story without finishing another story first?"

### Negotiable
- Details can be refined during planning
- Not a rigid specification
- Room for implementation decisions

**Check:** "Is this flexible enough to adjust during implementation?"

### Valuable
- Delivers value to the user (not just technical work)
- User can see/experience the benefit
- Worth doing on its own

**Check:** "Would a user care if we shipped just this?"

### Estimable
- Clear enough to estimate effort
- Team understands scope
- No major unknowns

**Check:** "Can we confidently assign story points?"

### Small
- Fits in one work session
- 1-5 story points
- Few enough tasks to maintain quality

**Check:** "Can this complete without context degradation?"

### Testable
- Clear pass/fail criteria
- Acceptance criteria are verifiable
- Can write tests for it

**Check:** "Do we know when this is done?"

## Acceptance Criteria Guidelines

### How Many?
- **Target:** 2-4 acceptance criteria per story
- **Minimum:** 2 (happy path + at least one edge case)
- **Maximum:** 5 (if more, consider splitting story)

### Coverage
Each story should have ACs covering:
1. **Happy path** - Main success scenario
2. **Edge cases** - Important variations
3. **Error cases** - What happens when things go wrong
4. **Boundaries** - Limits and constraints

### Characteristics of Good ACs

**Testable:**
- Can determine pass/fail
- Observable outcome
- No ambiguity

**Independent:**
- Each AC stands alone
- Don't depend on other ACs
- Can verify separately

**Focused:**
- One behavior per AC
- Single Given/When/Then flow
- Not compound conditions

**Complete:**
- Cover the story's scope
- No gaps in functionality
- All important scenarios

## Common Patterns

### CRUD Operations

```
### AC1: Create item
- **Given** I am on the items page
- **When** I click "New Item" and fill in details and save
- **Then** the item appears in the list

### AC2: Read item details
- **Given** I have an existing item
- **When** I click on the item
- **Then** I see its full details

### AC3: Update item
- **Given** I am viewing an item's details
- **When** I edit a field and save
- **Then** the change is persisted

### AC4: Delete item
- **Given** I am viewing an item
- **When** I click "Delete" and confirm
- **Then** the item is removed from the list
```

### Form Validation

```
### AC1: Valid submission
- **Given** I have filled all required fields correctly
- **When** I submit the form
- **Then** the form is accepted

### AC2: Required field validation
- **Given** I have left a required field empty
- **When** I submit the form
- **Then** I see "This field is required" on the empty field

### AC3: Format validation
- **Given** I have entered an invalid email format
- **When** I submit the form
- **Then** I see "Please enter a valid email"
```

### Authentication

```
### AC1: Successful login
- **Given** I enter valid credentials
- **When** I click login
- **Then** I am authenticated and redirected to dashboard

### AC2: Invalid credentials
- **Given** I enter invalid credentials
- **When** I click login
- **Then** I see error and remain on login page

### AC3: Locked account
- **Given** I have failed login 5 times
- **When** I try again
- **Then** I see "Account locked" message
```

## Splitting Stories

### When to Split
- More than 5 story points
- More than 5 acceptance criteria
- Multiple user types involved
- Multiple technical domains

### How to Split

**By Acceptance Criteria:**
- Group related ACs into separate stories
- Each story remains independently valuable

**By User Journey:**
- Setup story, action story, completion story
- Each delivers partial flow

**By Interface:**
- API story, UI story
- Each layer separately testable

**By Data Operation:**
- Create story, update story, delete story
- Each CRUD operation separate
