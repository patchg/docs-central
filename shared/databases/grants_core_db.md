# grants_core_db — Shared Core Database

**Type:** PostgreSQL 14  
**Host:** grants-core-postgres.internal  
**Port:** 5432  
**Owner:** No single owner — legacy shared database used by multiple systems  

## ⚠️ Warning: Shared Database Anti-Pattern

This database is shared by `grants-portal`, `review-engine`, and `awards-management`. Each system reads and writes directly to shared tables. Schema changes require coordination across all three teams.

## Shared Tables

| Table | Primary Writer | Readers |
|-------|----------------|---------|
| `organizations` | grants-portal | review-engine, awards-management, grantee-connect |
| `applicants` | grants-portal | review-engine, awards-management |
| `grant_programs` | awards-management | grants-portal, review-engine, compliance-reporter |
| `applications` | grants-portal | review-engine, awards-management, compliance-reporter |
| `awards` | awards-management | compliance-reporter, grantee-connect |
| `review_scores` | review-engine | awards-management, compliance-reporter |
| `panel_assignments` | review-engine | awards-management |

## Connection Limits
Max connections: 200. Current allocation:
- grants-portal: 40 connections (pool)
- review-engine: 30 connections (pool)
- awards-management: 50 connections (pool)
- grantee-connect: 20 connections (read replica)
- compliance-reporter: 10 connections (read replica)

**Read replica:** grants-core-readonly.internal (grantee-connect and compliance-reporter should use this)

## Known Issues
- `applications` table has ~4.2M rows with no partition — full scans are slow
- `review_scores` lacks an index on `reviewer_id` — review-engine queries degrade under load
- No row-level security — all services have access to all rows

## Backup
Daily snapshots to S3 at 02:00 UTC. RTO: 4 hours. RPO: 24 hours.
