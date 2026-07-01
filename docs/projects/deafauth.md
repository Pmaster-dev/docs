# DeafAuth

Source: [`360magicians/deafauth`](https://github.com/360magicians/deafauth) · [`360magicians/infrastructure`](https://github.com/360magicians/infrastructure)

> Vertical Agent — Web2/Web3 authentication & verification for Deaf-first platforms.

---

## Overview

**DeafAUTH** is the universal identity and access layer for the 360Magicians ecosystem.

- Web2 & Web3 login (wallet-based)
- Role-based access control
- Magician agent role assignment
- API token security

---

## Authentication

### ******

```bash
export DEAFAUTH_TOKEN=<YOUR_ACCESS_TOKEN>
curl -H "Authorization: ******" \
     https://api.360magicians.com/jobs/list
```

### User Profile Schema

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

## Magician Jobs (Lifecycle API)

### IDEA Stage

| Job ID    | Assigned To         | Inputs                          | Outputs                    |
|-----------|---------------------|---------------------------------|----------------------------|
| `idea-001`| IdeaMagician        | `prompt`                        | `idea.json`                |
| `idea-002`| ValidatorMagician   | `idea.json`                     | `validation_report.json`   |
| `idea-003`| ResearchMagician    | `topic`, `target_audience`      | `market_research_report.md`|

**Generate idea:**
```bash
curl -X POST https://api.360magicians.com/idea \
  -H "Authorization: ******" \
  -d '{"prompt": "Deaf-first SaaS idea"}'
```

---

### BUILD Stage

| Job ID     | Assigned To        | Inputs                                  | Outputs                  |
|------------|--------------------|-----------------------------------------|--------------------------|
| `build-001`| DeveloperMagician  | `project_name`, `project_description`  | `repository_url`         |
| `build-002`| DesignMagician     | `idea.json`                             | `wireframes.svg`         |
| `build-003`| DeveloperMagician  | `wireframes.svg`, `tech_stack`          | `frontend_codebase.zip`  |
| `build-004`| DeveloperMagician  | `api_specs.json`, `tech_stack`          | `backend_codebase.zip`   |
| `build-005`| InfraMagician      | `frontend_codebase.zip`, `backend_codebase.zip` | `staging_url`  |

---

### GROW Stage

| Job ID     | Assigned To           | Inputs               | Outputs                  |
|------------|-----------------------|----------------------|--------------------------|
| `grow-001` | DataMagician          | `app_url`, `kpis`    | `dashboard_url`          |
| `grow-002` | GrowthMagician        | `campaign_brief.md`  | `campaign_report.md`     |
| `grow-003` | PartnershipMagician   | `partner_list.csv`   | `onboarding_status.json` |

---

### MANAGED Stage

| Job ID        | Assigned To         | Inputs               | Outputs                       |
|---------------|---------------------|----------------------|-------------------------------|
| `managed-001` | ComplianceMagician  | `reporting_period`   | `compliance_report.pdf`       |
| `managed-002` | FinanceMagician     | `transactions.csv`   | `financial_report.pdf`        |
| `managed-003` | OpsMagician         | `app_url`            | `monitoring_dashboard_url`    |

---

## Job Dependencies

Jobs support sequential orchestration via `depends_on`:

```json
{
  "job_id": "build-003",
  "depends_on": ["build-002"]
}
```

This ensures **frontend development waits for wireframes**.

---

## Tech Stack

Built with Next.js (TypeScript), shadcn/ui, Tailwind CSS, Firebase.

Key source files:
- `src_firebase_Version2.js` — Firebase config
- `src_components_Auth_Version2.js` — Auth UI components
- `src_hooks_useAccommodations_Version2.js` — Accessibility hooks
- `functions_index_Version2.js` — Cloud Functions entry point
