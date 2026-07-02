# Triggers, Webhooks & Prompts

> Complete reference for every event trigger, webhook configuration, payload schema, HMAC verification,
> and prompt pattern used across the 360Magicians MBTQ ecosystem.

---

## Overview

The ecosystem uses three trigger layers:

| Layer | Mechanism | Who listens |
|-------|-----------|-------------|
| **GitHub Webhooks** | HTTP POST from GitHub to agent receiver | Magician agents, CI bots |
| **API-Level Triggers** | Job completion callbacks from `api.360magicians.com` | Downstream Magician agents |
| **Prompt Triggers** | Issue labels, PR comments, chat commands | Copilot, Quinn AI, bots |

---

## GitHub App — Setup

All Magician agents authenticate to GitHub as a **GitHub App** (not a personal token).
This gives fine-grained, per-repo permissions that can be audited and revoked.

### App Registration

1. Go to **GitHub → Settings → Developer settings → GitHub Apps → New GitHub App**
2. Fill in:

   ```
   App name:          360Magicians Bot
   Homepage URL:      https://360magicians.com
   Webhook URL:       https://api.360magicians.com/webhooks/github
   Webhook secret:    <generate with: openssl rand -hex 32>
   ```

3. Permissions required:

   | Permission | Level | Reason |
   |-----------|-------|--------|
   | Contents | Read & Write | Push branches, read files |
   | Pull requests | Read & Write | Open PRs, post reviews |
   | Issues | Read & Write | Create/update tracking issues |
   | Checks | Read & Write | Post CI check results |
   | Metadata | Read | Required by GitHub |
   | Actions | Read | Read workflow run status |
   | Deployments | Read & Write | Trigger and track deploys |
   | Webhooks | Read | List registered hooks |

4. Subscribe to events:
   - `push`
   - `pull_request`
   - `pull_request_review`
   - `issues`
   - `check_run`
   - `check_suite`
   - `release`
   - `workflow_run`

5. Install the App on the `360magicians`, `pinkycollie`, `MBTQ-dev`, `PinkSync`, and `DeafAUTH` orgs.

### Generating an Installation Token (per-repo)

```bash
# 1. Create a JWT signed with the App's private key
APP_ID=<your-app-id>
PRIVATE_KEY_PATH=./private-key.pem

jwt=$(python3 - <<EOF
import jwt, time, sys
payload = {"iat": int(time.time()), "exp": int(time.time()) + 600, "iss": "$APP_ID"}
key = open("$PRIVATE_KEY_PATH").read()
print(jwt.encode(payload, key, algorithm="RS256"))
EOF
)

# 2. Get the installation ID for an org
curl -H "Authorization: ******" \
     https://api.github.com/app/installations

# 3. Create an installation access token
curl -X POST \
     -H "Authorization: ******" \
     https://api.github.com/app/installations/<installation_id>/access_tokens
# → { "token": "ghs_...", "expires_at": "..." }
```

---

## Webhook Receiver

The agent webhook receiver lives at:

```
POST https://api.360magicians.com/webhooks/github
```

### HMAC Signature Verification

Every incoming webhook must be verified before processing:

```typescript
import { createHmac, timingSafeEqual } from 'crypto';

function verifyGitHubWebhook(
  payload: Buffer,
  signature: string,
  secret: string
): boolean {
  const hmac = createHmac('sha256', secret);
  hmac.update(payload);
  const digest = Buffer.from(`sha256=${hmac.digest('hex')}`);
  const sig = Buffer.from(signature);
  if (digest.length !== sig.length) return false;
  return timingSafeEqual(digest, sig);
}

// Express middleware
app.post('/webhooks/github', (req, res) => {
  const sig = req.headers['x-hub-signature-256'] as string;
  if (!verifyGitHubWebhook(req.rawBody, sig, process.env.WEBHOOK_SECRET!)) {
    return res.status(401).send('Unauthorized');
  }
  // dispatch to handler...
});
```

---

## Full Webhook Event Catalog

### `push`

| Field | Value |
|-------|-------|
| Trigger | Any push to a watched branch |
| Agent | `ValidatorMagician` (on `develop`), `InfraMagician` (on `main`) |

```json
{
  "ref": "refs/heads/develop",
  "after": "<sha>",
  "repository": { "full_name": "360magicians/<repo>" },
  "pusher": { "name": "DeveloperMagician[bot]" },
  "commits": [{ "message": "feat: ...", "added": [], "modified": [] }]
}
```

**Agent action:** Queue lint + type-check jobs; post `check_run` in-progress.

---

### `pull_request.opened` / `pull_request.synchronize`

| Field | Value |
|-------|-------|
| Trigger | PR opened or new commits pushed to PR branch |
| Agent | `ValidatorMagician` |

```json
{
  "action": "opened",
  "number": 42,
  "pull_request": {
    "title": "feat(frontend): scaffold Next.js app router",
    "head": { "ref": "agent/build-003-frontend", "sha": "<sha>" },
    "base": { "ref": "develop" },
    "body": "Closes #15\n\nJob: build-003\nInputs: wireframes.svg"
  }
}
```

**Agent action:** Run full validation suite; post review comment with `validation_report.json`.

---

### `pull_request_review.submitted`

| Field | Value |
|-------|-------|
| Trigger | A review is submitted (approved / changes_requested / commented) |
| Agent | `DeveloperMagician` |

```json
{
  "action": "submitted",
  "review": { "state": "approved", "user": { "login": "ValidatorMagician[bot]" } },
  "pull_request": { "number": 42, "mergeable": true }
}
```

**Agent action (approved):** Merge PR, delete branch, mark job `completed`, trigger next job.  
**Agent action (changes_requested):** Reopen job, post comment with diff instructions.

---

### `issues.opened` — Idea Trigger

| Field | Value |
|-------|-------|
| Trigger | Issue opened with label `idea` |
| Agent | `IdeaMagician` |

```json
{
  "action": "opened",
  "issue": {
    "title": "Deaf-first SaaS for municipal governance",
    "body": "Target audience: Deaf community leaders\nTech stack: Next.js + Supabase",
    "labels": [{ "name": "idea" }]
  }
}
```

**Agent action:** Parse `title` + `body` as prompt → `POST /idea` → comment on issue with `idea.json`.

---

### `check_run.completed`

| Field | Value |
|-------|-------|
| Trigger | A CI check finishes |
| Agent | `InfraMagician` (on success of deploy-check on `main`) |

```json
{
  "action": "completed",
  "check_run": {
    "name": "build",
    "conclusion": "success",
    "head_sha": "<sha>",
    "check_suite": { "head_branch": "main" }
  }
}
```

**Agent action:** Trigger `make deploy-all ENV=prod IMAGE_TAG=<sha>`.

---

### `release.published`

| Field | Value |
|-------|-------|
| Trigger | A GitHub Release is published |
| Agent | `OpsMagician` |

```json
{
  "action": "published",
  "release": {
    "tag_name": "v1.2.0",
    "name": "Release 1.2.0",
    "html_url": "https://github.com/360magicians/<repo>/releases/tag/v1.2.0"
  }
}
```

**Agent action:** Update monitoring dashboard (`managed-003`); post release notes to Deaf community channels.

---

### `workflow_run.completed`

| Field | Value |
|-------|-------|
| Trigger | A GitHub Actions workflow finishes |
| Agent | `InfraMagician` |

```json
{
  "action": "completed",
  "workflow_run": {
    "name": "CI",
    "conclusion": "success",
    "head_branch": "main",
    "head_sha": "<sha>"
  }
}
```

**Agent action:** Same as `check_run.completed` — deploy to prod if branch is `main`.

---

## API-Level Job Completion Triggers

When a Magician agent finishes a job it calls the lifecycle API to signal completion:

```
POST https://api.360magicians.com/jobs/<job_id>/complete
Authorization: ******
Content-Type: application/json

{
  "status": "completed",
  "outputs": {
    "repository_url": "https://github.com/360magicians/deaf-gov-platform"
  }
}
```

The API then checks `depends_on` for all jobs that depend on `<job_id>` and
enqueues them automatically.

### Job Completion Webhook (outbound)

The API emits an outbound webhook to any registered subscriber on job status change:

```json
{
  "event": "job.completed",
  "job_id": "build-001",
  "stage": "BUILD",
  "assigned_to": "DeveloperMagician",
  "outputs": {
    "repository_url": "https://github.com/360magicians/deaf-gov-platform"
  },
  "triggered_jobs": ["build-002", "build-004"],
  "timestamp": "2024-01-20T10:30:00Z"
}
```

Subscribers register via:

```bash
curl -X POST https://api.360magicians.com/webhooks/subscribe \
  -H "Authorization: ******" \
  -d '{
    "url": "https://your-listener.example.com/magician-events",
    "events": ["job.completed", "job.failed"],
    "secret": "<hmac-secret>"
  }'
```

---

## Prompt Triggers

### GitHub Issue Label → Agent

| Label | Agent Triggered | Action |
|-------|----------------|--------|
| `idea` | `IdeaMagician` | Generate idea from issue title/body |
| `validate` | `ValidatorMagician` | Validate linked `idea.json` attachment |
| `research` | `ResearchMagician` | Run market research on issue topic |
| `build` | `DeveloperMagician` | Create repo from issue spec |
| `compliance-check` | `ComplianceMagician` | Generate compliance report |
| `incident` | `OpsMagician` | Open incident tracking and alert |

### GitHub PR Comment → Agent

Post a comment beginning with `/magician` to trigger an agent directly:

| Command | Agent | Action |
|---------|-------|--------|
| `/magician validate` | `ValidatorMagician` | Re-run validation on this PR |
| `/magician review` | `ValidatorMagician` | Post full code review |
| `/magician deploy staging` | `InfraMagician` | Deploy this branch to staging |
| `/magician deploy prod` | `InfraMagician` | Deploy `main` to production |
| `/magician rollback` | `InfraMagician` | Roll back last Helm release |
| `/magician docs` | `DeveloperMagician` | Auto-generate docs for changed files |
| `/magician compliance` | `ComplianceMagician` | Run compliance scan |

Example:
```
@360MagiciansBot /magician deploy staging
```

### PinkSync Accessibility Event Trigger

```python
# Any application can emit an accessibility event that triggers PinkSync transforms
import httpx

httpx.post("https://api.pinksync.io/v1/events", json={
  "app_id": "deaf-gov-platform",
  "user_id": "user-12345",
  "intent": "visual_only",
  "compliance_level": "gold",
  "event_type": "page_load",
  "metadata": { "route": "/proposals/1" }
})
```

PinkSync responds by emitting transform events to all subscribed downstream services.

---

## Related

- [Pipeline Handoffs](./pipeline-handoffs.md)
- [Copilot, Bots & Auth Access](./copilot-bots.md)
- [Git Workflow](./git-workflow.md)
- [Environments (dev → prod)](./environments.md)
