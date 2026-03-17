# compliance-reporter — Operational Runbook

## ⚠️ Regulatory Risk
Failure to generate reports on time triggers federal compliance violations. SF-425 deadlines are 30 days after quarter end. Missed deadlines can result in funding suspension.

## Quarterly Report Deadline Calendar
| Quarter End | SF-425 Due | Drawdown Due |
|-------------|-----------|--------------|
| March 31 | April 30 | April 15 |
| June 30 | July 30 | July 15 |
| September 30 | October 30 | October 15 |
| December 31 | January 30 | January 15 |

## Common Incidents

### P1: SF-425 generation failed — deadline approaching
**Symptoms:** APScheduler job failed; no PDF in output directory  
**Resolution:**
1. SSH to instance: `ssh compliance-reporter-ec2`
2. Check logs: `tail -200 /var/log/compliance-reporter/app.log`
3. Most common cause: schema change in financial_db broke a raw SQL query
4. Identify failing query: grep logs for `psycopg2.ProgrammingError`
5. Contact awards-management team to identify recent migrations
6. Manual trigger: `cd /opt/compliance && python generate_report.py --type sf425 --quarter Q{n} --year {yyyy}`

### P1: Single EC2 instance down during report period
**Symptoms:** App unreachable; reports not generating  
**Resolution:**
1. Check instance: `aws ec2 describe-instance-status --instance-ids i-compliance-reporter`
2. If instance is stopped: `aws ec2 start-instances --instance-ids i-compliance-reporter`
3. If instance is terminated: launch from AMI `ami-compliance-reporter-latest`
4. Note: report output stored locally may be lost — check S3 backup bucket `compliance-reports-backup` (manual sync only)

### P2: SF-425 generation taking >1 hour
**Symptoms:** Report not ready; EC2 CPU at 100%  
**Cause:** No pagination on large program queries  
**Resolution:**
1. Increase EC2 instance type temporarily: stop → change to m5.xlarge → start
2. Monitor query: `ssh compliance-reporter-ec2 'ps aux | grep python'`
3. Do not kill the process during active database queries — can corrupt partial reports

## Schema Change Impact Assessment
Before any change to these tables, contact compliance-team@grants.org:
- `disbursements`, `disbursement_line_items`, `budget_lines` (financial_db)
- `applications`, `awards`, `review_scores` (grants_core_db)

Run test report generation in staging before deploying schema changes.
