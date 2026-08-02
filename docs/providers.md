# AI Providers, Vendors & Resources

> Reference for all AI providers, platform vendors, and resource types used across the 360Magicians / MBTQ ecosystem — classified as open-source, proprietary, or custom-built.

---

## AI Providers

| Provider | Model(s) | Classification | Used By |
|----------|---------|----------------|---------|
| **Google Vertex AI** | Gemini, PaLM | Proprietary | 360magicians.com hub, mbtq.dev |
| **Google AI Studio** | Gemini | Proprietary | mbtq.dev (Gemini CLI bridge) |
| **OpenAI** | GPT-4, GPT-4o | Proprietary | mbtq-dev templates, Quinn AI |
| **Anthropic** | Claude (all versions) | Proprietary | Quinn AI, agent review |
| **Grok** | Grok | Proprietary | Ecosystem partner |
| **Groq** | Mixtral, Llama 3 | Proprietary (inference) | Fast inference layer |
| **Fal** | Image / video models | Proprietary | Content generation, Deaf media |
| **HuggingFace** | Llama 3, Mistral, Whisper, CLIP, SDXL, embeddings | Open Source (HF Hub) | PinkSync, DeafAuth, local dev, fallback inference |

### Provider Selection by Stage

| Stage | Preferred Provider | Reason |
|-------|--------------------|--------|
| IDEA | OpenAI / Gemini | Creative generation |
| BUILD | Anthropic (Claude) | Code generation, architecture |
| GROW | OpenAI | Marketing copy, analytics summaries |
| MANAGED | Gemini (Vertex AI) | Compliance, data analysis at scale |

---

## Platform Vendors

### Cloud & Infrastructure

| Vendor | Service | Classification | Used By |
|--------|---------|----------------|---------|
| **Google Cloud (GCP)** | GKE, GCR, Secret Manager, Vertex AI | Proprietary | Full ecosystem (project: `mbtq.dev`) |
| **Railway** | PaaS deployment, health checks | Proprietary | `railway_nextjs_with_shadcn`, quick deploys |
| **Supabase** | Auth, PostgreSQL, storage, edge functions | Open Source (self-hostable) | mbtq-dev, general projects |
| **Firebase** | Auth, Firestore, Cloud Functions | Proprietary | DeafAuth (firebase version) |
| **Neon** | Serverless PostgreSQL | Proprietary | Ecosystem partner |

### Auth & Identity

| Vendor | Service | Classification | Used By |
|--------|---------|----------------|---------|
| **Auth0** | Enterprise OAuth/OIDC | Proprietary | DeafAUTH OAuth wrapper integration |
| **DeafAUTH** | Deaf-first identity compiler | **Custom** | All ecosystem services |
| **Web3 Wallets** | Wallet-based auth (EVM) | Open Protocol | MBTQUniverse, DAO governance |

### Developer Tools

| Vendor | Service | Classification | Used By |
|--------|---------|----------------|---------|
| **v0 (Vercel)** | AI UI generation | Proprietary | mbtq.dev (v0-studio-integration) |
| **Cursor AI** | AI code editor | Proprietary | Developer tooling |
| **GitHub** | Repos, Actions, GraphQL API | Proprietary (free tier) | All repos |
| **Redocly** | API docs / OpenAPI rendering | Proprietary (free tier) | This docs repo |

---

## Open-Source Dependencies

### Frameworks & Runtimes

| Package | Version | Runtime | Projects |
|---------|---------|---------|---------|
| **Next.js** | 15.x | Node.js | FibonRose, DeafAuth, PinkSync, Railway template |
| **React** | 19.x | Browser / Node.js | All Next.js projects |
| **Deno** | 1.x | Deno runtime | PinkSync microservices |
| **Fresh** | 1.x | Deno | PinkSync (lightweight Deno web framework) |
| **Vite** | 5.x | Node.js | mbtq-dev client |

### UI & Styling

| Package | Classification | Projects |
|---------|---------------|---------|
| **shadcn/ui** | Open Source (MIT) | FibonRose, Railway template, DeafAuth, PinkSync |
| **Tailwind CSS** | Open Source (MIT) | All Next.js projects |
| **Interact.js** | Open Source (MIT) | mbtq-dev (drag-and-drop widgets) |

### Backend & Data

| Package | Classification | Projects |
|---------|---------------|---------|
| **Prisma ORM** | Open Source (Apache 2.0) | mbtq-dev server |
| **Socket.IO** | Open Source (MIT) | mbtq-dev (real-time collab), PinkSync |
| **PostgreSQL** | Open Source (PostgreSQL License) | mbtq-dev, Supabase projects |
| **axe-core** | Open Source (MPL 2.0) | mbtq-dev (accessibility testing) |

### Validation & Utilities

| Package | Classification | Projects |
|---------|---------------|---------|
| **Zod** | Open Source (MIT) | Railway template, DeafAuth |
| **React Hook Form** | Open Source (MIT) | Railway template |
| **bcryptjs** | Open Source (MIT) | Railway template (password hashing) |

---

## Proprietary & Custom Components

### Custom-Built (360Magicians / MBTQ)

| Component | Repo | Purpose |
|-----------|------|---------|
| **DeafAUTH** | `MBTQ-dev/DeafAUTH` | Universal Deaf-first identity compiler (custom protocol) |
| **PinkSync API Broker** | `PinkSync/Core-principles` | Accessibility signal exchange (custom protocol) |
| **Magician Agents** | `MBTQ-dev/Magician_Platform` | IDEA→BUILD→GROW→MANAGED lifecycle agents |
| **Quinn AI** | `pinkycollie/mbtq-dev` | Dev assistant with Fibonacci task validation |
| **MBTQUniverse DAO** | `pinkycollie/mbtquniverse` | Custom on-chain governance framework |
| **FibonRose Task Validation** | `360magicians/fibonrose` | Fibonacci-scaled confirmation system |

### Proprietary AI Services (vendor-locked)

| Service | Vendor | Alternative (open source) |
|---------|--------|--------------------------|
| Vertex AI | Google | Ollama + open LLMs |
| GPT-4 | OpenAI | Llama 3 (via Groq or local) |
| Claude | Anthropic | Mistral (self-hosted) |
| Firebase Auth | Google | Supabase Auth (open source) |
| v0 | Vercel | Lovable.dev, open UI generators |

---

## Resource Summary by Environment

| Resource Type | Dev | Staging | Prod |
|--------------|-----|---------|------|
| AI provider | Mock / local Ollama | Vertex AI (lower quota) | Vertex AI + OpenAI + Claude |
| Database | Local PostgreSQL or Supabase free | Supabase project | Supabase / Neon paid tier |
| Auth | DeafAUTH dev instance | DeafAUTH staging | DeafAUTH prod |
| K8s | Local (kind / minikube) | GKE staging cluster | GKE prod cluster (`mbtq.dev`) |
| Container registry | Local | `gcr.io/mbtq-dev/*:staging-*` | `gcr.io/mbtq-dev/*:<git-sha>` |
| Secrets | `.env.local` files | GCP Secret Manager (staging) | GCP Secret Manager (prod) |

---

## Related

- [Environments (dev → prod)](./environments.md)
- [Package Managers & Runtimes](./package-managers.md)
- [Infrastructure / Ecosystem Architecture](./infrastructure.md)
