# CLI and API Automation Reference

**Core principle:** If it has a CLI or API, Claude does it. Never ask the human to perform manual steps that Claude can automate.

This reference documents what Claude CAN and SHOULD automate during plan execution.

## Deployment Platforms

### Vercel
**CLI:** `vercel`

**What Claude automates:**
- Create and deploy projects: `vercel --yes`
- Set environment variables: `vercel env add KEY production`
- Link to git repo: `vercel link`
- Trigger deployments: `vercel --prod`
- Get deployment URLs: `vercel ls`
- Manage domains: `vercel domains add example.com`

### Railway
**CLI:** `railway`

**What Claude automates:**
- Initialize project: `railway init`
- Link to repo: `railway link`
- Deploy: `railway up`
- Set variables: `railway variables set KEY=value`
- Get deployment URL: `railway domain`

### Fly.io
**CLI:** `fly`

**What Claude automates:**
- Launch app: `fly launch --no-deploy`
- Deploy: `fly deploy`
- Set secrets: `fly secrets set KEY=value`
- Scale: `fly scale count 2`

## Payment & Billing

### Stripe
**CLI:** `stripe`

**What Claude automates:**
- Create webhook endpoints: `stripe listen --forward-to localhost:3000/api/webhooks`
- Trigger test events: `stripe trigger payment_intent.succeeded`
- Create products/prices: Stripe API via curl/fetch
- Manage customers: Stripe API via curl/fetch
- Check webhook logs: `stripe webhooks list`

## Databases & Backend

### Supabase
**CLI:** `supabase`

**What Claude automates:**
- Initialize project: `supabase init`
- Link to remote: `supabase link --project-ref {ref}`
- Create migrations: `supabase migration new {name}`
- Push migrations: `supabase db push`
- Generate types: `supabase gen types typescript`
- Deploy functions: `supabase functions deploy {name}`

### Upstash (Redis/Kafka)
**CLI:** `upstash`

**What Claude automates:**
- Create Redis database: `upstash redis create {name} --region {region}`
- Get connection details: `upstash redis get {id}`
- Create Kafka cluster: `upstash kafka create {name} --region {region}`

### PlanetScale
**CLI:** `pscale`

**What Claude automates:**
- Create database: `pscale database create {name} --region {region}`
- Create branch: `pscale branch create {db} {branch}`
- Deploy request: `pscale deploy-request create {db} {branch}`
- Connection string: `pscale connect {db} {branch}`

## Version Control & CI/CD

### GitHub
**CLI:** `gh`

**What Claude automates:**
- Create repo: `gh repo create {name} --public/--private`
- Create issues: `gh issue create --title "{title}" --body "{body}"`
- Create PR: `gh pr create --title "{title}" --body "{body}"`
- Manage secrets: `gh secret set {KEY}`
- Trigger workflows: `gh workflow run {name}`
- Check status: `gh run list`

## Build Tools & Testing

### Node/npm/pnpm/bun
**What Claude automates:**
- Install dependencies: `npm install`, `pnpm install`, `bun install`
- Run builds: `npm run build`
- Run tests: `npm test`, `npm run test:e2e`
- Type checking: `tsc --noEmit`

### Xcode (macOS/iOS)
**CLI:** `xcodebuild`

**What Claude automates:**
- Build project: `xcodebuild -project App.xcodeproj -scheme App build`
- Run tests: `xcodebuild test -project App.xcodeproj -scheme App`
- Archive: `xcodebuild archive -project App.xcodeproj -scheme App`
- Check compilation: Parse xcodebuild output for errors

## Environment Configuration

### .env Files
**Tool:** Write tool

**What Claude automates:**
- Create .env files: Use Write tool
- Append variables: Use Edit tool
- Read current values: Use Read tool

## Authentication Gates

**Critical distinction:** When Claude tries to use a CLI/API and gets an authentication error, this is NOT a failure - it's a gate that requires human input to unblock automation.

**Pattern: Claude encounters auth error -> creates checkpoint -> you authenticate -> Claude continues**

### Authentication Gate Protocol

**When Claude encounters authentication error during execution:**

1. **Recognize it's not a failure** - Missing auth is expected, not a bug
2. **Stop current task** - Don't retry repeatedly
3. **Create checkpoint:human-action on the fly** - Dynamic checkpoint, not pre-planned
4. **Provide exact authentication steps** - CLI commands, where to get keys
5. **Verify authentication** - Test that auth works before continuing
6. **Retry the original task** - Resume automation where it left off
7. **Continue normally** - One auth gate doesn't break the flow

## Quick Reference: "Can Claude automate this?"

| Action | CLI/API? | Claude does it? |
|--------|----------|-----------------|
| Deploy to Vercel | `vercel` | YES |
| Create Stripe webhook | Stripe API | YES |
| Run xcodebuild | `xcodebuild` | YES |
| Write .env file | Write tool | YES |
| Create Upstash DB | `upstash` CLI | YES |
| Install npm packages | `npm` | YES |
| Create GitHub repo | `gh` | YES |
| Run tests | `npm test` | YES |
| Create Supabase project | Web dashboard | NO (then CLI for everything else) |
| Click email verification link | No API | NO |
| Enter credit card with 3DS | No API | NO |

**Default answer: YES.** Unless explicitly in the "NO" category, Claude automates it.

## Summary

**The rule:** If Claude CAN do it, Claude MUST do it.

Checkpoints are for:
- **Verification** - Confirming Claude's automated work looks/behaves correctly
- **Decisions** - Choosing between valid approaches
- **True blockers** - Rare actions with literally no API/CLI (email links, 2FA)

**This keeps the agentic coding workflow intact - Claude does the work, you verify results.**
