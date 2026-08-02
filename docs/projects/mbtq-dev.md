# MBTQ.dev

Source: [`pinkycollie/mbtq-dev`](https://github.com/pinkycollie/mbtq-dev)

> AI-powered full-stack development platform — accessible starter kit for LGBTQ+ and Deaf communities.

---

## Overview

**MBTQ.dev** is a transparent generative AI development starter that powers full-stack applications with modern tools for visual design creation and communication. Originally built to support Deaf entrepreneurs, it now serves as an educational and starter kit platform.

Live Demo: [pinkycollie.github.io/mbtq-dev](https://pinkycollie.github.io/mbtq-dev/)

---

## Key Features

- **Generative AI Integration** — Templates integrating GPT-4, Claude, and Gemini
- **Backend Connectors** — Supabase auth, database, storage, and edge functions
- **Accessibility First** — WCAG compliant, visual notifications, caption widget, high-contrast toggle
- **Movable/Resizable Widgets** — Interact.js drag-and-drop workspace
- **Real-time Multiuser Sync** — Socket.IO collaboration
- **Quinn AI Assistant** — AI agent with Fibonrose task validation
- **Content Fulfillment API** — Full-stack API for video requests and creator fulfillment

---

## AI Development Assistant (Quinn)

Quinn is an AI agent powered by the **Fibonrose Task Validation System**:

- Code architecture guidance
- Feature implementation assistance
- Debugging help
- PR review and suggestions
- Documentation generation

### Fibonrose Task Validation

Tasks are validated with Fibonacci-based checkpoints scaled by complexity:

| Complexity | Confirmations | Example |
|-----------|---------------|---------|
| 0–1 | 1 | Fix typo, update docs |
| 2 | 2 | Add component prop, update config |
| 3 | 3 | Create UI component, add API endpoint |
| 4 | 5 | Implement feature with tests |
| 5 | 8 | Build complete feature |
| 6+ | 13+ | Major architectural changes |

---

## Tech Stack

| Technology | Purpose |
|-----------|---------|
| React 18 + TypeScript | Frontend UI |
| Vite | Build tooling |
| Tailwind CSS | Styling |
| Socket.IO | Real-time collaboration |
| PostgreSQL + Prisma ORM | Database |
| Supabase | Auth, storage, edge functions |
| axe-core | Accessibility testing |

---

## Architecture

```
mbtq-dev/
├── client/      # React UI with drag-and-drop widgets
├── server/      # TypeScript REST API
│   ├── routes/  # API routes
│   ├── models/  # Prisma models
│   └── hooks/   # Business logic
└── docs/        # Guides and references
```

---

## Getting Started

```bash
# Client
cd client && npm install && npm run dev

# Server
cd server && npm install && npm run dev
```

---

## Related

- [PinkSync (accessibility layer)](./pinksync.md)
- [DeafAuth (identity layer)](./deafauth.md)
- [Infrastructure Docs](../infrastructure.md)
