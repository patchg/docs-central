# grants-portal — Architecture

## System Overview
Public-facing Next.js web application for grant application intake. Applicants register, complete eligibility questionnaires, upload supporting documents, and submit formal applications. Back-end is a Node.js REST API backed by grants_core_db.

## Components

### Frontend
- **Framework:** Next.js 13 (Pages Router)
- **Hosting:** CloudFront + S3 static export
- **Auth:** Redirects to auth-service for SSO login; stores JWT in httpOnly cookie

### Backend API
- **Runtime:** Node.js 18, Express
- **Hosting:** AWS Lambda + API Gateway
- **Database:** grants_core_db (PostgreSQL 14) — shared with review-engine, awards-management
- **Writes to tables:** `organizations`, `applicants`, `applications`
- **Reads from tables:** `grant_programs`, `fiscal_years`

### Document Handling
- Delegates all file uploads to `document-service`
- Documents stored in S3: `grants-documents-310207263177/applications/{id}/`

## Data Flow
```
[Applicant Browser]
    → [CloudFront/S3] (Next.js frontend)
    → [API Gateway] (grants-portal-api Lambda)
    → [grants_core_db] (writes: applications, applicants, organizations)
    → [document-service] (attachment uploads)
    → [notification-service] (confirmation emails)
    → [auth-service] (JWT validation on every request)
```

## Shared Database Risk
grants-portal shares grants_core_db with review-engine and awards-management. A schema migration by any team can break all three systems simultaneously. No migration coordination process exists — this is a known risk.

## Known Technical Debt
- Session tokens stored in Redis but Redis is also shared with grantee-connect
- No rate limiting on application submission — spam submissions have occurred
- Eligibility logic duplicated in both frontend and backend (gets out of sync)
- `applications` table has no soft-delete — cancelled applications are hard-deleted, breaking foreign keys in review_scores
