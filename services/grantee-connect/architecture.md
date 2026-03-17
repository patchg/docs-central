# grantee-connect — Architecture

## System Overview
Vue.js SPA + Ruby on Rails API. Public-facing but authentication-gated — only active grantees can log in. Reads from grants_core_db (read replica) and financial_db (read replica). Does not write to shared databases — uses its own `grantee_connect_db` for progress report data.

## Components

### Frontend
- **Framework:** Vue 3, Vite
- **Hosting:** CloudFront + S3 static export
- **Auth:** JWT via auth-service; grantee_contact and grantee_finance roles

### Backend API
- **Runtime:** Ruby 3.2, Rails 7.1
- **Hosting:** AWS Lambda + API Gateway (containerized Lambda)
- **Own Database:** grantee_connect_db (PostgreSQL 14) — progress reports, amendments, report attachments metadata
- **Shared Read:** grants_core_db read replica — reads `awards`, `organizations`, `grant_programs`
- **Shared Read:** financial_db read replica — reads `disbursements`, `grant_budgets`

### Shared Resources
- **Redis:** grants-sessions-redis (shared with grants-portal) — session storage
- **document-service:** progress report attachment uploads

### Data Flow — Progress Report Submission
```
[Grantee Browser]
    → [CloudFront] (Vue.js frontend)
    → [API Gateway] (grantee-connect-api Lambda)
    → [grantee_connect_db] (write: progress_reports, report_periods)
    → [document-service] (upload report attachments)
    → [notification-service] (notify grants officer of submission)
    → [grants_core_db read replica] (read: award details, program requirements)
    → [financial_db read replica] (read: disbursement status, budget)
```

### Data Flow — Disbursement View
```
[Grantee Browser]
    → [grantee-connect-api]
    → [financial_db read replica] (reads disbursements, grant_budgets)
    → [grants_core_db read replica] (reads award, grant_program)
```

## Shared Redis Risk
grantee-connect shares grants-sessions-redis with grants-portal. A Redis failure or memory exhaustion in grants-portal can evict grantee-connect sessions and log out active grantees mid-progress-report submission (data loss risk).

## Known Technical Debt
- Sessions in shared Redis — no namespacing, no memory quotas per service
- Disbursement data read directly from financial_db — no API contract with awards-management
- Budget amendment requests stored in grantee_connect_db but approval status is in awards-management — no sync mechanism, requires manual email workflow
- No auto-save on progress reports — network interruption loses unsaved work
