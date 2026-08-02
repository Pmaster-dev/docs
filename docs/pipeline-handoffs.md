# Pipeline Handoffs

> Every agent-to-agent handoff point in the IDEA → BUILD → GROW → MANAGED pipeline:
> trigger, input contract, output contract, and the exact commands to run at each step.

---

## Pipeline Overview

```
 ┌────────────────────────────────────────────────────────────────────┐
 │  IDEA Stage           BUILD Stage          GROW Stage   MANAGED    │
 │                                                                    │
 │  issue (label:idea)                                                │
 │       │                                                            │
 │  IdeaMagician ──────► ValidatorMagician ──► ResearchMagician       │
 │                              │                    │               │
 │                         idea.json        market_research.md        │
 │                              │                    │               │
 │                         DeveloperMagician ◄────────┘               │
 │                         (build-001: create repo)                   │
 │                              │                                     │
 │                    ┌─────────┴──────────┐                          │
 │               DesignMagician     DeveloperMagician                 │
 │               (build-002)        (build-004: backend)              │
 │                    │                    │                          │
 │              wireframes.svg      backend_codebase.zip              │
 │                    │                    │                          │
 │               DeveloperMagician ◄────────┘                         │
 │               (build-003: frontend)                                │
 │                    │                                               │
 │              frontend_codebase.zip                                 │
 │                    │                                               │
 │               InfraMagician (build-005: deploy staging)            │
 │                    │                                               │
 │               staging_url                                          │
 │                    │                                               │
 │          ┌─────────┼──────────┐                                    │
 │     DataMagician  GrowthMagician  PartnershipMagician               │
 │     (grow-001)   (grow-002)       (grow-003)                        │
 │          │            │                │                           │
 │   dashboard_url  campaign_report  onboarding_status                │
 │          └─────────┬──────────┘                                    │
 │                    ▼                                               │
 │          ComplianceMagician / FinanceMagician / OpsMagician        │
 │          (managed-001 / managed-002 / managed-003)                 │
 └────────────────────────────────────────────────────────────────────┘
```

---

## Handoff 1: Human → IdeaMagician

**Trigger:** GitHub Issue opened with label `idea`  
**Or:** Direct API call

### Input

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `prompt` | string | ✅ | Idea description (from issue title + body) |
| `target_audience` | string | ❌ | Optional audience hint |

### Command

```bash
# Via API
curl -X POST https://api.360magicians.com/idea \
  -H "Authorization: ******" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Deaf-first SaaS for municipal governance",
    "target_audience": "Deaf community leaders"
  }'

# Via GitHub Issue
# Open issue with label "idea" — IdeaMagician picks it up automatically
```

### Output (`idea.json`)

```json
{
  "idea_id": "idea-abc123",
  "title": "Deaf Municipal Governance Platform",
  "description": "...",
  "target_audience": "Deaf community leaders",
  "accessibility_features": ["ASL interface", "visual notifications", "captions"],
  "suggested_tech_stack": "Next.js, TypeScript, Supabase",
  "created_at": "2024-01-20T10:00:00Z"
}
```

### Handoff Signal

IdeaMagician posts `idea.json` as a comment on the triggering issue and adds label `validate`.

---

## Handoff 2: IdeaMagician → ValidatorMagician

**Trigger:** Issue comment containing `idea.json` + label `validate`

### Input

`idea.json` (output from Handoff 1)

### Command

```bash
curl -X POST https://api.360magicians.com/idea/validate \
  -H "Authorization: ******" \
  -H "Content-Type: application/json" \
  -d @idea.json
```

### Output (`validation_report.json`)

```json
{
  "idea_id": "idea-abc123",
  "valid": true,
  "compliance_score": 92,
  "accessibility_score": 98,
  "risks": [],
  "recommendations": ["Add offline mode for low-bandwidth users"],
  "approved_for_research": true
}
```

### Handoff Signal

ValidatorMagician adds label `research` to the issue if `approved_for_research: true`.  
On failure (`valid: false`) it adds label `needs-revision` and stops the pipeline.

---

## Handoff 3: ValidatorMagician → ResearchMagician

**Trigger:** Issue label `research`

### Input

| Field | Type | Required |
|-------|------|----------|
| `topic` | string | ✅ |
| `target_audience` | string | ✅ |

### Command

```bash
curl -X POST https://api.360magicians.com/idea/research \
  -H "Authorization: ******" \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "Deaf Municipal Governance Platform",
    "target_audience": "Deaf community leaders"
  }'
```

### Output

`market_research_report.md` — attached to the issue as a file comment.

### Handoff Signal

ResearchMagician adds label `build` to the issue.

---

## Handoff 4: ResearchMagician → DeveloperMagician (build-001)

**Trigger:** Issue label `build`

### Input

| Field | Type | Required |
|-------|------|----------|
| `project_name` | string | ✅ |
| `project_description` | string | ✅ |
| `tech_stack` | string | ❌ |

### Command

```bash
curl -X POST https://api.360magicians.com/build/repo \
  -H "Authorization: ******" \
  -H "Content-Type: application/json" \
  -d '{
    "project_name": "deaf-gov-platform",
    "project_description": "Deaf-first municipal governance platform",
    "tech_stack": "Next.js, TypeScript, Supabase"
  }'
```

### Output

```json
{ "repository_url": "https://github.com/360magicians/deaf-gov-platform" }
```

### Handoff Signal

DeveloperMagician:
1. Creates the GitHub repo
2. Creates `main` + `develop` branches
3. Opens tracking issue in new repo linking back to original idea issue
4. Marks job `build-001` complete → API enqueues `build-002` and `build-004` in parallel

---

## Handoff 5a: DeveloperMagician → DesignMagician (build-002)

**Trigger:** `build-001` complete

### Input

`idea.json`

### Command

```bash
# DesignMagician calls the design endpoint (internal)
curl -X POST https://api.360magicians.com/build/design \
  -H "Authorization: ******" \
  -d @idea.json
```

### Output

`wireframes.svg` — committed to `agent/build-002-wireframes` branch.

### Handoff Signal

DesignMagician opens PR `agent/build-002-wireframes` → `develop`.  
On merge, marks `build-002` complete → API enqueues `build-003`.

---

## Handoff 5b: DeveloperMagician → DeveloperMagician (build-004)

**Trigger:** `build-001` complete (runs in parallel with build-002)

### Input

| Field | Type |
|-------|------|
| `api_specs.json` | derived from `idea.json` |
| `tech_stack` | string |

### Output

`backend_codebase.zip` — committed to `agent/build-004-backend` branch.

### Handoff Signal

PR `agent/build-004-backend` → `develop`. On merge → marks `build-004` complete.

---

## Handoff 6: DesignMagician → DeveloperMagician (build-003)

**Trigger:** `build-002` complete (wireframes merged)

**`depends_on`: `["build-002"]`**

### Input

| Field | Type |
|-------|------|
| `wireframes.svg` | from `build-002` output |
| `tech_stack` | string |

### Output

`frontend_codebase.zip` — committed to `agent/build-003-frontend` branch.

### Handoff Signal

PR `agent/build-003-frontend` → `develop`. ValidatorMagician auto-reviews.  
On merge → marks `build-003` complete.

---

## Handoff 7: DeveloperMagician → InfraMagician (build-005)

**Trigger:** Both `build-003` AND `build-004` complete

**`depends_on`: `["build-003", "build-004"]`**

### Input

| Field | Type |
|-------|------|
| `frontend_codebase.zip` | from `build-003` |
| `backend_codebase.zip` | from `build-004` |

### Commands

```bash
# 1. Build Docker images
docker build -t gcr.io/mbtq-dev/deaf-gov-platform:$GIT_SHA .

# 2. Push to GCR
docker push gcr.io/mbtq-dev/deaf-gov-platform:$GIT_SHA

# 3. Deploy to staging
make deploy-mbtq-platform ENV=staging IMAGE_TAG=$GIT_SHA

# 4. Wait for rollout
kubectl rollout status deployment/mbtq-platform -n staging

# 5. Run smoke test
curl -f https://staging.deaf-gov-platform.mbtq.dev/api/health
```

### Output

```json
{ "staging_url": "https://staging.deaf-gov-platform.mbtq.dev" }
```

### Handoff Signal

InfraMagician marks `build-005` complete → API enqueues `grow-001`, `grow-002`, `grow-003`.

---

## Handoff 8: InfraMagician → GROW Agents (parallel)

### grow-001 — DataMagician

**Trigger:** `build-005` complete

```bash
curl -X POST https://api.360magicians.com/grow/analytics \
  -H "Authorization: ******" \
  -d '{ "app_url": "https://deaf-gov-platform.mbtq.dev", "kpis": ["dau", "accessibility_score"] }'
```

**Output:** `dashboard_url`

---

### grow-002 — GrowthMagician

```bash
curl -X POST https://api.360magicians.com/grow/campaign \
  -H "Authorization: ******" \
  -d @campaign_brief.md    # Content-Type: text/markdown
```

**Output:** `campaign_report.md`

---

### grow-003 — PartnershipMagician

```bash
curl -X POST https://api.360magicians.com/grow/partners \
  -H "Authorization: ******" \
  -d @partner_list.csv
```

**Output:** `onboarding_status.json`

---

## Handoff 9: GROW Agents → MANAGED Agents

**Trigger:** All `grow-*` jobs complete

### managed-001 — ComplianceMagician

```bash
curl -X POST https://api.360magicians.com/managed/compliance \
  -H "Authorization: ******" \
  -d '{ "reporting_period": "2024-Q1" }'
```

**Output:** `compliance_report.pdf` URL

---

### managed-002 — FinanceMagician

```bash
curl -X POST https://api.360magicians.com/managed/finance \
  -H "Authorization: ******" \
  --data-binary @transactions.csv
```

**Output:** `financial_report.pdf` URL

---

### managed-003 — OpsMagician

```bash
curl -X POST https://api.360magicians.com/managed/ops \
  -H "Authorization: ******" \
  -d '{ "app_url": "https://deaf-gov-platform.mbtq.dev" }'
```

**Output:** `monitoring_dashboard_url`

---

## Complete Handoff Table

| # | From | To | Trigger | Input | Output | Signal |
|---|------|----|---------|-------|--------|--------|
| 1 | Human | IdeaMagician | Issue label `idea` | prompt | `idea.json` | Label `validate` |
| 2 | IdeaMagician | ValidatorMagician | Label `validate` | `idea.json` | `validation_report.json` | Label `research` or `needs-revision` |
| 3 | ValidatorMagician | ResearchMagician | Label `research` | topic + audience | `market_research_report.md` | Label `build` |
| 4 | ResearchMagician | DeveloperMagician | Label `build` | project spec | `repository_url` | Job `build-001` complete |
| 5a | DeveloperMagician | DesignMagician | `build-001` ✅ | `idea.json` | `wireframes.svg` | Job `build-002` complete |
| 5b | DeveloperMagician | DeveloperMagician | `build-001` ✅ | api specs | `backend_codebase.zip` | Job `build-004` complete |
| 6 | DesignMagician | DeveloperMagician | `build-002` ✅ | wireframes | `frontend_codebase.zip` | Job `build-003` complete |
| 7 | DeveloperMagician × 2 | InfraMagician | `build-003` ✅ + `build-004` ✅ | codebases | `staging_url` | Job `build-005` complete |
| 8a | InfraMagician | DataMagician | `build-005` ✅ | `app_url` | `dashboard_url` | Job `grow-001` complete |
| 8b | InfraMagician | GrowthMagician | `build-005` ✅ | `campaign_brief.md` | `campaign_report.md` | Job `grow-002` complete |
| 8c | InfraMagician | PartnershipMagician | `build-005` ✅ | `partner_list.csv` | `onboarding_status.json` | Job `grow-003` complete |
| 9a | GROW ✅ | ComplianceMagician | All grow jobs ✅ | `reporting_period` | `compliance_report.pdf` | Job `managed-001` complete |
| 9b | GROW ✅ | FinanceMagician | All grow jobs ✅ | `transactions.csv` | `financial_report.pdf` | Job `managed-002` complete |
| 9c | GROW ✅ | OpsMagician | All grow jobs ✅ | `app_url` | `monitoring_dashboard_url` | Job `managed-003` complete |

---

## Failure Handling at Handoff Points

| Scenario | Action |
|----------|--------|
| Job fails | Agent marks job `failed`, posts error as issue comment, adds label `needs-revision` |
| `depends_on` job timed out | Pipeline paused; `OpsMagician` alerts via monitoring dashboard |
| Validation rejected (PR changes requested) | Job re-queued; agent iterates on branch |
| Staging smoke test fails | `build-005` fails; `InfraMagician` rolls back Helm release |
| Webhook delivery fails | GitHub retries up to 3 times with exponential back-off |

---

## Related

- [Triggers, Webhooks & Prompts](./triggers-webhooks.md)
- [Copilot, Bots & Auth Access](./copilot-bots.md)
- [Git Workflow](./git-workflow.md)
- [Helm Deployment](./helm.md)
