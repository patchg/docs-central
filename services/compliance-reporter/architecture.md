# compliance-reporter — Architecture

## System Overview
Python/Flask reporting application running on a single EC2 instance. Generates scheduled and on-demand compliance reports for federal and state funders. Accesses grants_core_db and financial_db using read-only credentials.

## Components

### Application
- **Runtime:** Python 3.10, Flask 2.3
- **Hosting:** Single EC2 t3.medium (no redundancy)
- **Report Generation:** ReportLab (PDF), openpyxl (Excel), Jinja2 (HTML)
- **Scheduler:** APScheduler — generates quarterly reports automatically

### Database Access (Direct — No API)
- **grants_core_db (read replica):** applications, awards, grant_programs, organizations, review_scores
- **financial_db (read replica):** disbursements, budget_lines, grant_budgets, audit_log, fiscal_years
- **document-service S3 bypass:** reads `grants-documents-310207263177` directly via boto3 for bulk document exports

This is the most tightly coupled system in the portfolio. Any schema change to either database must consider compliance-reporter's raw SQL queries.

### Report Types
| Report | Frequency | Database(s) | Federal Requirement |
|--------|-----------|-------------|---------------------|
| SF-425 Financial Status | Quarterly | financial_db | 2 CFR 200.328 |
| Annual Performance Report | Annual | grants_core_db | Program-specific |
| Audit Support Package | Annual | both | 2 CFR 200 Subpart F |
| Drawdown Summary | Monthly | financial_db | Treasury requirement |
| Active Awards Report | Ad-hoc | grants_core_db | Congressional |

## Critical Risk: No Redundancy
compliance-reporter runs on a single EC2 instance. An instance failure during quarterly reporting close (last week of March, June, September, December) can trigger federal reporting violations with financial penalties.

## Known Technical Debt
- Raw SQL queries throughout — no ORM, no query parameterization in 3 legacy reports (SQL injection risk identified in 2023 audit — not yet remediated)
- S3 direct access bypasses document-service access controls
- No pagination on large report queries — SF-425 generation can take 45 minutes for large portfolios
- Report output is stored locally on EC2 — no backup, no version history
