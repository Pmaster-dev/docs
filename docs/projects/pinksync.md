# PinkSync

Sources: [`360magicians/pinksync`](https://github.com/360magicians/pinksync) · [`PinkSync/PinkSync`](https://github.com/PinkSync/PinkSync) · [`PinkSync/Core-principles`](https://github.com/PinkSync/Core-principles)

> Accessibility layer + real-time sync engine · Layer 1 accessibility orchestration platform · Deaf-first API broker.

---

## Overview

PinkSync is the **Accessibility AI Middleware** of the 360Magicians ecosystem. It bridges AI services
with accessible delivery across visual, tactile, sign-language, and high-contrast modalities.

**Core services:**
- `ai-transformation` — Transform AI outputs into accessible formats
- `accessibility-layer` — Apply modality-specific transformations
- `data-synchronization` — Keep data consistent across devices
- `offline-sync` — Queue and replay updates when offline
- `cross-platform-delivery` — Deliver content to web, mobile, and VR

---

## Infrastructure

Kubernetes: 3 replicas · `gcr.io/mbtq-dev/pinksync-accessibility:latest` · port 8081

Environment variables:
```
DEAFAUTH_API_URL=https://deafauth.mbtq.dev
VERTEX_AI_ENDPOINT=https://api.360magicians.com
OFFLINE_SYNC_ENABLED=true
ACCESSIBILITY_TRANSFORMS=visual,tactile,sign-language,high-contrast
```

---

## Tech Stack

Built with Next.js (TypeScript), shadcn/ui, Tailwind CSS.
- `app/` — Next.js App Router pages
- `backend/` — Backend services
- `browser-extension/` — Browser extension for accessibility overlays
- `components/` — Shared UI components
- `contexts/` — React context providers
- `hooks/` — Custom hooks

---

## PinkSync Platform (PinkSync/PinkSync)

The [PinkSync/PinkSync](https://github.com/PinkSync/PinkSync) repository is the comprehensive Layer 1 platform. It acts as a unified gateway connecting Deaf communities with accessible services while providing real-time accessibility enhancements.

### Two Primary Components

**DeafAUTH Backend (The Brain)**
- Centralized authentication and preference management
- API gateway for service integrations
- Real-time content transformation engine

**PinkSync Browser Extension (The Hands)**
- Chrome extension that runs on every website
- Automatically applies accessibility preferences
- Enables captions on all video platforms
- Converts audio alerts to visual notifications
- Auto-fills accessibility forms

### Microservices

| Service | Purpose |
|---------|---------|
| `deafauth` | Visual-first authentication and preference management |
| `event-orchestrator` | Node-based event listener and processing |
| `rag-engine` | Research-Augmented Generation for accessibility learning |
| `workers` | Background job processors for automation |
| `api-broker` | Unified gateway for partner service integrations |
| `pinkflow` | Real-time accessibility adjustment engine |
| `asl-glosser` | ASL glossing and sign language processing |
| `sign-speak` | Multi-language sign language recognition |
| `vcode` | Deaf-first video communication |
| `interpreters` | Interpreter booking and management |

> Powered by Deno for lightweight runtime — no build step, native TypeScript support.

---

## PinkSync API Broker (PinkSync/Core-principles)

The [PinkSync/Core-principles](https://github.com/PinkSync/Core-principles) repository defines PinkSync as an **accessibility API broker** — "Stripe, but for accessibility signals."

### Core Philosophy

> **Not a Deaf app. A Deaf-first protocol.**

PinkSync operates as an **accessibility signal exchange**:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/events` | POST | Accept accessibility events from applications |
| `/v1/capabilities` | GET | List registered application capabilities |
| `/v1/subscribe` | POST | Subscribe to accessibility events |
| `/v1/compliance/{app_id}` | GET | Check compliance status |
| `/v1/validate` | POST | Validate target for compliance |
| `/v1/context/initialize` | POST | Initialize accessibility context |

### Compliance Levels

Bronze → Silver → Gold → Platinum — each tier enforces more stringent contract requirements with machine-verifiable signals and cryptographically signed audit logs.

```python
# Submit accessibility event
event = {
    "app_id": "my-app-v2",
    "user_id": "user-12345",
    "intent": "visual_only",
    "compliance_level": "gold"
}
response = httpx.post("https://api.pinksync.io/v1/events", json=event)
```

---

## Related

- [Infrastructure Docs](../infrastructure.md)
- [DeafAuth (identity layer)](./deafauth.md)
