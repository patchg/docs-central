# review-engine — Operational Runbook

## Common Incidents

### P1: Review portal inaccessible during panel session
**Symptoms:** Reviewers cannot log in or load applications; deadline at risk  
**Common Causes:**
1. auth-service outage (SAML not working)
2. grants_core_db connection exhaustion
3. EC2 instances in bad state  

**Resolution:**
1. Check EC2 health: `aws ec2 describe-instance-status --filters Name=instance-state-name,Values=running`
2. Check auth-service: `curl https://auth.internal/health`
3. Check DB connections: log into grants-core-postgres, run `SELECT count(*) FROM pg_stat_activity WHERE application_name='review-engine'`
4. If instances are healthy but application is broken: rolling restart via ASG refresh

### P1: Scores missing in awards-management after panel closes
**Symptoms:** awards-management reports review_scores table has no records for a program  
**Cause:** Panel chair forgot to click "Finalize" — scores in draft state are excluded from awards-management query  
**Resolution:**
1. Log in as program officer in review-engine
2. Navigate to the panel → click "Finalize Panel"
3. Verify scores appear: `SELECT count(*) FROM review_scores WHERE program_id = ? AND status = 'final'`

### P2: N+1 query timeout on large panels (>500 applications)
**Symptoms:** Application list page times out (30s); Django logs show hundreds of SQL queries  
**Resolution:**
1. Enable query cache for the affected panel: `redis-cli SET panel_cache:{panel_id} 1 EX 900`
2. Restart the affected EC2 instance to clear Django ORM cache state
3. Long-term fix: tracked in JIRA GMP-312 (select_related optimization)

## JWT Key Rotation Procedure
review-engine caches the JWT signing key at startup.
1. Coordinate with auth-service team before rotating
2. After rotation, restart all EC2 instances in the ASG
3. Rolling restart: `aws autoscaling start-instance-refresh --auto-scaling-group-name review-engine-asg`

## Deployment
- Manual deployment via Ansible playbook: `ansible-playbook deploy-review-engine.yml`
- Deploy time: ~12 minutes (AMI bake + rolling restart)
- **Do not deploy during active review panels**
