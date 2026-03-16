---
service: pip-test-service
system: pip
criticality: low
---

# Runbook: pip-test-service

## Service Overview
- **Purpose**: Test service for PIP Doc Mirror validation
- **Criticality**: Low (test environment only)
- **On-Call Team**: team-platform
- **Slack Channel**: #pip-platform

## Health Checks
```bash
curl https://pip-test-service.example.com/health
# Expected: 200 OK {"status": "healthy"}
```

## Common Issues

### Service Unavailable
```bash
kubectl get pods -n test | grep pip-test-service
kubectl logs -n test deployment/pip-test-service --tail=100
```
