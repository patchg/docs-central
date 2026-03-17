# awards-management — Operational Runbook

## ⚠️ High Stakes System
Failures in awards-management can delay federal grant disbursements and trigger compliance violations. Escalate P1 incidents immediately.

## Common Incidents

### P1: Disbursement batch job failed — payments not sent
**Symptoms:** Grantees report non-receipt of payment; `disbursements` table shows status=PENDING past 06:00 UTC  
**Resolution:**
1. Check batch job logs: `aws logs filter-log-events --log-group /aws/ec2/awards-management --filter-pattern BATCH_ERROR`
2. Check federal payment gateway: `curl https://pay.treasury.gov/status`
3. **DO NOT re-run the batch manually without checking for partial runs** — duplicate ACH risk
4. Verify idempotency: `SELECT id, status, created_at FROM disbursements WHERE status='PENDING' AND created_at < NOW() - INTERVAL '12 hours'`
5. If safe to re-run: `ssh awards-mgmt-ec2-01 'sudo systemctl restart awards-batch'`
6. Escalate to finance director if payment gateway is unavailable

### P1: Partial commit across grants_core_db and financial_db
**Symptoms:** Award appears in `awards` table but no `disbursements` record; or vice versa  
**Cause:** awards-management crash between cross-database writes  
**Resolution:**
1. Identify orphaned records: compare `awards.id` with `grant_budgets.award_id` in financial_db
2. Manual reconciliation required — no automated fix
3. Engage DBA and grants officer to determine correct state
4. Document in audit_log manually

### P2: EC2 instance unhealthy — running on single node
**Symptoms:** Response times doubled; one instance failing health checks  
**Resolution:**
1. Terminate unhealthy instance: `aws ec2 terminate-instances --instance-ids <id>`
2. Launch replacement from AMI: `aws ec2 run-instances --launch-template Lt-awards-mgmt`
3. Monitor until new instance passes health checks (~8 min)

## Schema Change Coordination
Before ANY schema change to grants_core_db or financial_db:
1. Notify review-engine team (reads review_scores) 
2. Notify compliance-reporter team (reads financial_db directly)
3. Notify grantee-connect team (reads disbursements)
4. Require 5-business-day notice minimum

## Deployment
- Manual: `ssh awards-mgmt-ec2-01 'cd /opt/awards && ./deploy.sh <version>'`
- Rolling deploy to node 2, then node 1
- **Never deploy during last 3 business days of a fiscal quarter**
