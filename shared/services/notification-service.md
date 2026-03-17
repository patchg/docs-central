# notification-service — Shared Notification Service

**Repo:** patchg/notification-service  
**Owner:** Platform Team  
**Endpoint:** https://notifications.internal/api/v1  
**Type:** Async email/SMS/in-app notifications via SQS

## Overview
Centralized notification service used by grants-portal, awards-management, and grantee-connect. Sends transactional emails via AWS SES, SMS via Twilio, and in-app notifications stored in Redis.

## Consumer Systems

| System | Notification Types | Volume (daily avg) |
|--------|-------------------|-------------------|
| grants-portal | Application received, deadline reminders | ~200/day |
| awards-management | Award issued, payment processed, declined | ~50/day |
| grantee-connect | Report due, payment released, amendment approved | ~150/day |

## Message Queue
- SQS queue: `notifications-prod`
- Dead letter queue: `notifications-prod-dlq` (max receives: 3)
- Visibility timeout: 30 seconds
- Message retention: 4 days

## Known Issues
- No deduplication — grants-portal occasionally sends duplicate "application received" emails due to retry logic
- grantee-connect bypasses the queue for urgent messages (direct SES calls) — creates split delivery tracking
- SMS failures are silently dropped (Twilio errors logged but no alerting)
