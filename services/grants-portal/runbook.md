# grants-portal — Operational Runbook

## Common Incidents

### P1: Portal completely down (auth-service outage)
**Symptoms:** All users see login error; no applications can be submitted  
**Cause:** auth-service is unavailable — grants-portal has no fallback  
**Resolution:**
1. Page auth-service oncall: `auth-service-oncall@grants.org`
2. Check auth-service status: `curl https://auth.internal/health`
3. If auth-service is up but portal is down, restart grants-portal-api Lambda
4. Estimated recovery: 15–30 min

### P1: grants_core_db connection pool exhausted
**Symptoms:** 500 errors on application submission; DB errors in CloudWatch  
**Cause:** Connection limit (40) exceeded — often caused by review-engine or awards-management holding connections  
**Resolution:**
1. Check connections by service: `SELECT application_name, count(*) FROM pg_stat_activity GROUP BY 1;`
2. Kill idle connections from other services if needed
3. If grants-portal is exhausted, scale Lambda concurrency down temporarily
4. Long-term: implement read replica routing for GET requests

### P2: Document upload failures (virus scan timeout)
**Symptoms:** Users report "upload failed" on files that are clean  
**Cause:** ClamAV scan in document-service takes 3–8s; Lambda timeout fires  
**Resolution:**
1. Increase grants-portal-api Lambda timeout from 10s to 30s
2. Check document-service health: `curl https://documents.internal/health`
3. Check ClamAV queue depth in document-service CloudWatch

### P2: Duplicate confirmation emails
**Symptoms:** Applicants report receiving 2-3 confirmation emails  
**Cause:** Lambda retry on notification-service timeout sends duplicate SQS messages  
**Resolution:**
1. Check notification-service DLQ for backed-up messages
2. No immediate fix — inform affected applicants manually
3. Known bug: tracked in JIRA GMP-445

## Deployment
- CI/CD: GitHub Actions → Lambda deployment
- Deploy time: ~4 minutes
- Rollback: `aws lambda update-function-code --function-name grants-portal-api --s3-key previous-version.zip`
- **Do not deploy during application deadlines** — portal has no maintenance mode

## Escalation
- L1: applicant-experience-oncall@grants.org
- L2: platform-team@grants.org (for DB/shared service issues)
- L3: CTO (for deadline-day P1 incidents)
