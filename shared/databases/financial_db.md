# financial_db — Shared Financial Database

**Type:** PostgreSQL 14  
**Host:** grants-financial-postgres.internal  
**Port:** 5432  
**Owner:** No single owner — shared by awards-management and compliance-reporter

## ⚠️ Warning: Shared Database Anti-Pattern

`awards-management` writes disbursement and budget data; `compliance-reporter` reads it for regulatory reports. Direct table access creates tight coupling and prevents independent deployments.

## Shared Tables

| Table | Primary Writer | Readers |
|-------|----------------|---------|
| `disbursements` | awards-management | compliance-reporter, grantee-connect |
| `disbursement_line_items` | awards-management | compliance-reporter |
| `budget_lines` | awards-management | compliance-reporter |
| `fiscal_years` | awards-management | compliance-reporter, grants-portal |
| `grant_budgets` | awards-management | compliance-reporter, grantee-connect |
| `audit_log` | awards-management, compliance-reporter | compliance-reporter |

## Compliance Requirements
This database contains financial data subject to federal audit requirements (2 CFR Part 200). Retention: 7 years minimum. All writes are logged to `audit_log`.

## Connection Limits
Max connections: 100.
- awards-management: 30 connections
- compliance-reporter: 20 connections
- grantee-connect: 10 connections (read replica)

## Backup
Continuous WAL archiving + daily snapshots. RTO: 2 hours. RPO: 1 hour.
