# Magicians VR4Deaf Platform — API Documentation

OpenAPI 3.1 specification for the **Magicians VR4Deaf Platform** internal and
public API, rendered with [Redocly](https://redocly.com/).

## Base URLs

| Environment | URL |
|-------------|-----|
| Public API gateway | `https://magicians.vr4deaf.org/api/v1` |
| Internal (service-to-service, cluster only) | `https://internal.magicians.vr4deaf.org/api/v1` |

## API Sections

| Tag | Description |
|-----|-------------|
| **User** | User account management |
| **Admin** | Administrator-only operations |
| **Info** | Platform information and health |
| **Network** | Internal network topology and node status |
| **Registry** | Internal-only npm/Docker/Helm package registry |
| **Vendors** | Third-party vendor integrations |
| **Rate** | Rate limit policy management |
| **Rules** | Business rules engine (JSON Logic DSL) |
| **NPX** | Ephemeral Node package execution jobs |
| **Ingress** | Ingress routing table and path rewrites |
| **Database** | Database proxy layer — schema introspection |

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

## Contributing

### File layout

```
openapi/
├── openapi.yaml              # Root spec (info, servers, paths index, webhooks)
├── paths/                    # One YAML file per route
│   ├── network.yaml
│   ├── registry.yaml
│   ├── vendors.yaml
│   ├── rate.yaml
│   ├── rules.yaml
│   ├── npx.yaml
│   ├── ingress.yaml
│   ├── db.yaml
│   └── ...
└── components/
    ├── schemas/              # Reusable schema objects
    ├── headers/              # Reusable response headers
    └── responses/            # Reusable response objects
```

### Adding a new path

1. Create `openapi/paths/<your-path>.yaml` (replace `/` with `_`).
2. Add a `$ref` entry in `openapi/openapi.yaml` under `paths:`.
3. Run `npm test` to validate.

### Adding a new schema

1. Create `openapi/components/schemas/<SchemaName>.yaml`.
2. Reference it in your path file with `$ref: '../components/schemas/<SchemaName>.yaml'`.

### Redocly configuration

`redocly.yaml` controls linting rules and code-sample generation.
See the [Redocly CLI docs](https://redocly.com/docs/cli/configuration/) for
all available options.

