# auth-service — Shared Authentication Service

**Repo:** patchg/auth-service  
**Owner:** Platform Team  
**Endpoint:** https://auth.internal/api/v1  
**Type:** JWT-based SSO with SAML 2.0 federation

## Overview
Central authentication and authorization service used by all five grants management systems. Integrates with the organization's Active Directory via SAML federation. Issues short-lived JWTs (15 min) with refresh tokens (7 days).

## Consumer Systems

| System | Auth Method | User Roles Issued |
|--------|-------------|------------------|
| grants-portal | JWT | applicant, org_admin |
| review-engine | JWT + SAML | reviewer, panel_chair, program_officer |
| awards-management | JWT + SAML | grants_officer, finance_officer, admin |
| compliance-reporter | JWT + SAML | compliance_officer, auditor, read_only |
| grantee-connect | JWT | grantee_contact, grantee_finance |

## Critical Dependency
**All five systems are hard-blocked if auth-service is unavailable.** There is no graceful degradation — an auth-service outage brings down all grants management systems simultaneously.

## SLA
- Uptime: 99.9% (8.7 hours downtime/year allowed)
- Latency p99: < 200ms for token validation
- Latency p99: < 500ms for SAML login flow

## Known Issues
- No circuit breaker in consumer systems — cascading failures observed during auth-service restarts
- Token refresh storms during scheduled restarts (all sessions expire simultaneously)
- review-engine caches JWT signing keys locally — requires manual restart after key rotation
