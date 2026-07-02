# Package Managers & Runtimes

> Which runtime and package manager each project uses, why, and how to run dev/build/test across the ecosystem.

---

## Quick Reference

| Project | Runtime | Package Manager | Server Pattern | Framework |
|---------|---------|----------------|----------------|-----------|
| **FibonRose** | Node.js 20 | `pnpm` | Next.js App Router | Next.js 15 |
| **PinkSync (360magicians)** | Node.js 20 | `pnpm` | Next.js App Router | Next.js 15 |
| **PinkSync microservices** | Deno 1.x | native (import maps) | Deno HTTP | Fresh |
| **Railway Template** | Node.js 20 | `npm` | Next.js App Router + `/api/health` | Next.js 15 |
| **Municipal DAO** | Node.js 20 | `npm` | WebSocket + Next.js | Next.js 15 |
| **mbtq-dev client** | Node.js 20 | `npm` | Vite SPA | React 18 + Vite |
| **mbtq-dev server** | Node.js 20 | `npm` | Express REST API | TypeScript + Prisma |
| **MBTQUniverse** | Node.js 20 | `npm` | Express API gateway | Vanilla JS / ESM |
| **DeafAuth (360magicians)** | Node.js 20 | `pnpm` | Next.js App Router | Next.js 15 + Firebase |
| **DeafAUTH SDK (MBTQ-dev)** | Node.js 20 | `npm` | Framework-agnostic library | TypeScript |
| **Auto-API (MBTQ-dev)** | Python 3.11 | `pip` | FastAPI | FastAPI |
| **pinksync-starter (MBTQ-dev)** | Python 3.11 | `pip` | FastAPI / Flask | Python |

---

## pnpm Projects

**Used by:** FibonRose, PinkSync (360magicians org), DeafAuth (360magicians org)

pnpm is preferred for Next.js/TypeScript projects in the 360magicians org because it:
- Uses hard links for a shared content-addressable store (faster installs, less disk)
- Enforces strict dependency hoisting (avoids phantom dependencies)
- Has first-class monorepo support via workspaces

### Common Commands

```bash
# Install
pnpm install

# Dev server
pnpm dev

# Production build
pnpm build

# Start built server
pnpm start

# Lint
pnpm lint

# Type check
pnpm type-check
```

### FibonRose — Getting Started

```bash
git clone https://github.com/360magicians/fibonrose
cd fibonrose
git checkout ux/ui
pnpm install
cp .env.example .env.local   # fill in NEXT_PUBLIC_* vars
pnpm dev
# → http://localhost:3000
```

---

## npm Projects

**Used by:** Railway Template, Municipal DAO, mbtq-dev, MBTQUniverse, DeafAUTH SDK

npm is used in projects where Railway auto-detection, simplicity, or contributor accessibility is the priority.

### Railway Template — Getting Started

```bash
git clone https://github.com/360magicians/railway_nextjs_with_shadcn
cd railway_nextjs_with_shadcn
npm install
cp .env.example .env.local
npm run dev
# → http://localhost:3000
# Demo: demo@example.com / password123
```

### Municipal DAO — Getting Started

```bash
git clone https://github.com/360magicians/municipal-dao
cd municipal-dao
npm install

# Terminal 1 — WebSocket server
npm run ws-server     # → ws://localhost:8080

# Terminal 2 — Next.js app
npm run dev           # → http://localhost:3000
```

### mbtq-dev — Getting Started (client + server)

```bash
git clone https://github.com/pinkycollie/mbtq-dev
cd mbtq-dev

# Client (React + Vite)
cd client
npm install
npm run dev           # → http://localhost:5173

# Server (Express + Prisma)
cd ../server
npm install
npx prisma migrate dev
npm run dev           # → http://localhost:4000
```

#### Server Structure (`/server`)

```
server/
├── routes/
│   ├── auth.ts          # DeafAUTH integration routes
│   ├── ideas.ts         # IDEA lifecycle endpoints
│   ├── builds.ts        # BUILD lifecycle endpoints
│   └── content.ts       # Content fulfillment API
├── models/
│   └── schema.prisma    # Prisma ORM schema
├── hooks/               # Business logic (service layer)
│   ├── useIdea.ts
│   ├── useBuild.ts
│   └── useContent.ts
├── middleware/
│   ├── auth.ts          # DeafAUTH JWT validation
│   └── accessibility.ts # Accessibility headers
└── index.ts             # Express entry point (PORT=4000)
```

#### Server API Targets & Outputs

| Route | Method | Input | Output |
|-------|--------|-------|--------|
| `/api/health` | GET | — | `{ status: "ok" }` |
| `/api/ideas` | POST | `{ prompt }` | `idea.json` |
| `/api/builds` | POST | `{ idea_id, tech_stack }` | `{ job_id, branch_url }` |
| `/api/content` | POST | `{ creator_id, request }` | `{ fulfillment_id }` |
| `/api/auth/login` | POST | `{ email, password }` | DeafAUTH JWT |

---

## Deno Projects

**Used by:** PinkSync microservices (`PinkSync/PinkSync`)

Deno provides:
- Native TypeScript execution (no build step)
- Secure by default (explicit permissions)
- Built-in test runner and formatter
- URL-based imports (no `node_modules`)

### PinkSync Deno Services

```bash
# No install step — Deno fetches deps on first run

# Run event-orchestrator
deno run --allow-net --allow-env services/event-orchestrator/main.ts

# Run asl-glosser
deno run --allow-net --allow-read services/asl-glosser/main.ts

# Run tests
deno test --allow-net services/

# Format
deno fmt

# Lint
deno lint
```

### Import Map (Deno)

```json
{
  "imports": {
    "std/": "https://deno.land/std@0.208.0/",
    "oak": "https://deno.land/x/oak@v12.6.1/mod.ts",
    "djwt/": "https://deno.land/x/djwt@v3.0.2/"
  }
}
```

---

## Deno Fresh

**Used by:** PinkSync API broker (lightweight routes)

[Fresh](https://fresh.deno.dev) is a Deno-native full-stack web framework with:
- Islands architecture (minimal JS to browser)
- File-based routing
- Zero build step
- No virtual DOM

### Fresh Getting Started

```bash
# Requires Deno installed
deno run -A -r https://fresh.deno.dev pinksync-fresh
cd pinksync-fresh
deno task start    # → http://localhost:8000
```

### Fresh Project Layout

```
pinksync-fresh/
├── routes/
│   ├── index.tsx          # Home page (server-rendered)
│   ├── v1/
│   │   ├── events.ts      # POST /v1/events
│   │   ├── capabilities.ts
│   │   └── subscribe.ts
├── islands/
│   └── AccessibilityWidget.tsx   # Client-interactive component
├── static/
│   └── styles.css
└── deno.json
```

---

## Python Projects

**Used by:** Auto-API (`MBTQ-dev/Auto-API`), pinksync-starter (`MBTQ-dev/pinksync-starter`, `PinkSync/Core-principles`)

```bash
# Bootstrap
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Run FastAPI dev server
uvicorn main:app --reload --port 8000
```

---

## HTML / Static

**Used by:** PinkSync browser extension, landing pages, `public/` directories

The PinkSync browser extension is built as plain HTML + vanilla JS (no bundler):

```
browser-extension/
├── manifest.json
├── popup.html           # Extension popup UI
├── popup.js             # Vanilla JS — no framework
├── content.js           # Injected content script
└── icons/
```

Build (pack for Chrome Web Store):
```bash
cd browser-extension
zip -r extension.zip . -x "*.DS_Store"
```

---

## AI SDKs & Inference Clients

| Package | Classification | Runtime | Projects |
|---------|---------------|---------|---------|
| **`ai` (Vercel AI SDK)** | Open Source (Apache 2.0) | Node.js / Edge | Next.js projects, streaming chat routes |
| **`@ai-sdk/openai`** | Open Source (Apache 2.0) | Node.js / Edge | GPT-4, GPT-4o (AI SDK provider) |
| **`@ai-sdk/anthropic`** | Open Source (Apache 2.0) | Node.js / Edge | Claude (AI SDK provider) |
| **`@ai-sdk/google`** | Open Source (Apache 2.0) | Node.js / Edge | Gemini (AI SDK provider) |
| **`@google/generative-ai`** | Open Source (Apache 2.0) | Node.js / Deno | Gemini direct SDK |
| **`openai`** | Open Source (Apache 2.0) | Node.js | OpenAI + vLLM-compatible endpoints |
| **`@anthropic-ai/sdk`** | Open Source (MIT) | Node.js | Claude SDK — BUILD stage agents |
| **`@huggingface/inference`** | Open Source (Apache 2.0) | Node.js / Deno | HF Hub models (Llama 3, Mistral, CLIP, Whisper) |
| **`groq-sdk`** | Open Source (Apache 2.0) | Node.js | Groq LPU fast inference |
| **`@fal-ai/client`** | Open Source (MIT) | Node.js / Deno | Image/video generation (SDXL, FLUX.1) |
| **`langchain`** | Open Source (MIT) | Node.js | Orchestration, RAG chains |
| **`@orpc/server`** | Open Source (MIT) | Node.js / Deno | Type-safe RPC server wrapping all AI SDKs |
| **`@orpc/client`** | Open Source (MIT) | Node.js / Browser | Type-safe RPC client |

See [AI Model Manifest](./ai-model-manifest.md) for per-model SDK assignments, and [AI Inference Architecture](./ai-inference.md) for routing and fallback chains.

---

## Consistency Rules

To keep the ecosystem consistent:

1. **360magicians org** projects → use `pnpm`
2. **pinkycollie / MBTQ-dev** projects → use `npm`
3. **PinkSync microservices** → use Deno (no node_modules)
4. **All `.env` files** → must be gitignored; use `.env.example` as template
5. **All Node.js projects** → pin to Node.js 20 LTS (specify in `.nvmrc` or `engines` in `package.json`)
6. **All pnpm projects** → include `pnpm-lock.yaml` in git; never commit `package-lock.json`
7. **All npm projects** → include `package-lock.json` in git; never commit `pnpm-lock.yaml`

---

## Related

- [Environments (dev → prod)](./environments.md)
- [Providers, Vendors & Resources](./providers.md)
- [Git Workflow](./git-workflow.md)
