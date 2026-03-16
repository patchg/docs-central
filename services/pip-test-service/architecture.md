---
service: pip-test-service
system: pip
owners: [team-platform]
last-reviewed: 2026-03-16
tags: [api, test]
---

# Architecture: pip-test-service

## Overview
A test service used to validate the PIP Doc Mirror sync pipeline.

## Components
- **API Layer**: Simple REST endpoints
- **Data Layer**: PostgreSQL database

## Data Flow
```
[Client] --> [API Gateway] --> [pip-test-service] --> [PostgreSQL]
```

## Design Decisions
- Stateless design for easy horizontal scaling
- JWT authentication via shared auth-service

## Security
- Authentication: JWT via auth-service
- Authorization: Role-based (admin, read-only)
- Data encryption: TLS in transit, AES-256 at rest
