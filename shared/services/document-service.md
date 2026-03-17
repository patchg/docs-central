# document-service — Shared Document Storage Service

**Repo:** patchg/document-service  
**Owner:** Platform Team  
**Endpoint:** https://documents.internal/api/v1  
**Storage Backend:** S3 bucket `grants-documents-310207263177`

## Overview
Centralized document upload, virus scanning, and retrieval service. Wraps S3 with access control and ClamAV virus scanning. Used by grants-portal (application attachments) and grantee-connect (progress report attachments).

## Consumer Systems

| System | Operations | Avg File Size | Monthly Volume |
|--------|-----------|---------------|----------------|
| grants-portal | upload, retrieve, delete | 2.1 MB | ~8,000 files |
| grantee-connect | upload, retrieve | 1.8 MB | ~3,500 files |
| compliance-reporter | retrieve (read-only) | N/A | ~500 retrievals |

## S3 Structure
```
grants-documents-310207263177/
├── applications/{application_id}/          # grants-portal uploads
│   ├── narrative.pdf
│   ├── budget.xlsx
│   └── attachments/
├── progress-reports/{award_id}/{period}/   # grantee-connect uploads
└── compliance/{fiscal_year}/               # compliance-reporter archives
```

## Access Control
Documents are access-controlled by service JWT. Cross-service document access (e.g., compliance-reporter reading application documents) requires a service-to-service token from auth-service.

## Known Issues
- Virus scan adds 3–8 second latency to uploads — grants-portal shows false "upload failed" to users during scan
- No versioning — overwritten documents are unrecoverable
- compliance-reporter accesses S3 directly (bypassing document-service) for bulk exports
