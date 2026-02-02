# Production Boilerplate Specification

> A comprehensive, framework-agnostic specification for generating production-ready full-stack application boilerplates.

---

## Table of Contents

1. [Pre-Built Features](#pre-built-features)
2. [Frontend Specifications](#1-frontend-specifications)
3. [Backend Specifications](#2-backend-specifications)
4. [Database Specifications](#3-database-specifications)
5. [Caching & Session Specifications](#4-caching--session-specifications)
6. [Security Specifications](#5-security-specifications)
7. [Testing Specifications](#6-testing-specifications)
8. [CI/CD Specifications](#7-cicd-specifications)
9. [Observability Specifications](#8-observability-specifications)
10. [Infrastructure Specifications](#9-infrastructure-specifications)
11. [Developer Experience Specifications](#10-developer-experience-specifications)
12. [API Documentation Specifications](#11-api-documentation-specifications)
13. [Project Structure Specifications](#12-project-structure-specifications)

---

## Pre-Built Features

These features come **fully implemented and working** out of the box. No additional coding required — just configure environment variables and deploy.

### 🔐 Authentication Features

#### Email/Password Authentication
- [x] User registration with email and password
- [x] Email validation (format, disposable email blocking optional)
- [x] Password strength validation (min length, complexity rules)
- [x] Password hashing with Argon2/bcrypt (secure cost factor)
- [x] User login with email and password
- [x] "Remember me" functionality (extended session)
- [x] Account email verification via OTP or magic link
- [x] Resend verification email
- [x] Forgot password flow (request reset)
- [x] Password reset via secure token (time-limited)
- [x] Change password (authenticated)
- [x] Account deactivation/deletion

#### OAuth/Social Authentication
- [x] Google OAuth login (default)
- [x] OAuth callback handling
- [x] Account linking (connect social to existing account)
- [x] Profile picture sync from OAuth provider
- [x] First-time OAuth user profile completion flow
- [ ] GitHub OAuth (extensible, template provided)
- [ ] Facebook OAuth (extensible, template provided)
- [ ] Apple OAuth (extensible, template provided)
- [ ] Twitter/X OAuth (extensible, template provided)
- [ ] LinkedIn OAuth (extensible, template provided)
- [ ] Microsoft OAuth (extensible, template provided)
- [ ] Discord OAuth (extensible, template provided)

#### Token Management
- [x] JWT access token generation
- [x] JWT refresh token generation
- [x] Access token refresh endpoint
- [x] Token blacklisting on logout
- [x] Token revocation (all devices)
- [x] Secure cookie storage for tokens
- [x] Token expiration configuration
- [x] Automatic token refresh on frontend

#### Session Management
- [x] Active sessions list (view all logged-in devices)
- [x] Session termination (logout specific device)
- [x] Logout from all devices
- [x] Session activity tracking (last active, IP, user agent)
- [x] Concurrent session limits (optional, configurable)

#### Security Features
- [x] Account lockout after N failed login attempts
- [x] Lockout notification email
- [x] Login attempt logging (IP, timestamp, success/fail)
- [x] Suspicious login detection hooks
- [x] CSRF protection on auth endpoints
- [x] Rate limiting on auth endpoints

---

### 👤 User Management Features

#### Profile Management
- [x] Get current user profile
- [x] Update profile (name, bio, etc.)
- [x] Upload profile picture
- [x] Delete profile picture
- [x] Profile picture resize/optimization

#### Account Settings
- [x] Update email (with re-verification)
- [x] Update password
- [x] Two-factor authentication setup hooks (extensible)
- [x] Notification preferences
- [x] Privacy settings structure
- [x] Account export (GDPR compliance structure)
- [x] Account deletion with confirmation

---

### 🔑 Authorization Features

#### Role-Based Access Control (RBAC)
- [x] User roles (admin, user, moderator - extensible)
- [x] Role assignment to users
- [x] Role-based route protection
- [x] Role-based UI element visibility
- [x] Role hierarchy support

#### Permission System
- [x] Granular permissions structure
- [x] Permission assignment to roles
- [x] Permission checking middleware
- [x] Permission checking hooks for frontend
- [x] Resource-based permissions (own resources vs others)

---

### 📧 Email Features

#### Transactional Emails (Pre-Built Templates)
- [x] Welcome email (on registration)
- [x] Email verification email
- [x] Password reset email
- [x] Password changed confirmation
- [x] Login from new device alert
- [x] Account locked notification
- [x] Account deletion confirmation

#### Email System
- [x] Email service abstraction (Nodemailer, SendGrid, SES ready)
- [x] HTML + plain text email templates
- [x] Email queue (non-blocking sending)
- [x] Email preview in development
- [x] Template variable injection
- [x] Unsubscribe link structure

---

### 📁 File Management Features

#### File Upload
- [x] Single file upload endpoint
- [x] Multiple file upload endpoint
- [x] File type validation (whitelist)
- [x] File size limits
- [x] Virus scanning hooks (extensible)
- [x] Upload progress tracking (frontend)

#### File Storage
- [x] Local storage (development)
- [x] Cloud storage abstraction (S3/GCS/Azure ready)
- [x] Signed URL generation (private files)
- [x] Public URL generation (public files)
- [x] File deletion
- [x] File metadata tracking

#### Image Processing
- [x] Image resize on upload
- [x] Thumbnail generation
- [x] Image compression
- [x] Format conversion hooks

---

### 🔔 Notification Features

#### In-App Notifications
- [x] Notification model and storage
- [x] Create notification
- [x] Get user notifications (paginated)
- [x] Mark notification as read
- [x] Mark all as read
- [x] Delete notification
- [x] Unread count endpoint

#### Real-Time Notifications (Hooks)
- [x] WebSocket/SSE structure (extensible)
- [x] Push notification structure (extensible)
- [x] Notification preferences per type

---

### 📊 API Features

#### Standard Endpoints
- [x] Health check endpoint (`/health`)
- [x] Readiness endpoint (`/health/ready`)
- [x] Liveness endpoint (`/health/live`)
- [x] API version endpoint
- [x] Server time endpoint

#### Request Handling
- [x] Request body validation
- [x] Query parameter validation
- [x] Path parameter validation
- [x] File upload handling
- [x] Multipart form handling

#### Response Handling
- [x] Standardized response envelope
- [x] Pagination helpers (offset + cursor-based)
- [x] Sorting helpers
- [x] Filtering helpers
- [x] Field selection (partial response)
- [x] Error response formatting

#### Rate Limiting
- [x] Global rate limit
- [x] Per-endpoint rate limit
- [x] Per-user rate limit
- [x] Per-IP rate limit
- [x] Rate limit headers in response
- [x] Custom rate limit windows

---

### 🛡️ Security Features (Pre-Configured)

#### HTTP Security
- [x] CORS configured (origin whitelist)
- [x] Helmet/security headers
- [x] Content-Security-Policy
- [x] X-Frame-Options (anti-clickjacking)
- [x] X-Content-Type-Options
- [x] Strict-Transport-Security (HSTS)
- [x] Referrer-Policy
- [x] Request size limits
- [x] Request timeout

#### Attack Prevention
- [x] XSS prevention (input sanitization)
- [x] SQL injection prevention (parameterized queries)
- [x] NoSQL injection prevention
- [x] CSRF tokens
- [x] Path traversal prevention
- [x] Parameter pollution prevention

#### Data Protection
- [x] Password hashing
- [x] Sensitive data encryption helpers
- [x] PII masking in logs
- [x] Secure session cookies
- [x] HTTPOnly cookies
- [x] SameSite cookie policy

---

### 💾 Database Features

#### Schema
- [x] User table with all auth fields
- [x] Session/token table
- [x] Role and permission tables
- [x] Audit log table
- [x] Notification table
- [x] File/media table

#### Operations
- [x] Migration runner
- [x] Rollback capability
- [x] Seed scripts (dev + prod)
- [x] Connection pooling
- [x] Transaction helpers
- [x] Soft delete support
- [x] Timestamps auto-managed

---

### 🔴 Caching Features

#### Redis Integration
- [x] Redis client configured
- [x] Connection health check
- [x] Session storage in Redis
- [x] Token blacklist in Redis
- [x] Rate limit counters in Redis
- [x] Cache helpers (get, set, delete, invalidate)
- [x] Cache TTL management
- [x] Cache key namespacing

---

### 📝 Logging Features

- [x] Structured JSON logging
- [x] Log levels (debug, info, warn, error)
- [x] Request logging middleware
- [x] Request correlation IDs
- [x] Error logging with stack traces
- [x] PII masking in logs
- [x] Log output configuration (console, file)
- [x] Log rotation (file-based)

---

### 🧪 Testing Features (Pre-Written Tests)

#### Auth Tests
- [x] Registration success/failure tests
- [x] Login success/failure tests
- [x] Password reset flow tests
- [x] Token refresh tests
- [x] Logout tests
- [x] OAuth flow tests

#### API Tests
- [x] Health check tests
- [x] Rate limiting tests
- [x] Validation error tests
- [x] Authorization tests
- [x] Error handling tests

#### Infrastructure Tests
- [x] Database connectivity tests
- [x] Redis connectivity tests
- [x] Email service tests (mock)
- [x] File upload tests

---

### 🚀 Deployment Features

#### Docker
- [x] Multi-stage Dockerfile
- [x] docker-compose.yml (dev)
- [x] docker-compose.prod.yml (production)
- [x] Non-root user in container
- [x] Health check in Dockerfile
- [x] Optimized layer caching
- [x] .dockerignore configured

#### CI/CD Pipeline Template
- [x] Lint stage
- [x] Type check stage
- [x] Unit test stage
- [x] Integration test stage
- [x] Build stage
- [x] Security scan stage
- [x] Docker build stage
- [x] Deploy stage (template)

---

### 🖥️ Frontend Features

#### UI Components (Pre-Integrated)
- [x] Component library setup (Shadcn/Radix/equivalent)
- [x] Design system with tokens
- [x] Dark/Light mode toggle
- [x] Toast notifications
- [x] Modal/dialog system
- [x] Loading spinners
- [x] Skeleton loaders
- [x] Error boundaries

#### Auth UI (Pre-Built Pages)
- [x] Login page
- [x] Registration page
- [x] Forgot password page
- [x] Reset password page
- [x] Email verification page
- [x] OAuth callback handler
- [x] Profile page
- [x] Settings page
- [x] Change password section

#### Forms
- [x] Form library integrated
- [x] Validation with schema
- [x] Error message display
- [x] Loading states on submit
- [x] File upload with preview

#### API Integration
- [x] HTTP client configured
- [x] Auth token interceptor
- [x] Automatic token refresh
- [x] Error handling
- [x] Request cancellation
- [x] Retry logic

#### Routing
- [x] Protected route component
- [x] Public-only route component
- [x] Role-based route guards
- [x] 404 page
- [x] Error page
- [x] Loading page

---

### 📚 Documentation Features

- [x] README with full setup instructions
- [x] Environment variables documentation
- [x] API documentation auto-generated
- [x] Architecture overview
- [x] Folder structure explanation
- [x] Contribution guidelines template
- [x] License file

---

### 🛠️ Developer Experience Features

- [x] ESLint configured with strict rules
- [x] Prettier configured
- [x] TypeScript strict mode
- [x] Pre-commit hooks (lint, format)
- [x] Commit message linting
- [x] VS Code settings/extensions recommended
- [x] Debug configuration (.vscode/launch.json)
- [x] Hot reload in development
- [x] API client collection (Postman/Insomnia)

---

## Core Categories Overview

| Category | What's Covered |
|----------|----------------|
| **1. Frontend** | Auth, State, UI, Forms, API Layer, Performance |
| **2. Backend** | Auth, API, Security, Middleware, Error Handling |
| **3. Database** | ORM, Migrations, Seeds, Connection Pooling |
| **4. Caching & Session** | Redis, Session Management, Cache Strategies |
| **5. Security** | OWASP Top 10, Headers, Encryption, Secrets |
| **6. Testing** | Unit, Integration, E2E, Contract Tests |
| **7. CI/CD** | Pipelines, Deployment, Environment Management |
| **8. Observability** | Logging, Metrics, Tracing, Alerting |
| **9. Infrastructure** | Docker, Orchestration, Reverse Proxy |
| **10. Developer Experience** | Linting, Formatting, Git Hooks, Documentation |
| **11. API Design** | Versioning, Documentation, Rate Limiting |
| **12. Background Jobs** | Queue System, Scheduled Tasks, Retries |

---

## 1. Frontend Specifications

### Authentication & Authorization
- [ ] Complete auth flow (Login, Register, Forgot Password, Reset Password, Email Verification)
- [ ] OAuth integration (Google default, extensible to GitHub, Facebook, etc.)
- [ ] JWT/Session token handling with automatic refresh
- [ ] Protected route guards/middleware
- [ ] Role-based access control (RBAC) helpers
- [ ] Persistent auth state (survives refresh)
- [ ] Logout with token invalidation

### State Management
- [ ] Global state solution configured (context/store pattern)
- [ ] Server state management (caching, revalidation, optimistic updates)
- [ ] Persist critical state to storage
- [ ] Devtools integration for debugging

### UI/Component System
- [ ] Component library integrated (Shadcn, Radix, or equivalent)
- [ ] Design tokens/theme system (colors, spacing, typography)
- [ ] Dark/Light mode with system preference detection
- [ ] Responsive breakpoints defined
- [ ] Loading states (skeletons, spinners)
- [ ] Error boundaries with fallback UI
- [ ] Toast/notification system
- [ ] Modal/dialog management
- [ ] Accessible components (ARIA, keyboard navigation)

### Forms & Validation
- [ ] Form library with schema validation
- [ ] Client-side validation with clear error messages
- [ ] Form state persistence (draft saving)
- [ ] File upload handling with preview
- [ ] Input masks and formatting

### API Layer
- [ ] HTTP client configured with interceptors
- [ ] Automatic token attachment
- [ ] Request/response transformation
- [ ] Retry logic with exponential backoff
- [ ] Request cancellation on unmount
- [ ] Offline detection and handling
- [ ] API error normalization

### Performance
- [ ] Code splitting/lazy loading configured
- [ ] Image optimization
- [ ] Bundle analysis tooling
- [ ] Web Vitals tracking
- [ ] Preloading critical assets

### Environment & Configuration
- [ ] Environment variables setup (.env.local, .env.production)
- [ ] Build-time vs runtime config separation
- [ ] Feature flags structure

---

## 2. Backend Specifications

### Authentication System
- [ ] Complete auth implementation (not scaffolding—working code)
- [ ] Password hashing (Argon2/bcrypt with proper cost factor)
- [ ] JWT with access + refresh token pattern
- [ ] Token blacklisting for logout/revocation
- [ ] OAuth2 integration (Google default, extensible)
- [ ] Email verification flow with OTP/magic link
- [ ] Password reset with secure tokens
- [ ] Account lockout after failed attempts
- [ ] Session management (concurrent session limits optional)

### API Design
- [ ] RESTful conventions or GraphQL schema
- [ ] API versioning strategy (/api/v1/)
- [ ] Request validation middleware (Zod/Joi/class-validator)
- [ ] Response envelope standardization `{ success, data, error, meta }`
- [ ] Pagination helpers (cursor-based and offset)
- [ ] Sorting and filtering utilities
- [ ] Partial response (field selection)
- [ ] Bulk operations pattern

### Middleware Stack
- [ ] Request ID generation and propagation
- [ ] Request/response logging
- [ ] CORS configuration (origin whitelist)
- [ ] Body parsing (JSON, URL-encoded, multipart)
- [ ] Compression (gzip/brotli)
- [ ] Request timeout handling
- [ ] Graceful shutdown handler

### Error Handling
- [ ] Global error handler middleware
- [ ] Custom error classes (HttpException, ValidationError, etc.)
- [ ] Error serialization (hide stack in production)
- [ ] Async error wrapper (no try-catch boilerplate)
- [ ] Unhandled rejection/exception handlers
- [ ] Error reporting integration hook

### File Handling
- [ ] File upload middleware (size limits, type validation)
- [ ] Cloud storage abstraction (S3/GCS/Azure Blob)
- [ ] Signed URL generation
- [ ] Image processing hooks (resize, compress)

### Background Processing
- [ ] Job queue system (Bull/BullMQ, Celery, Sidekiq pattern)
- [ ] Scheduled tasks/cron setup
- [ ] Job retry with exponential backoff
- [ ] Dead letter queue handling
- [ ] Job progress tracking

### Email System
- [ ] Email service abstraction
- [ ] Template-based emails (HTML + text fallback)
- [ ] Queue-based sending (don't block requests)
- [ ] Common transactional emails pre-built (welcome, verify, reset)

---

## 3. Database Specifications

### ORM/Query Builder
- [ ] ORM configured with connection pooling
- [ ] Repository pattern or service layer
- [ ] Transaction helpers
- [ ] Soft delete support
- [ ] Timestamps (created_at, updated_at) auto-managed
- [ ] Query logging in development

### Schema & Migrations
- [ ] Migration system configured
- [ ] Initial schema with User + essential tables
- [ ] Migration naming conventions
- [ ] Rollback capability
- [ ] Schema versioning

### Seeding
- [ ] Seed script structure
- [ ] Development seeds (test users, sample data)
- [ ] Production seeds (roles, permissions, system config)
- [ ] Idempotent seed execution

### Performance
- [ ] Connection pooling configured
- [ ] Index recommendations structure
- [ ] Query timeout settings
- [ ] Slow query logging hook

---

## 4. Caching & Session Specifications

### Redis/Cache Layer
- [ ] Redis client configured with connection handling
- [ ] Cache abstraction (get, set, delete, invalidate)
- [ ] TTL strategies per cache type
- [ ] Cache key namespacing
- [ ] Cache warming hooks
- [ ] Distributed lock implementation

### Session Management
- [ ] Session store (Redis-backed for distributed systems)
- [ ] Session cookie configuration (httpOnly, secure, sameSite)
- [ ] Session regeneration on auth state change
- [ ] Session data structure

---

## 5. Security Specifications

### HTTP Security Headers
- [ ] Helmet/security headers middleware
- [ ] Content-Security-Policy
- [ ] X-Frame-Options (clickjacking protection)
- [ ] X-Content-Type-Options
- [ ] Strict-Transport-Security (HSTS)
- [ ] Referrer-Policy
- [ ] Permissions-Policy

### Attack Prevention
- [ ] XSS prevention (input sanitization, output encoding)
- [ ] CSRF protection (tokens for state-changing requests)
- [ ] SQL injection prevention (parameterized queries)
- [ ] NoSQL injection prevention
- [ ] Rate limiting (per IP, per user, per endpoint)
- [ ] Request size limits
- [ ] Path traversal prevention

### Secrets Management
- [ ] Environment variable validation on startup
- [ ] No secrets in code or logs
- [ ] Secret rotation support structure
- [ ] Encryption at rest for sensitive data

### Authentication Security
- [ ] Secure password requirements
- [ ] Brute force protection
- [ ] Secure token generation (crypto-random)
- [ ] Token expiration configuration

---

## 6. Testing Specifications

### Unit Tests
- [ ] Test runner configured
- [ ] Mock/stub utilities
- [ ] Test utilities (factories, fixtures)
- [ ] Coverage reporting
- [ ] Snapshot testing where appropriate

### Integration Tests
- [ ] Test database setup/teardown
- [ ] API testing utilities
- [ ] Authentication helpers for tests
- [ ] Database seeding for tests
- [ ] Test isolation (transactions/cleanup)

### End-to-End Tests
- [ ] E2E framework configured
- [ ] Critical path tests (auth flow, main features)
- [ ] Visual regression testing hooks
- [ ] Cross-browser testing setup

### Contract Tests
- [ ] API contract validation
- [ ] Schema validation tests
- [ ] Breaking change detection

### Boilerplate Verification Tests
- [ ] Auth flow works (register → verify → login → refresh → logout)
- [ ] Protected routes actually protect
- [ ] Rate limiting triggers correctly
- [ ] Error handling returns correct formats
- [ ] Database connectivity verified
- [ ] Redis connectivity verified
- [ ] Environment variables loaded correctly

---

## 7. CI/CD Specifications

### Pipeline Stages
- [ ] Install dependencies
- [ ] Lint and format check
- [ ] Type checking
- [ ] Unit tests
- [ ] Integration tests
- [ ] Build artifacts
- [ ] Security scan (dependency vulnerabilities)
- [ ] Docker image build
- [ ] Push to registry
- [ ] Deploy to staging/production

### Environment Management
- [ ] Environment-specific configurations
- [ ] Secrets injection (not in repo)
- [ ] Feature flag integration
- [ ] Database migration execution in pipeline

### Deployment Strategy
- [ ] Zero-downtime deployment pattern
- [ ] Rollback capability
- [ ] Health check endpoints
- [ ] Deployment notifications

---

## 8. Observability Specifications

### Logging
- [ ] Structured logging (JSON format)
- [ ] Log levels (debug, info, warn, error)
- [ ] Request correlation IDs
- [ ] PII masking in logs
- [ ] Log rotation/retention
- [ ] Centralized logging hook (ELK, CloudWatch, etc.)

### Metrics
- [ ] Application metrics endpoint (/metrics)
- [ ] Request duration histograms
- [ ] Error rate counters
- [ ] Custom business metric helpers
- [ ] Resource usage (memory, CPU)

### Tracing
- [ ] Distributed tracing setup (OpenTelemetry)
- [ ] Trace ID propagation
- [ ] Span creation helpers
- [ ] External service call tracing

### Health Checks
- [ ] Liveness endpoint (/health/live)
- [ ] Readiness endpoint (/health/ready)
- [ ] Dependency health checks (DB, Redis, external services)

### Alerting
- [ ] Alert integration hooks
- [ ] Alert threshold configuration
- [ ] Incident response runbook structure

---

## 9. Infrastructure Specifications

### Docker
- [ ] Multi-stage Dockerfile (dev + prod)
- [ ] .dockerignore optimized
- [ ] Non-root user
- [ ] Health check instruction
- [ ] Proper signal handling (SIGTERM)

### Docker Compose
- [ ] Development compose file
- [ ] All services (app, db, redis, etc.)
- [ ] Volume mounts for development
- [ ] Network configuration
- [ ] Environment file loading

### Reverse Proxy (Optional)
- [ ] Nginx/Traefik configuration
- [ ] SSL termination
- [ ] Request buffering
- [ ] Static file serving

### Orchestration Readiness
- [ ] Kubernetes manifests or Helm charts (optional)
- [ ] Horizontal pod autoscaling config
- [ ] Resource limits defined
- [ ] ConfigMaps/Secrets structure

---

## 10. Developer Experience Specifications

### Code Quality
- [ ] ESLint/Linter configured with strict rules
- [ ] Prettier/Formatter configured
- [ ] EditorConfig for consistency
- [ ] TypeScript/type checking strict mode

### Git Workflow
- [ ] Pre-commit hooks (lint, format, type-check)
- [ ] Commit message conventions (Conventional Commits)
- [ ] Branch naming conventions
- [ ] PR template
- [ ] Issue templates

### Documentation
- [ ] README with setup instructions
- [ ] Architecture decision records (ADR) structure
- [ ] API documentation generation
- [ ] Environment variables documentation
- [ ] Contribution guidelines

### Development Tooling
- [ ] Hot reload configured
- [ ] Debug configuration (VS Code launch.json)
- [ ] Database GUI tool recommendation
- [ ] API client collection (Postman/Insomnia)

### Scripts
- [ ] `dev` - Start development server
- [ ] `build` - Production build
- [ ] `test` - Run all tests
- [ ] `lint` - Check linting
- [ ] `format` - Fix formatting
- [ ] `db:migrate` - Run migrations
- [ ] `db:seed` - Seed database
- [ ] `db:reset` - Reset database
- [ ] `docker:up` - Start Docker services
- [ ] `docker:down` - Stop Docker services

---

## 11. API Documentation Specifications

- [ ] Auto-generated API docs (Swagger/OpenAPI)
- [ ] Interactive API explorer
- [ ] Request/response examples
- [ ] Authentication documentation
- [ ] Error code catalog
- [ ] Changelog/versioning documentation

---

## 12. Project Structure Specifications

### Monorepo vs Polyrepo
- [ ] Clear separation if monorepo (`/apps`, `/packages`)
- [ ] Shared utilities/types package
- [ ] Workspace configuration (npm/yarn/pnpm workspaces)

### Folder Conventions

```
/src
  /config         # App configuration
  /modules        # Feature modules (user, auth, etc.)
  /common         # Shared utilities, decorators, guards
  /middleware     # Express/framework middleware
  /jobs           # Background job handlers
  /templates      # Email/notification templates
/tests
  /unit
  /integration
  /e2e
  /fixtures
/scripts          # Utility scripts (seed, migrate, etc.)
/docs             # Documentation
/docker           # Docker-related files
```

---

## Summary: What Makes This Different from Basic Scaffolds

| Aspect | Basic Scaffold | Production Boilerplate |
|--------|----------------|------------------------|
| Auth | Login form only | Complete flows + OAuth + verification |
| Security | Maybe CORS | Full OWASP hardening |
| Testing | Jest installed | Tests that verify the boilerplate works |
| Logging | console.log | Structured, correlated, production-ready |
| Errors | Generic 500 | Typed errors, proper serialization |
| Deployment | "Run npm start" | Docker + CI/CD + health checks |
| Documentation | README | Architecture, API docs, runbooks |

---

## Feature Legend

- [x] = **Pre-built and working** — Just configure and use
- [ ] = **Structure/hooks provided** — Extend as needed
- Template provided = **Code template exists** — Fill in credentials

---

## Quick Reference: What You Get Out of the Box

### Working Endpoints (Backend)
```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh
POST   /api/v1/auth/forgot-password
POST   /api/v1/auth/reset-password
POST   /api/v1/auth/verify-email
POST   /api/v1/auth/resend-verification
GET    /api/v1/auth/google
GET    /api/v1/auth/google/callback
GET    /api/v1/auth/sessions
DELETE /api/v1/auth/sessions/:id
DELETE /api/v1/auth/sessions (logout all)

GET    /api/v1/users/me
PATCH  /api/v1/users/me
DELETE /api/v1/users/me
POST   /api/v1/users/me/avatar
DELETE /api/v1/users/me/avatar
PATCH  /api/v1/users/me/password
PATCH  /api/v1/users/me/email

GET    /api/v1/notifications
PATCH  /api/v1/notifications/:id/read
PATCH  /api/v1/notifications/read-all
DELETE /api/v1/notifications/:id

POST   /api/v1/files/upload
GET    /api/v1/files/:id
DELETE /api/v1/files/:id

GET    /health
GET    /health/live
GET    /health/ready
```

### Working Pages (Frontend)
```
/login
/register
/forgot-password
/reset-password
/verify-email
/oauth/callback
/dashboard (protected)
/profile (protected)
/settings (protected)
/404
/error
```

### Database Tables (Pre-Created)
```
users
sessions
refresh_tokens
password_reset_tokens
email_verification_tokens
roles
permissions
role_permissions
user_roles
notifications
files
audit_logs
```

---

## Usage Instructions

1. **Generate the boilerplate** using this specification
2. **Configure environment variables** (database, Redis, OAuth, email)
3. **Run migrations** to create database schema
4. **Run seeds** to create default roles and admin user
5. **Run tests** to verify everything works
6. **Start development** — auth and user management ready to go
7. **Extend** by adding your business logic modules

---

*This specification is framework-agnostic. The AI should adapt patterns and implementations to match the chosen tech stack while maintaining all specified functionality.*
