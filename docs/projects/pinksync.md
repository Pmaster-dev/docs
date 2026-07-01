# PinkSync

Source: [`360magicians/pinksync`](https://github.com/360magicians/pinksync)

> Accessibility layer + real-time sync engine for the MBTQ ecosystem.

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

## Related

- [Infrastructure Docs](../infrastructure.md)
- [DeafAuth (identity layer)](./deafauth.md)
