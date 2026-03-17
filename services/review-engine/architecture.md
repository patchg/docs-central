# review-engine — Architecture

## System Overview
Internal React SPA + Python/Django REST API for grant review workflow. Used exclusively by internal staff (reviewers, panel chairs, program officers). Not publicly accessible.

## Components

### Frontend
- **Framework:** React 17, Create React App (legacy build)
- **Hosting:** Internal nginx server (not on CloudFront)
- **Auth:** SAML 2.0 via auth-service; role-based access enforced in Django

### Backend API
- **Runtime:** Python 3.9, Django 4.2, Django REST Framework
- **Hosting:** EC2 Auto Scaling Group (2–6 instances, t3.large)
- **Database:** grants_core_db (shared) — reads `applications`, `organizations`, `applicants`; writes `review_scores`, `panel_assignments`
- **Cache:** Elasticache Redis (dedicated, not shared) — application list cache, 15-min TTL

### Review Workflow
1. Program officer creates review panel and assigns applications
2. System writes `panel_assignments` records to grants_core_db
3. Reviewers score each criterion via rubric form
4. Scores written to `review_scores` in grants_core_db
5. Panel chair views aggregated scores and enters funding recommendation
6. awards-management reads `review_scores` directly from grants_core_db

## Shared Database Risk
review-engine shares grants_core_db with grants-portal and awards-management. The `review_scores` and `panel_assignments` tables have no API layer — awards-management reads them with direct SQL, creating an invisible coupling point.

## Known Technical Debt
- EC2 deployment (not Lambda/ECS) — manual AMI updates required for security patches
- Django ORM generates N+1 queries on application list views — degrades at >500 applications/panel
- JWT signing key is cached at startup — does not rotate automatically, requires server restart after auth-service key rotation
- No audit trail for score edits — reviewers can change scores after panel reconciliation
