# Environments: Dev → Staging → Production

> Environment separation, bootstrap steps, and per-project environment variable reference for the 360Magicians MBTQ ecosystem.

---

## Environment Tiers

| Tier | Purpose | URL pattern | Deployed by |
|------|---------|-------------|------------|
| **dev** | Local development | `localhost` | Developer / agent on local machine |
| **staging** | Integration & QA | `*.staging.mbtq.dev` | CI after merge to `develop` |
| **prod** | Live production | `*.mbtq.dev`, `*.360magicians.com` | CI after merge to `main` |

---

## Bootstrap (First-Time Setup)

### Prerequisites

| Tool | Min version | Install |
|------|------------|---------|
| Node.js | 20 LTS | [nodejs.org](https://nodejs.org) |
| pnpm | 9.x | `npm i -g pnpm` |
| Deno | 1.x | [deno.land](https://deno.land) |
| Docker | 24.x | [docker.com](https://docker.com) |
| `kubectl` | 1.29 | [kubernetes.io](https://kubernetes.io/docs/tasks/tools/) |
| Helm | 3.x | [helm.sh](https://helm.sh) |
| GCP CLI (`gcloud`) | Latest | [cloud.google.com/sdk](https://cloud.google.com/sdk) |

### Local Bootstrap (dev)

```bash
# 1. Authenticate with GCP (for Secret Manager / GCR access)
gcloud auth login
gcloud config set project mbtq-dev

# 2. Clone the repo you want to work on (example: fibonrose)
git clone https://github.com/360magicians/fibonrose
cd fibonrose
git checkout develop

# 3. Copy the dev env template and fill in values
cp .env.example .env.local

# 4. Install dependencies (see Package Managers doc for which tool per project)
pnpm install     # fibonrose, pinksync, railway-template
# OR
npm install      # municipal-dao, mbtq-dev client+server

# 5. Start the dev server
pnpm dev         # or npm run dev
```

### Staging Bootstrap (CI)

Staging is deployed automatically from the `develop` branch by GitHub Actions.
The workflow:
1. Runs lint + type-check + tests
2. Builds Docker image → pushes to `gcr.io/mbtq-dev/<service>:staging-<sha>`
3. Runs `make deploy-all ENV=staging IMAGE_TAG=staging-<sha>`
4. Posts the staging URL as a PR/commit status

### Production Bootstrap (CI)

Production is deployed from `main` only:
1. Same lint/test/build steps
2. Image tagged as `gcr.io/mbtq-dev/<service>:<git-sha>`
3. Runs `make deploy-all ENV=prod IMAGE_TAG=<git-sha>`
4. `OpsMagician` updates the monitoring dashboard URL (`managed-003`)

---

## Environment Variables — Per Project

> All secrets must be stored in **GCP Secret Manager** for staging and prod.
> For dev, use `.env.local` (gitignored). **Never commit `.env` files.**

### FibonRose

```env
# .env.local (dev only)
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_DEAFAUTH_URL=http://localhost:4000
```

```env
# staging / prod (GCP Secret Manager keys)
NEXT_PUBLIC_API_URL=https://api.360magicians.com
NEXT_PUBLIC_DEAFAUTH_URL=https://deafauth.mbtq.dev
```

---

### PinkSync

```env
# All environments — values differ per tier
DEAFAUTH_API_URL=https://deafauth.mbtq.dev        # or localhost:4000 in dev
VERTEX_AI_ENDPOINT=https://api.360magicians.com   # or mock in dev
OFFLINE_SYNC_ENABLED=true
ACCESSIBILITY_TRANSFORMS=visual,tactile,sign-language,high-contrast

# Staging / prod only
GOOGLE_APPLICATION_CREDENTIALS=/var/secrets/gcp-sa.json
```

---

### mbtq.dev (client + server)

```env
# Client (NEXT_PUBLIC_ vars are bundled into the frontend)
NEXT_PUBLIC_SUPABASE_URL=https://<project>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon key>
NEXT_PUBLIC_WS_URL=ws://localhost:8080       # dev
# NEXT_PUBLIC_WS_URL=wss://mbtq.dev/ws      # prod

# Server
DATABASE_URL=******localhost:5432/mbtq_dev
SUPABASE_SERVICE_ROLE_KEY=<service role key>
OPENAI_API_KEY=<key>
ANTHROPIC_API_KEY=<key>

# WebSocket server
WS_PORT=8080
WS_HOST=0.0.0.0
```

---

### Municipal DAO

```env
# WebSocket server
WS_PORT=8080
WS_HOST=localhost                           # dev
# WS_HOST=0.0.0.0                          # prod

NEXT_PUBLIC_WS_URL=ws://localhost:8080      # dev
# NEXT_PUBLIC_WS_URL=wss://your-domain.com/ws  # prod

# Auth
DEAFAUTH_TOKEN_SECRET=<jwt secret>
```

---

### DeafAuth

```env
# Firebase config (dev: use Firebase emulator)
FIREBASE_PROJECT_ID=mbtq-dev
FIREBASE_CLIENT_EMAIL=<service account email>
FIREBASE_PRIVATE_KEY=<private key>

# JWT
JWT_SECRET=<minimum 32 chars>
JWT_EXPIRY=7d

# API
DEAFAUTH_API_URL=https://api.deafauth.mbtq.dev
PORT=4000
```

---

### Railway Template

```env
# Auth (uses mock values in dev — demo user is built in)
DATABASE_URL=******localhost:5432/railway_dev

# App
NODE_ENV=development   # or production
PORT=3000
HOSTNAME=0.0.0.0       # required for Railway container networking

# Optional: email verification
SMTP_HOST=localhost
SMTP_PORT=1025
```

---

### MBTQUniverse

```env
NODE_ENV=development
PORT=3001

# DAO config
DAO_QUORUM_THRESHOLD=0.51
STAKING_REWARD_RATE=0.05

# Future: blockchain RPC
# RPC_URL=https://mainnet.infura.io/v3/<key>
```

---

## Secrets Management

| Tier | Method |
|------|--------|
| dev | `.env.local` (gitignored, never committed) |
| staging | GCP Secret Manager, mounted via CSI driver |
| prod | GCP Secret Manager, mounted via CSI driver |

**Rule:** any key suffixed with `_KEY`, `_SECRET`, `_TOKEN`, `_PASSWORD`, or `_PRIVATE_KEY` must
always come from Secret Manager in non-dev environments.

### Adding a new secret

```bash
# Add to GCP Secret Manager
echo -n "my-secret-value" | gcloud secrets create MY_SECRET_NAME \
  --data-file=- --project=mbtq-dev

# Grant the Kubernetes service account access
gcloud secrets add-iam-policy-binding MY_SECRET_NAME \
  --member="serviceAccount:<k8s-sa>@mbtq-dev.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

---

## Environment Promotion Checklist

Before promoting from staging → prod:

- [ ] All CI checks pass on `main`
- [ ] Staging smoke tests pass
- [ ] Helm diff reviewed (`make diff-<service> ENV=prod`)
- [ ] Secrets updated in GCP Secret Manager (prod)
- [ ] `OpsMagician` monitoring dashboard shows healthy metrics
- [ ] Rollback plan confirmed (`make rollback-<service> ENV=prod`)

---

## Related

- [Helm Deployment](./helm.md)
- [Git Workflow](./git-workflow.md)
- [Providers, Vendors & Resources](./providers.md)
- [Package Managers & Runtimes](./package-managers.md)
