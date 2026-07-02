# 360 Magicians — Documentation

This directory contains aggregated documentation for the [360Magicians](https://github.com/360magicians)
Deaf-First AI ecosystem and all associated repositories across the MBTQ universe.

---

## Contents

### Ecosystem Guides

| File | Description |
|------|-------------|
| [about.md](./about.md) | Organization overview, mission, and project list |
| [infrastructure.md](./infrastructure.md) | MBTQ ecosystem architecture (GCP / Kubernetes) |
| [git-workflow.md](./git-workflow.md) | Branch strategy, agent git flow, GitHub Graph API, CI/CD events |
| [helm.md](./helm.md) | Helm charts, deployment targets, env overrides (dev → prod) |
| [environments.md](./environments.md) | Dev → staging → prod separation, bootstrap steps, env vars |
| [providers.md](./providers.md) | AI providers, platform vendors, open-source vs proprietary vs custom |
| [package-managers.md](./package-managers.md) | pnpm / npm / Deno / Fresh / HTML — which project uses what |

### Project Docs

| File | Description |
|------|-------------|
| [projects/deafauth.md](./projects/deafauth.md) | DeafAuth — identity & access layer + Magician Jobs API |
| [projects/pinksync.md](./projects/pinksync.md) | PinkSync — accessibility AI middleware + API broker |
| [projects/fibonrose.md](./projects/fibonrose.md) | FibonRose — Next.js UI application (pnpm, shadcn/ui) |
| [projects/municipal-dao.md](./projects/municipal-dao.md) | Municipal DAO — real-time WebSocket governance platform |
| [projects/railway-template.md](./projects/railway-template.md) | Railway Next.js template — production-ready starter (npm) |
| [projects/mbtq-dev.md](./projects/mbtq-dev.md) | MBTQ.dev — AI-powered full-stack development platform |
| [projects/mbtquniverse.md](./projects/mbtquniverse.md) | MBTQUniverse — DAO governance, tokenization, staking |

---

## Repositories Covered

### 360Magicians Org

| Repo | Status | Language |
|------|--------|----------|
| [360magicians/about](https://github.com/360magicians/about) | Active | — |
| [360magicians/infrastructure](https://github.com/360magicians/infrastructure) | Active | — |
| [360magicians/deafauth](https://github.com/360magicians/deafauth) | Active | JavaScript |
| [360magicians/pinksync](https://github.com/360magicians/pinksync) | Active | TypeScript |
| [360magicians/fibonrose](https://github.com/360magicians/fibonrose) | Active | TypeScript |
| [360magicians/municipal-dao](https://github.com/360magicians/municipal-dao) | Active | TypeScript |
| [360magicians/railway_nextjs_with_shadcn](https://github.com/360magicians/railway_nextjs_with_shadcn) | Active | TypeScript |

### pinkycollie

| Repo | Status | Language |
|------|--------|----------|
| [pinkycollie/mbtq-dev](https://github.com/pinkycollie/mbtq-dev) | Active | TypeScript |
| [pinkycollie/mbtquniverse](https://github.com/pinkycollie/mbtquniverse) | Active | JavaScript |
| [pinkycollie/deaf-first-platform](https://github.com/pinkycollie/deaf-first-platform) | Active | HCL |
| [pinkycollie/Nextjs-DeafAUTH](https://github.com/pinkycollie/Nextjs-DeafAUTH) | Active | TypeScript |
| [pinkycollie/NegraRosa](https://github.com/pinkycollie/NegraRosa) | Active | TypeScript |
| [pinkycollie/VR4Deaf](https://github.com/pinkycollie/VR4Deaf) | Active | TypeScript |
| [pinkycollie/ai-magicians-gcp](https://github.com/pinkycollie/ai-magicians-gcp) | Active | TypeScript |
| [pinkycollie/FibonRoseTrust](https://github.com/pinkycollie/FibonRoseTrust) | Active | TypeScript |
| [pinkycollie/PinkSync-Visualizer](https://github.com/pinkycollie/PinkSync-Visualizer) | Active | TypeScript |
| [pinkycollie/Deaf-Creators-Platform](https://github.com/pinkycollie/Deaf-Creators-Platform) | Active | TypeScript |

### MBTQ-dev Org

| Repo | Status | Language |
|------|--------|----------|
| [MBTQ-dev/DeafAUTH](https://github.com/MBTQ-dev/DeafAUTH) | Active | TypeScript |
| [MBTQ-dev/Magician_Platform](https://github.com/MBTQ-dev/Magician_Platform) | Active | TypeScript |
| [MBTQ-dev/DEAF-FIRST-PLATFORM2](https://github.com/MBTQ-dev/DEAF-FIRST-PLATFORM2) | Active | HCL |
| [MBTQ-dev/deploy-admin](https://github.com/MBTQ-dev/deploy-admin) | Active | TypeScript |
| [MBTQ-dev/Auto-API](https://github.com/MBTQ-dev/Auto-API) | Active | Python |
| [MBTQ-dev/pinksync-starter](https://github.com/MBTQ-dev/pinksync-starter) | Active | Python |

### DeafAUTH Org

| Repo | Status | Language |
|------|--------|----------|
| [DeafAUTH/readme.md](https://github.com/DeafAUTH/readme.md) | Active | — |

### PinkSync Org

| Repo | Status | Language |
|------|--------|----------|
| [PinkSync/PinkSync](https://github.com/PinkSync/PinkSync) | Active | TypeScript |
| [PinkSync/Core-principles](https://github.com/PinkSync/Core-principles) | Active | Python |
| [PinkSync/pinksync-starter](https://github.com/PinkSync/pinksync-starter) | Active | — |
| [PinkSync/awesome-deaf-first](https://github.com/PinkSync/awesome-deaf-first) | Active | — |
| [PinkSync/manifesto](https://github.com/PinkSync/manifesto) | Active | — |

---

## API Quick Reference

Base URL: `https://api.360magicians.com`

```bash
# Authenticate via DeafAUTH
export DEAFAUTH_TOKEN=<YOUR_ACCESS_TOKEN>

# List jobs
curl -H "Authorization: ******" https://api.360magicians.com/jobs/list

# Submit idea
curl -X POST https://api.360magicians.com/idea \
  -H "Authorization: ******" \
  -d '{"prompt": "Deaf-first SaaS idea"}'
```

Full API reference: [DeafAuth Docs](./projects/deafauth.md)

