# grantee-connect — Operational Runbook

## Common Incidents

### P1: Grantees logged out mid-report — progress report deadline day
**Symptoms:** Grantees report being logged out; unsaved progress reports lost  
**Cause:** grants-sessions-redis evicted grantee-connect sessions due to memory pressure from grants-portal  
**Resolution:**
1. Check Redis memory: `redis-cli -h grants-redis.internal INFO memory | grep used_memory_human`
2. Check which keys are consuming memory: `redis-cli -h grants-redis.internal --bigkeys`
3. If grants-portal sessions are dominant: contact grants-portal team to reduce session TTL temporarily
4. Grantees who lost work must re-enter — no auto-save. Communicate via email.
5. Long-term fix: separate Redis instances per service (tracked in INFRA-89)

### P1: Grantees cannot view disbursements (financial_db slow)
**Symptoms:** Disbursement page times out; grantees calling helpdesk  
**Cause:** awards-management running heavy queries on financial_db during batch processing  
**Resolution:**
1. Check financial_db replica lag: `SELECT now() - pg_last_xact_replay_timestamp() AS lag;`
2. If replica lag >5min: read replica is behind, grantees see stale data
3. Check awards-management batch job status — if running, wait for completion
4. Disable disbursement view temporarily via feature flag: `DISABLE_DISBURSEMENT_VIEW=true` in Lambda env vars

### P2: Amendment request stuck — no response from grants officer
**Symptoms:** Grantee submitted amendment but status shows "Pending" for >5 business days  
**Cause:** Amendment requests are stored in grantee_connect_db but awards-management has no integration — grants officers must be manually notified  
**Resolution:**
1. Email grants officer directly with amendment_request_id from grantee_connect_db
2. Once approved in awards-management, manually update grantee_connect_db:
   `UPDATE amendment_requests SET status='approved', approved_at=NOW() WHERE id=?`
3. This is a known workflow gap — tracked in GMP-678

### P2: Progress report deadline approaching — report submission failing
**Symptoms:** Grantees get error on Submit button; likely document-service issue  
**Resolution:**
1. Check document-service: `curl https://documents.internal/health`
2. If document-service is down: enable text-only mode via Lambda env var: `DOCUMENT_UPLOAD_ENABLED=false`
3. Communicate to grantees: submit text narrative now, attach documents when service recovers
4. Extend soft deadline in grantee_connect_db: `UPDATE report_periods SET soft_deadline = soft_deadline + INTERVAL '2 days' WHERE program_id=?`

## Reporting Deadline Calendar (Typical)
- Quarterly progress reports: 30 days after quarter end
- Annual performance reports: 90 days after program year end
- Final closeout reports: 90 days after award end date

## Deployment
- CI/CD: GitHub Actions → Lambda container deployment
- Deploy time: ~6 minutes (container build + Lambda update)
- **Do not deploy within 48 hours of a major reporting deadline**
