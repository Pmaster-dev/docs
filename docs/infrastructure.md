# MBTQ Deaf-First AI Ecosystem Architecture

Source: [`360magicians/municipal-dao`](https://github.com/360magicians/municipal-dao) — `ecosystem-architecture/mbtq-ecosystem.yaml`

---

## Platform Overview

The ecosystem runs on **GCP project: mbtq.dev** and consists of five interconnected platforms:

| Platform | Domain | Role |
|----------|--------|------|
| **360magicians.com** | Vertex AI Multi-Domain Hub | Creative, design, content, and accessibility AI endpoints |
| **pinksync.io** | Accessibility AI Middleware | AI transformation, offline sync, cross-platform delivery |
| **mbtq.dev** | Deaf-First Platform | v0 Studio + Gemini CLI integration, Deaf-first UI dashboard |
| **mbtquniverse.com** | DAO + Web3 + Staking | On-chain governance, staking, community treasury |
| **DeafAUTH** | Universal Identity Layer | Cross-ecosystem auth, accessibility preferences, AI personalization |

---

## Service Map

### 360magicians.com — Vertex AI Multi-Domain Hub

Endpoints:
- `creative-ai.360magicians.com`
- `design-ai.360magicians.com`
- `content-ai.360magicians.com`
- `accessibility-ai.360magicians.com`

Kubernetes: 5 replicas · `gcr.io/mbtq-dev/360magicians-vertex-ai:latest` · port 8080  
Environment: `VERTEX_AI_PROJECT=mbtq.dev`, `VERTEX_AI_LOCATION=us-central1`, `ACCESSIBILITY_FIRST=true`

---

### pinksync.io — Accessibility AI Middleware

Services:
- `ai-transformation`
- `accessibility-layer`
- `data-synchronization`
- `offline-sync`
- `cross-platform-delivery`

Kubernetes: 3 replicas · `gcr.io/mbtq-dev/pinksync-accessibility:latest` · port 8081  
Accessibility transforms: `visual, tactile, sign-language, high-contrast`

---

### mbtq.dev — Deaf-First Platform

Services:
- `v0-studio-integration`
- `gemini-cli-bridge`
- `deaf-first-ui-dashboard`
- `ai-google-studio-proxy`

Kubernetes: 4 replicas · `gcr.io/mbtq-dev/mbtq-platform:latest` · port 3000

---

### DeafAUTH — Universal Identity Layer

Services:
- `cross-ecosystem-auth`
- `accessibility-preferences`
- `ai-personalization`
- `community-verification`

See also: [DeafAuth Project Docs](./projects/deafauth.md)

---

## API Reference

Base URL: `https://api.360magicians.com`

Authentication via DeafAUTH bearer token:

```bash
export DEAFAUTH_TOKEN=<YOUR_ACCESS_TOKEN>
curl -H "Authorization: ******" \
     https://api.360magicians.com/jobs/list
```

User profile example:

```json
{
  "user_id": "deaf123",
  "auth_type": "web3_wallet",
  "permissions": ["IDEA_GENERATOR", "BUILD_ACCESS", "GROW_DASHBOARD"],
  "profile": {
    "display_name": "PinkMagician",
    "language": "ASL",
    "privacy": "private"
  }
}
```

---

## References

- [DeafAUTH Docs](https://github.com/deafauth/docs)
- [360Magicians API](https://github.com/pmaster-dev/docs)
- [LifecycleBlueprint JSON Schema](https://github.com/mbtq&dev/docs/lifecycleblueprint)
- [AI Model Manifest](./ai-model-manifest.md)
- [AI Inference Architecture](./ai-inference.md)
