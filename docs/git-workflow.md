# Git Workflow & GitHub Integration

> How agents and humans interact with git, GitHub branches, the GitHub Graph API, and CI/CD events.

---

## Branch Strategy

The 360Magicians ecosystem uses a **lifecycle-aligned branching model**.

| Branch | Purpose | Who creates it |
|--------|---------|----------------|
| `main` | Production-ready code | Protected — merged via PR only |
| `develop` | Integration branch | Humans + agents merge feature branches here |
| `agent/<job-id>` | Work created by a Magician agent | `DeveloperMagician` automatically |
| `feature/<slug>` | Human-driven feature work | Developer |
| `ux/ui` | UI/UX iteration branch | Designer (e.g. FibonRose `ux/ui`) |
| `fix/<slug>` | Bug fixes | Developer or `OpsMagician` |
| `release/<version>` | Release preparation | `InfraMagician` |

### Naming Conventions

```
agent/build-003-frontend-codebase
feature/deaf-auth-wallet-login
fix/websocket-reconnect-timeout
release/1.2.0
ux/ui
```

---

## Agent Git Flow

When `DeveloperMagician` receives a `build-001` job (create repo), it:

1. Calls `POST /build/repo` → GitHub API `POST /orgs/{org}/repos`
2. Initialises the repo with a `main` branch and README
3. Creates a `develop` branch from `main`
4. Opens a tracking issue with the full job spec (`job_id`, `inputs`, `outputs`)
5. Subsequent build jobs (`build-003`, `build-004`) branch from `develop` as `agent/<job-id>`
6. On job completion the agent opens a PR: `agent/<job-id>` → `develop`
7. `ValidatorMagician` reviews the PR and posts a review comment with `validation_report.json`
8. On approval, the PR is merged and the branch deleted

```
main
 └── develop
      ├── agent/build-003-frontend   ← DeveloperMagician
      ├── agent/build-004-backend    ← DeveloperMagician
      └── feature/dashboard-ui       ← human
```

---

## GitHub Graph API

The GitHub **GraphQL API** (`api.github.com/graphql`) is used by agents to:

- Query repository status and open PRs
- Retrieve commit history for a job branch
- Check CI check-run statuses before merging
- List review comments from `ValidatorMagician`

### Example: check PR status

```graphql
query CheckPR($owner: String!, $repo: String!, $number: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $number) {
      state
      mergeable
      reviewDecision
      commits(last: 1) {
        nodes {
          commit {
            statusCheckRollup {
              state
            }
          }
        }
      }
    }
  }
}
```

### Example: list agent branches

```graphql
query AgentBranches($owner: String!, $repo: String!) {
  repository(owner: $owner, name: $repo) {
    refs(refPrefix: "refs/heads/agent/", first: 20) {
      nodes {
        name
        target {
          ... on Commit { authoredDate }
        }
      }
    }
  }
}
```

---

## GitHub Events & Webhooks

Agents subscribe to GitHub webhook events to trigger lifecycle actions:

| Event | Payload | Agent Action |
|-------|---------|-------------|
| `push` (to `develop`) | Branch, commits | `ValidatorMagician` runs lint/test |
| `pull_request.opened` | PR number, branch | `ValidatorMagician` starts review |
| `pull_request_review.submitted` | Review state | `DeveloperMagician` checks approval |
| `check_run.completed` | Status, conclusion | `InfraMagician` triggers deploy if passing |
| `release.published` | Tag, release URL | `OpsMagician` updates monitoring dashboard |
| `issues.opened` | Title, body | `IdeaMagician` parses idea prompt if labelled `idea` |

### Webhook Payload Example (push)

```json
{
  "ref": "refs/heads/agent/build-003-frontend",
  "after": "abc1234",
  "repository": { "full_name": "360magicians/deaf-gov-platform" },
  "pusher": { "name": "DeveloperMagician[bot]" },
  "commits": [
    {
      "message": "feat(frontend): scaffold Next.js app router",
      "added": ["src/app/page.tsx"],
      "modified": []
    }
  ]
}
```

---

## CI/CD Pipeline

Each repository runs GitHub Actions:

```
push / PR → lint → type-check → test → build → (on main) deploy
```

### Standard Workflow Steps

| Step | Tool | Target | Output |
|------|------|--------|--------|
| Lint | ESLint / oxlint | `src/**` | Pass / fail with annotations |
| Type check | `tsc --noEmit` | `tsconfig.json` | Type errors |
| Unit tests | Vitest / Jest | `**/*.test.ts` | Coverage report |
| Build | Next.js / Vite | `src/` | `.next/` or `dist/` |
| Docker build | Docker / Buildx | `Dockerfile` | OCI image pushed to GCR |
| Deploy | Helm / Railway | Helm values or Railway project | Staging / production URL |

### Branch Protection Rules (recommended)

```
main:
  required_status_checks: [lint, type-check, test, build]
  required_reviews: 1
  dismiss_stale_reviews: true
  enforce_admins: false

develop:
  required_status_checks: [lint, type-check]
  required_reviews: 0
```

---

## Job Dependency Graph (DAG)

The `depends_on` field in the Job schema forms a directed acyclic graph (DAG) that maps directly to git branch sequencing:

```
idea-001 (IdeaMagician)
    └── idea-002 (ValidatorMagician)
         └── idea-003 (ResearchMagician)
              └── build-001 (DeveloperMagician — creates repo & develop branch)
                   ├── build-002 (DesignMagician — wireframes on agent/build-002)
                   │    └── build-003 (DeveloperMagician — frontend on agent/build-003)
                   └── build-004 (DeveloperMagician — backend on agent/build-004)
                        └── build-005 (InfraMagician — deploys staging)
                             └── grow-001 / grow-002 / grow-003 …
```

Each node is a git branch; each edge is a `depends_on` constraint enforced before the next branch is created.

---

## Related

- [Infrastructure / Ecosystem Architecture](./infrastructure.md)
- [Helm Deployment](./helm.md)
- [Environments (dev → prod)](./environments.md)
- [DeafAuth — Lifecycle Job API](./projects/deafauth.md)
