# Copilot, Bots & Auth Access

> GitHub Copilot suggested prompts, authorized bot/app registrations, and all auth access patterns
> (OAuth, PAT, JWT, GitHub App installation tokens) used across the 360Magicians ecosystem.

---

## GitHub Copilot — Suggested Prompts

Use these prompts in GitHub Copilot Chat (VS Code, GitHub.com, or Copilot CLI) to get the most
effective assistance across this ecosystem.

### Codebase Orientation

```
@workspace What is the IDEA → BUILD → GROW → MANAGED lifecycle and which agents handle each stage?
@workspace Show me all Magician agent types and their job IDs.
@workspace How does DeafAUTH authentication work across the ecosystem?
@workspace Which projects use pnpm vs npm vs Deno?
@workspace What environment variables does PinkSync need?
```

### Code Generation

```
# Next.js / TypeScript
Generate a Next.js server action for submitting an idea to POST /idea using DeafAUTH bearer auth.

Generate a TypeScript hook that connects to the Municipal DAO WebSocket and listens for vote events.

Create a Zod schema for the BuildRepoRequest matching the openapi/components/schemas/BuildRepoRequest.yaml spec.

Generate a DeafAUTH SDK registration call with ASL as the preferred language.

# Accessibility
Add WCAG 2.1 AA compliant visual notification component using shadcn/ui and Tailwind.

Generate a PinkSync accessibility event POST call for a page_load event with compliance_level gold.
```

### PR Review

```
@workspace Review this PR for accessibility issues — does it meet Deaf-first design principles?

@workspace Does this change break any Magician agent job contracts (inputs/outputs)?

@workspace Check if this PR respects the pnpm-only rule for 360magicians org projects.
```

### Debugging

```
@workspace Why would a ValidatorMagician review be failing on PR #<number>?

@workspace How do I fix a WebSocket reconnection issue in Municipal DAO?

@workspace What's the correct HMAC signature verification for GitHub webhooks?
```

### Documentation Generation

```
Generate a markdown doc for this new API endpoint following the style of docs/projects/deafauth.md.

Add JSDoc comments to this server action following the Railway template patterns.

Create an .env.example file for this new project following the style in docs/environments.md.
```

---

## Copilot CLI — Suggested Commands

```bash
# Explain what a file does
gh copilot explain "$(cat docs/git-workflow.md)"

# Suggest shell commands
gh copilot suggest "deploy pinksync to staging with helm"

# Translate a natural language request to a command
gh copilot suggest "create a GitHub branch named agent/build-003-frontend"
gh copilot suggest "verify the HMAC signature of an incoming GitHub webhook"
gh copilot suggest "list all open PRs from agent branches in a repo"
```

---

## Authorized Bots & GitHub Apps

### 360Magicians Bot (main lifecycle agent)

| Setting | Value |
|---------|-------|
| App name | `360MagiciansBot` |
| Login | `360MagiciansBot[bot]` |
| Webhook endpoint | `https://api.360magicians.com/webhooks/github` |
| Auth method | GitHub App installation token (JWT → access token) |
| Scopes | Contents R/W, Pull Requests R/W, Issues R/W, Checks R/W, Actions R, Deployments R/W |
| Orgs installed | `360magicians`, `pinkycollie`, `MBTQ-dev`, `PinkSync`, `DeafAUTH` |

### ValidatorMagician Bot

| Setting | Value |
|---------|-------|
| App name | `ValidatorMagicianBot` |
| Trigger | PR opened/synchronized |
| Posts | PR review comments, `validation_report.json` |
| Merge permission | Approve + merge when all checks pass |

### GitHub Copilot (Coding Agent)

| Setting | Value |
|---------|-------|
| App name | `github-copilot[bot]` |
| Auth | GitHub OAuth (user-authorized) |
| Allowed | Comment on PRs, suggest code, generate docs |
| Not allowed | Merge PRs, push directly to `main` |

### Dependabot

| Setting | Value |
|---------|-------|
| Config | `.github/dependabot.yml` |
| Ecosystem | `npm` (root `/`) |
| Schedule | Daily |
| Allowed packages | `@redocly/cli` only (intentionally restricted) |

---

## Auth Access Patterns

### 1. GitHub App Installation Token (agents → GitHub API)

Used by all Magician bots. Scoped to a specific installation (org or repo).
Expires after 1 hour — agents refresh as needed.

```typescript
import { App } from '@octokit/app';

const app = new App({
  appId: process.env.GITHUB_APP_ID!,
  privateKey: process.env.GITHUB_PRIVATE_KEY!,
  webhooks: { secret: process.env.WEBHOOK_SECRET! },
});

// Get an Octokit instance scoped to a specific installation
const octokit = await app.getInstallationOctokit(installationId);

// Use it
await octokit.rest.pulls.create({
  owner: '360magicians',
  repo: 'deaf-gov-platform',
  title: 'feat(frontend): scaffold Next.js',
  head: 'agent/build-003-frontend',
  base: 'develop',
});
```

---

### 2. DeafAUTH ****** (humans + agents → lifecycle API)

All calls to `https://api.360magicians.com` require a DeafAUTH JWT.

#### Obtain a token

```bash
# Register (first time)
curl -X POST https://api.deafauth.mbtq.dev/v1/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "agent@360magicians.com",
    "password": "<secure-password>",
    "name": "DeveloperMagician"
  }'

# Login
curl -X POST https://api.deafauth.mbtq.dev/v1/login \
  -H "Content-Type: application/json" \
  -d '{ "email": "agent@360magicians.com", "password": "<password>" }'
# → { "token": "eyJ...", "expires_in": 604800 }
```

#### Use the token

```bash
export DEAFAUTH_TOKEN=eyJ...

curl -H "Authorization: ******" \
     https://api.360magicians.com/jobs/list
```

#### Token structure (JWT payload)

```json
{
  "user_id": "agent-dev-001",
  "auth_type": "service_account",
  "permissions": ["IDEA_GENERATOR", "BUILD_ACCESS", "GROW_DASHBOARD", "MANAGED_ACCESS"],
  "profile": {
    "display_name": "DeveloperMagician",
    "language": "ASL",
    "privacy": "private"
  },
  "iat": 1705747200,
  "exp": 1706352000
}
```

---

### 3. GitHub Personal Access Token (PAT) — human developers only

PATs are for **human developers only**, not agents. Fine-grained PATs are preferred.

| Use case | Minimum scopes |
|----------|---------------|
| Clone private repos | `repo:contents (read)` |
| Push to feature branch | `repo:contents (write)` |
| Open PR | `pull_requests (write)` |
| Trigger workflow manually | `actions (write)` |

```bash
# Store in environment (never hardcode)
export GITHUB_TOKEN=github_pat_...

# Use with gh CLI
gh auth login --with-token <<< $GITHUB_TOKEN

# Verify
gh auth status
```

**Fine-grained PAT settings:**
- Resource owner: `360magicians` (or the specific org)
- Repository access: only the repos you need
- Expiry: maximum 90 days; rotate at 60 days

---

### 4. Supabase Service Role Key (server-side only)

Used by mbtq-dev server for privileged database operations.

```typescript
// server/lib/supabase.ts
import { createClient } from '@supabase/supabase-js';

// NEVER expose this key to the browser
const supabaseAdmin = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!, // server only
);
```

The anon key (`NEXT_PUBLIC_SUPABASE_ANON_KEY`) is safe to expose in the browser — it is
Row Level Security (RLS) protected.

---

### 5. GCP Service Account (InfraMagician → GCP)

Used by `InfraMagician` for GKE deployments and GCR image pushes.

```bash
# Authenticate in CI (GitHub Actions)
- uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_SA_KEY }}

- uses: google-github-actions/setup-gcloud@v2

# Push image to GCR
gcloud auth configure-docker gcr.io
docker push gcr.io/mbtq-dev/<service>:<tag>

# Deploy via kubectl
gcloud container clusters get-credentials mbtq-cluster \
  --region us-central1 --project mbtq-dev
```

**Service account permissions (least privilege):**

| Role | Reason |
|------|--------|
| `roles/container.developer` | Deploy to GKE |
| `roles/storage.admin` (scoped to `gcr.io/mbtq-dev`) | Push/pull container images |
| `roles/secretmanager.secretAccessor` | Read secrets at deploy time |

---

### 6. Firebase Service Account (DeafAuth)

```typescript
// functions/index.ts
import * as admin from 'firebase-admin';

admin.initializeApp({
  credential: admin.credential.cert({
    projectId: process.env.FIREBASE_PROJECT_ID,
    clientEmail: process.env.FIREBASE_CLIENT_EMAIL,
    privateKey: process.env.FIREBASE_PRIVATE_KEY?.replace(/\\n/g, '\n'),
  }),
});
```

For local development, use the **Firebase Emulator**:

```bash
firebase emulators:start --only auth,firestore,functions
# Auth emulator: http://localhost:9099
# Firestore emulator: http://localhost:8080
```

---

## Bot Authorization Checklist

When onboarding a new bot or GitHub App to the ecosystem:

- [ ] Register as a GitHub App (not a personal token)
- [ ] Set webhook URL to `https://api.360magicians.com/webhooks/github`
- [ ] Store `WEBHOOK_SECRET` and `GITHUB_PRIVATE_KEY` in GCP Secret Manager
- [ ] Grant minimum required permissions only
- [ ] Install on specific repos/orgs (not "all repositories" unless required)
- [ ] Add bot login (`BotName[bot]`) to the `CODEOWNERS` exception list if it needs to push to protected branches
- [ ] Register the bot's DeafAUTH service account with the `service_account` auth type
- [ ] Document the bot in this file under **Authorized Bots & GitHub Apps**
- [ ] Add webhook subscription to `api.360magicians.com/webhooks/subscribe` if it needs job events

---

## Protected Branch Push Access for Bots

Bots that need to push to `develop` (e.g. `DeveloperMagician`) require a branch protection bypass rule:

**GitHub → Settings → Branches → develop → Edit → Allow specified actors to bypass required pull requests**

Add: `360MagiciansBot[bot]`

> Bots **must never** bypass protection on `main`. All `main` merges must go through a PR with at least one human review.

---

## Related

- [Triggers, Webhooks & Prompts](./triggers-webhooks.md)
- [Pipeline Handoffs](./pipeline-handoffs.md)
- [Environments & Bootstrap](./environments.md)
- [Git Workflow](./git-workflow.md)
