# 360 Magicians — Documentation

This directory contains aggregated documentation for the [360Magicians](https://github.com/360magicians)
Deaf-First AI ecosystem and all associated repositories.

---

## Contents

| File | Description |
|------|-------------|
| [about.md](./about.md) | Organization overview, mission, and project list |
| [infrastructure.md](./infrastructure.md) | MBTQ ecosystem architecture (GCP / Kubernetes) |
| [projects/deafauth.md](./projects/deafauth.md) | DeafAuth — identity & access layer + Magician Jobs API |
| [projects/pinksync.md](./projects/pinksync.md) | PinkSync — accessibility AI middleware |
| [projects/fibonrose.md](./projects/fibonrose.md) | FibonRose — Next.js UI application |
| [projects/municipal-dao.md](./projects/municipal-dao.md) | Municipal DAO — real-time WebSocket governance platform |
| [projects/railway-template.md](./projects/railway-template.md) | Railway Next.js template — production-ready starter |

---

## Repositories Covered

| Repo | Status | Language |
|------|--------|----------|
| [360magicians/about](https://github.com/360magicians/about) | Active | — |
| [360magicians/infrastructure](https://github.com/360magicians/infrastructure) | Active | — |
| [360magicians/deafauth](https://github.com/360magicians/deafauth) | Active | JavaScript |
| [360magicians/pinksync](https://github.com/360magicians/pinksync) | Active | TypeScript |
| [360magicians/fibonrose](https://github.com/360magicians/fibonrose) | Active | TypeScript |
| [360magicians/municipal-dao](https://github.com/360magicians/municipal-dao) | Active | TypeScript |
| [360magicians/railway_nextjs_with_shadcn](https://github.com/360magicians/railway_nextjs_with_shadcn) | Active | TypeScript |
| [360magicians/business](https://github.com/360magicians/business) | Empty | — |
| [360magicians/VURI](https://github.com/360magicians/VURI) | Empty | — |
| [360magicians/job](https://github.com/360magicians/job) | Empty | — |

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
