# awards-management — Architecture

## System Overview
Internal Java/Spring Boot application for award lifecycle management. Used by grants officers and finance staff. The most operationally critical system — a failure here can delay federal grant disbursements.

## Components

### Frontend
- **Framework:** Angular 14 SPA
- **Hosting:** Internal nginx (same server as API)
- **Auth:** JWT via auth-service + SAML for finance officer role

### Backend API
- **Runtime:** Java 17, Spring Boot 3.1
- **Hosting:** EC2 (dedicated, 2x m5.xlarge — not auto-scaled)
- **Primary Database:** grants_core_db — reads `review_scores`, `panel_assignments`, `applications`; writes `awards`, `grant_programs` updates
- **Financial Database:** financial_db — writes `disbursements`, `disbursement_line_items`, `budget_lines`, `grant_budgets`; reads `fiscal_years`

### Disbursement Processing
1. Grants officer initiates disbursement in UI
2. awards-management writes to `disbursements` table in financial_db
3. Nightly batch job reads `disbursements` with status=PENDING
4. Batch calls federal payment gateway API (ACH transfer)
5. Updates disbursement status to PROCESSED or FAILED
6. notification-service sends payment confirmation to grantee-connect

## Cross-System Data Reads (No API Contract)
awards-management directly queries `review_scores` and `panel_assignments` from grants_core_db — tables owned by review-engine. No API layer exists between these systems. This means:
- review-engine schema changes can silently break awards-management
- No versioning or backwards compatibility guarantees

## Known Technical Debt
- Two dedicated EC2 instances with no auto-scaling — failure of one instance degrades performance by 50%
- Disbursement batch job has no idempotency key — double-runs can cause duplicate ACH transfers (happened once in 2022)
- financial_db and grants_core_db transactions are NOT coordinated — a crash mid-award can leave records partially committed across both databases
- No API for external systems — compliance-reporter reads financial_db tables directly
