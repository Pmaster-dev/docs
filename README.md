# Magicians VR4Deaf Platform — API Documentation

OpenAPI definition and aggregated documentation for the **Magicians VR4Deaf Platform** internal and public API, rendered with [Redocly](https://redocly.com/).

## Project Docs

See the [`docs/`](./docs/README.md) directory for full project documentation:

### Ecosystem Guides
- [About 360Magicians](./docs/about.md)
- [Infrastructure / Ecosystem Architecture](./docs/infrastructure.md)
- [Git Workflow & GitHub Integration](./docs/git-workflow.md)
- [Helm Deployment (dev → prod)](./docs/helm.md)
- [Environments & Bootstrap](./docs/environments.md)
- [AI Providers, Vendors & Resources](./docs/providers.md)
- [AI Model Manifest](./docs/ai-model-manifest.md)
- [AI Inference Architecture](./docs/ai-inference.md)
- [Package Managers & Runtimes](./docs/package-managers.md)
- [Triggers, Webhooks & Prompts](./docs/triggers-webhooks.md)
- [Pipeline Handoffs](./docs/pipeline-handoffs.md)
- [Copilot, Bots & Auth Access](./docs/copilot-bots.md)

### Project Docs
- [DeafAuth](./docs/projects/deafauth.md)
- [PinkSync](./docs/projects/pinksync.md)
- [FibonRose](./docs/projects/fibonrose.md)
- [Municipal DAO](./docs/projects/municipal-dao.md)
- [Railway Next.js Template](./docs/projects/railway-template.md)
- [MBTQ.dev](./docs/projects/mbtq-dev.md)
- [MBTQUniverse](./docs/projects/mbtquniverse.md)

## Base URLs

| Environment | URL |
|-------------|-----|
| Public API gateway | `https://magicians.vr4deaf.org/api/v1` |
| Internal (service-to-service, cluster only) | `https://internal.magicians.vr4deaf.org/api/v1` |

## OpenAPI Definition

The `openapi/` directory contains the OpenAPI 3.1 definition for the **Magicians VR4Deaf Platform** API.

## Authentication

Services use short-lived `X-Service-Token` JWTs (service-to-service).
Human administrators use OAuth2 implicit flow against
`https://magicians.vr4deaf.org/api/oauth/dialog`.

Long-lived `api_key` header tokens are also supported for automated
integrations.

## Development

### Prerequisites

- [Node.js](https://nodejs.org/) ≥ 18

### Install

```bash
npm install
```

### Preview docs locally

```bash
npm start
```

Opens a live-reloading Redoc preview at `http://localhost:8080`.

### Validate the spec

```bash
npm test
```

Runs `redocly lint` against all API definitions.
