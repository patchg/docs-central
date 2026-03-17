# compliance-reporter — Documentation

Federal and state regulatory compliance reporting system. This /docs folder syncs to docs-central via Doc Mirror.

## Files
- `architecture.md` — Architecture
- `dependencies.yaml` — Dependencies
- `runbook.md` — Operational runbook

## Overview
compliance-reporter generates mandated federal and state grant reports: SF-425 financial status reports, annual performance reports, OMB A-133 audit support packages, and ad-hoc Congressional inquiries. Reads from grants_core_db and financial_db directly — no API intermediary.
