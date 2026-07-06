# Helm Deployment

> Kubernetes deployment via Helm for the 360Magicians MBTQ ecosystem services on GCP (project: `mbtq.dev`).

---

## Overview

All long-running services in the MBTQ ecosystem are deployed to Kubernetes (GKE) using **Helm charts**.
`InfraMagician` is the agent responsible for `helm upgrade --install` calls during the BUILD and MANAGED lifecycle stages.

---

## Chart Layout

```
helm/
├── charts/
│   ├── 360magicians/       # Vertex AI Multi-Domain Hub
│   ├── pinksync/           # Accessibility AI Middleware
│   ├── mbtq-platform/      # Deaf-First Platform (mbtq.dev)
│   ├── deafauth/           # Universal Identity Layer
│   └── mbtquniverse/       # DAO + Web3 + Staking
├── environments/
│   ├── dev/
│   │   └── values.yaml     # Development overrides
│   ├── staging/
│   │   └── values.yaml     # Staging overrides
│   └── prod/
│       └── values.yaml     # Production values
└── Makefile                # Deployment targets
```

---

## Services & Chart Values

### 360 Magicians (Vertex AI Hub)

```yaml
# Default values
image:
  repository: gcr.io/mbtq-dev/360magicians-vertex-ai
  tag: latest
  pullPolicy: Always

replicaCount: 5

service:
  port: 8080

env:
  VERTEX_AI_PROJECT: mbtq.dev
  VERTEX_AI_LOCATION: us-central1
  ACCESSIBILITY_FIRST: "true"

ingress:
  hosts:
    - creative-ai.360magicians.com
    - design-ai.360magicians.com
    - content-ai.360magicians.com
    - accessibility-ai.360magicians.com
```

### pinksync (Accessibility Middleware)

```yaml
image:
  repository: gcr.io/mbtq-dev/pinksync-accessibility
  tag: latest

replicaCount: 3

service:
  port: 8081

env:
  DEAFAUTH_API_URL: https://deafauth.mbtq.dev
  VERTEX_AI_ENDPOINT: https://api.360magicians.com
  OFFLINE_SYNC_ENABLED: "true"
  ACCESSIBILITY_TRANSFORMS: visual,tactile,sign-language,high-contrast
```

### mbtq-platform (Deaf-First Platform)

```yaml
image:
  repository: gcr.io/mbtq-dev/mbtq-platform
  tag: latest

replicaCount: 4

service:
  port: 3000

env:
  NEXT_PUBLIC_WS_URL: wss://mbtq.dev/ws
  DEAFAUTH_API_URL: https://deafauth.mbtq.dev
```

### deafauth (Identity Layer)

```yaml
image:
  repository: gcr.io/mbtq-dev/deafauth
  tag: latest

replicaCount: 3

service:
  port: 4000

env:
  JWT_SECRET: <from Secret Manager>
  FIREBASE_PROJECT_ID: mbtq-dev
```

### mbtquniverse (DAO + Web3)

```yaml
image:
  repository: gcr.io/mbtq-dev/mbtquniverse
  tag: latest

replicaCount: 2

service:
  port: 3001

env:
  DAO_QUORUM_THRESHOLD: "0.51"
  STAKING_REWARD_RATE: "0.05"
```

---

## Environment Overrides

### dev `environments/dev/values.yaml`

```yaml
replicaCount: 1          # Single replica to save cost
image:
  tag: dev-latest
env:
  LOG_LEVEL: debug
  MOCK_AI: "true"        # Use mock AI responses in dev
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

### staging `environments/staging/values.yaml`

```yaml
replicaCount: 2
image:
  tag: staging-latest
env:
  LOG_LEVEL: info
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi
```

### prod `environments/prod/values.yaml`

```yaml
replicaCount: 5          # Full HA replica count
image:
  tag: ""               # Set by CI via --set image.tag=$GIT_SHA
env:
  LOG_LEVEL: warn
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
podDisruptionBudget:
  minAvailable: 2
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 2000m
    memory: 2Gi
```

---

## Deployment Targets (Makefile)

```makefile
ENV ?= dev

# Deploy a single service
deploy-%:
	helm upgrade --install $* ./helm/charts/$* \
	  -f ./helm/environments/$(ENV)/values.yaml \
	  --namespace $(ENV) \
	  --create-namespace \
	  --set image.tag=$(IMAGE_TAG)

# Deploy all services
deploy-all:
	$(MAKE) deploy-360magicians ENV=$(ENV)
	$(MAKE) deploy-pinksync     ENV=$(ENV)
	$(MAKE) deploy-mbtq-platform ENV=$(ENV)
	$(MAKE) deploy-deafauth     ENV=$(ENV)
	$(MAKE) deploy-mbtquniverse ENV=$(ENV)

# Rollback a service
rollback-%:
	helm rollback $* 0 --namespace $(ENV)

# Show diff before upgrading
diff-%:
	helm diff upgrade $* ./helm/charts/$* \
	  -f ./helm/environments/$(ENV)/values.yaml

# Render templates without deploying
template-%:
	helm template $* ./helm/charts/$* \
	  -f ./helm/environments/$(ENV)/values.yaml
```

### Usage

```bash
# Deploy pinksync to dev
make deploy-pinksync ENV=dev IMAGE_TAG=dev-latest

# Deploy all services to staging
make deploy-all ENV=staging IMAGE_TAG=abc1234

# Deploy to prod (CI sets IMAGE_TAG automatically)
make deploy-all ENV=prod IMAGE_TAG=$GIT_SHA
```

---

## InfraMagician Job (`build-005`)

| Field | Value |
|-------|-------|
| Job ID | `build-005` |
| Agent | `InfraMagician` |
| Inputs | `frontend_codebase.zip`, `backend_codebase.zip`, `tech_stack` |
| Output | `staging_url` |
| `depends_on` | `["build-003", "build-004"]` |

Steps performed:
1. Build Docker image from codebase zip
2. Push to `gcr.io/mbtq-dev/<project>:<git-sha>`
3. Run `helm upgrade --install` with staging values
4. Wait for rollout: `kubectl rollout status deployment/<name>`
5. Run smoke test against `staging_url`
6. Return `staging_url` as job output

---

## Secrets Management

Sensitive values are **never stored in Helm values files**. All secrets are pulled from
**GCP Secret Manager** at pod startup via the [Secret Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/):

```yaml
volumes:
  - name: secrets
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: mbtq-secrets
```

---

## Related

- [Infrastructure / Ecosystem Architecture](./infrastructure.md)
- [Git Workflow](./git-workflow.md)
- [Environments (dev → prod)](./environments.md)
