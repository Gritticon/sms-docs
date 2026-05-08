---
title: Architecture Overview
type: architecture
project: platform
last-updated: 2026-03-18
---

# Architecture Overview

## System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                        School Website                           │
│         ┌──────────────────┐  ┌──────────────────┐             │
│         │   SMS iframe     │  │   KYC iframe     │             │
│         └────────┬─────────┘  └────────┬─────────┘             │
└──────────────────┼──────────────────────┼───────────────────────┘
                   │                      │
     ┌─────────────▼──────┐  ┌────────────▼───────┐
     │     SMS Web App    │  │    KYC Web App      │
     │   (Flutter Web)    │  │  (Flutter Web,      │
     │   Staff-facing     │  │   Responsive)       │
     └─────────────┬──────┘  └────────────┬────────┘
                   │                      │
                   └──────────┬───────────┘
                              │
                   ┌──────────▼───────────┐
                   │    SMS API           │
                   │  (FastAPI / Python)  │
                   │  AWS ECS Fargate     │
                   └──────────┬───────────┘
                              │
               ┌──────────────┼──────────────┐
               │              │              │
    ┌──────────▼───┐  ┌───────▼──────┐  ┌───▼──────────┐
    │    MySQL     │  │   AWS S3     │  │ AWS Secrets  │
    │  (Database)  │  │ (File Store) │  │   Manager    │
    └──────────────┘  └──────────────┘  └──────────────┘

     ┌──────────────────────────────────┐
     │    School-Branded Mobile App     │
     │    (Flutter — one per school)    │
     │    Wraps KYC Web via WebView     │
     └──────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| SMS Frontend | Flutter Web | Material 3, GetX state management, GoRouter |
| KYC Frontend | Flutter Web | Claymorphism design, GetX, GoRouter, responsive |
| Mobile Wrapper | Flutter (iOS + Android) | One app per school, wraps KYC web |
| Backend API | Python / FastAPI | Async, service-layer architecture |
| ORM | SQLAlchemy 2.0 | Async support |
| Database | MySQL | Multi-tenant via dynamic tables |
| Auth | JWT (python-jose) | Configurable expiry, 30 days default |
| File Storage | AWS S3 (boto3) | Images, documents, uploads |
| Secrets | AWS Secrets Manager | No hardcoded credentials |
| Compute | AWS ECS Fargate | Serverless containers |
| Load Balancer | AWS ALB | Health checks, traffic routing |
| Container Registry | AWS ECR | Docker image storage |
| CI/CD | GitHub Actions | Build → ECR → ECS deploy pipeline |
| Marketing Site | Static HTML/CSS/JS | Nginx Alpine, Docker |

---

## Multi-Tenancy

### Strategy: Dynamic Table Isolation
Each school gets its own set of database tables, prefixed by `school_id`:

```
school_1_staff
school_1_students
school_1_classes
school_2_staff
school_2_students
school_2_classes
...
```

This provides **strong data isolation** between schools. No row-level filtering needed. The tradeoff is more database overhead as schools scale.

### Central Tables
- `customers` — tracks all schools with status: `ACTIVE`, `INACTIVE`, `SUSPENDED`, `TRIAL`
- `internal_employees` — Gritticon admin users with cross-school access

### School Status Lifecycle
```
TRIAL → ACTIVE → INACTIVE / SUSPENDED
```

### Request Flow
1. Request arrives at API with JWT token
2. `auth_middleware.py` validates token, extracts `school_id` and `user_id`
3. `RequestContext` is populated with `school_id`, `staff_record`, and permissions
4. All database queries are scoped to the school's tables automatically

---

## Authentication

### Two Login Types

| Type | Endpoint | Scope |
|------|----------|-------|
| Internal (Gritticon admin) | `POST /auth/login/internal` | Cross-school access |
| School staff | `POST /auth/login/staff` | School-scoped only |

### JWT Token
- Issued on login, stored in `GetStorage` (Flutter)
- Injected automatically into all API requests via Dio interceptor
- Configurable expiry (default 30 days)

---

## Role-Based Access Control (RBAC)

### Permission Registry
- 180 permissions total (IDs 1–180)
- Structure: **10 modules × ~5 actions** (view, create, edit, delete, manage)
- Stored as a JSON array in the `roles.permissions` column

### Modules
1. Dashboard
2. Communications
3. Transport
4. Student Management
5. Academics
6. Staff & Payroll
7. Classes & Academic Calendar
8. Complaints
9. School Settings
10. Sessions

### Role Assignment
- Every staff member is assigned a role
- Role carries a set of permission IDs
- Middleware enforces permission checks on each request

---

## API Design

| Property | Detail |
|----------|--------|
| Base URL (production) | `https://api.gritticon.com` |
| Base URL (development) | `http://localhost:8000` |
| Docs | `/docs` (Swagger), `/redoc` (ReDoc) |
| Auth | Bearer JWT on all protected routes |
| CORS | Configurable (default `*` in development) |
| Error handling | Custom middleware for HTTP, validation, SQLAlchemy, and general errors |
| Pagination | Supported on all list endpoints |

### Service Layer Architecture
```
Route (HTTP handler)
  → Schema (Pydantic validation)
    → Service (business logic)
      → Model (SQLAlchemy ORM)
        → Database
```

---

## Deployment Pipeline

```
Push to main branch
  → GitHub Actions triggered
    → Configure AWS credentials
      → Login to ECR
        → Build Docker image
          → Push to ECR
            → Update ECS task definition
              → Deploy to ECS Fargate
```

---

## File Storage (S3)
- Staff and student profile images
- Student documents and certificates
- Homework attachments (KYC diary)
- Any uploaded files from SMS or KYC

---

## Infrastructure Notes
- All credentials managed via AWS Secrets Manager — fetched at runtime with fallback to environment variables
- Database connection pooling with auto-reconnection (`pool_pre_ping=True`)
- Connection recycling after 1 hour
- Transactions with rollback on error
