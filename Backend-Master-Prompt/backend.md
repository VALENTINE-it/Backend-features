## MASTER PROMPT: BACKEND IMPLEMENTATION

### ROLE

Act as a **Senior Go Backend Engineer, Software Architect, Database Engineer, DevOps Engineer, and Cybersecurity Specialist**.

Your responsibility is to inspect, improve, secure, optimize, and implement the backend of my existing personal portfolio website.

The objective is to build a **production-ready, secure, maintainable, high-performance backend designed to scale toward 10,000+ concurrent users**, subject to successful load testing and appropriate infrastructure.

**IMPORTANT INSTRUCTIONS:**

- This is an existing project, not a new application.
- Inspect the entire backend before implementing changes.
- Read all existing documentation, including `ARCHITECTURE.md`, `PRD.md`, `SECURITY.md`, `DECISIONS.md`, `TASKS.md`, and `README.md`, where available.
- Do not unnecessarily recreate existing functionality.
- Preserve the current frontend design and layout.
- Do not change existing API contracts unless absolutely necessary.
- Use Go's standard `net/http` package.
- Do not introduce Fiber, Gin, Echo, or another HTTP framework.
- Use SQLite as the primary database.
- Do not migrate to PostgreSQL without explicit approval.
- Follow idiomatic Go practices and clean architecture principles.
- Implement real, working functionality rather than placeholder code.
- Keep dependencies minimal.
- Do not introduce unnecessary microservices.
- Ensure compatibility with the Go version specified in `go.mod`.
- Add tests for all critical functionality.

---

# PHASE 1: BACKEND AUDIT AND ARCHITECTURE

Before writing any code, perform a complete backend audit.

### 1.1 Inspect Existing Implementation

Analyze:

1. Current directory structure.
2. HTTP server implementation.
3. API routes and handlers.
4. Database connection and schema.
5. Authentication and authorization.
6. Existing middleware.
7. Environment configuration.
8. Error handling.
9. Logging.
10. Testing coverage.
11. Deployment configuration.
12. Security vulnerabilities.
13. Performance bottlenecks.
14. Dependencies and their versions.

Identify functionality that is already implemented correctly.

Do not duplicate it.

### 1.2 Architecture Requirements

Use a modular monolithic architecture with separation of concerns.

Follow this structure where appropriate:

```text
backend/
│
├── cmd/
│   └── server/
│       └── main.go
│
├── internal/
│   ├── config/
│   │   └── config.go
│   │
│   ├── database/
│   │   ├── sqlite.go
│   │   └── migrations/
│   │
│   ├── handlers/
│   │   ├── auth.go
│   │   ├── admin.go
│   │   ├── projects.go
│   │   ├── skills.go
│   │   ├── contact.go
│   │   └── health.go
│   │
│   ├── middleware/
│   │   ├── authentication.go
│   │   ├── authorization.go
│   │   ├── cors.go
│   │   ├── csrf.go
│   │   ├── rate_limit.go
│   │   ├── security_headers.go
│   │   ├── logging.go
│   │   └── recovery.go
│   │
│   ├── models/
│   │   ├── admin.go
│   │   ├── project.go
│   │   ├── skill.go
│   │   └── message.go
│   │
│   ├── repository/
│   │   ├── admin_repository.go
│   │   ├── project_repository.go
│   │   └── message_repository.go
│   │
│   ├── services/
│   │   ├── auth_service.go
│   │   ├── project_service.go
│   │   ├── contact_service.go
│   │   └── email_service.go
│   │
│   ├── sessions/
│   │   └── session.go
│   │
│   ├── cache/
│   │   ├── cache.go
│   │   └── memory.go
│   │
│   ├── monitoring/
│   │   ├── metrics.go
│   │   └── health.go
│   │
│   └── utils/
│       ├── validation.go
│       ├── response.go
│       └── errors.go
│
├── tests/
│
├── docs/
│
├── .env.example
├── .gitignore
├── Dockerfile
├── go.mod
├── go.sum
└── README.md
```

Adapt this structure to the existing project instead of blindly moving files.

---

# PHASE 2: HTTP SERVER OPTIMIZATION

Improve the existing Go HTTP server.

### Requirements

Implement:

- HTTP request timeouts.
- Connection idle timeouts.
- Header size restrictions.
- Request body size limits.
- Graceful shutdown.
- Context cancellation.
- Structured error handling.
- HTTP health endpoints.
- Request IDs.
- Panic recovery.
- Connection management.

Use the standard `net/http` package.

### Server Configuration

Use configurable defaults such as:

```text
PORT=8080

READ_TIMEOUT=15s
WRITE_TIMEOUT=15s
IDLE_TIMEOUT=60s
READ_HEADER_TIMEOUT=5s

MAX_HEADER_BYTES=1048576
MAX_REQUEST_BODY_BYTES=1048576
```

Ensure these values can be changed through environment configuration.

Handle long-running operations appropriately.

### Graceful Shutdown

When the application receives SIGTERM or SIGINT:

1. Stop accepting new connections.
2. Allow existing requests to complete within a deadline.
3. Stop background workers.
4. Flush buffered logs where applicable.
5. Close database connections.
6. Release resources.
7. Exit safely.

Do not terminate active operations unnecessarily.

---

# PHASE 3: DATABASE OPTIMIZATION — SQLITE

SQLite is the primary database.

Do not replace it.

### Database Requirements

Implement:

- Secure database initialization.
- Database migrations.
- Foreign key enforcement.
- WAL mode where appropriate.
- Busy timeouts.
- Prepared or parameterized queries.
- Transaction management.
- Database indexing.
- Query optimization.
- Connection pool configuration.
- Database integrity checks.
- Automatic backups.
- Backup restoration procedures.

### SQLite Configuration

Evaluate settings such as:

```sql
PRAGMA journal_mode=WAL;

PRAGMA foreign_keys=ON;

PRAGMA busy_timeout=5000;
```

Configure connection limits based on the application's workload and SQLite driver's behavior.

Remember that some SQLite PRAGMAs are connection-specific.

### Database Security

Ensure:

- SQL injection is prevented.
- Database files are inaccessible through public HTTP routes.
- Database file permissions are restricted.
- Sensitive information is protected.
- Secrets are never stored in plaintext.
- Migrations are tracked.
- Transactions roll back correctly after errors.

### Database Performance

Implement indexes for frequently queried fields.

Use pagination where necessary.

Avoid unnecessary database queries.

Monitor slow operations.

Do not create excessive indexes.

### Database Backups

Implement scheduled backups using a SQLite-safe mechanism.

Requirements:

- Configurable backup frequency.
- Backup retention.
- Secure backup storage.
- Error logging.
- Restoration documentation.
- Periodic restoration testing.

Never rely on an unsafe copy of an actively written SQLite database.

---

# PHASE 4: ADMIN AUTHENTICATION AND AUTHORIZATION

My portfolio has an admin dashboard.

Public visitors do not need accounts.

Only authorized administrators should access administrative functionality.

### Authentication Requirements

Implement:

1. Secure admin login.
2. Admin logout.
3. Password hashing using Argon2id or bcrypt.
4. Secure session generation.
5. Server-side session storage.
6. HTTP-only cookies.
7. Secure cookie attributes.
8. Session expiration.
9. Session invalidation.
10. Session rotation after authentication.
11. Login attempt throttling.
12. Authentication event logging.

### Session Cookies

Use secure cookie settings:

```text
HttpOnly=true

Secure=true

SameSite=Lax

Path=/
```

Use appropriate production settings and account for local development.

Generate session identifiers using cryptographically secure randomness.

Never use predictable session identifiers.

Store only an opaque session identifier in the cookie.

Store session information server-side.

### Authorization

Protect administrative routes.

Examples:

```text
POST   /api/admin/login

POST   /api/admin/logout

GET    /api/admin/dashboard

POST   /api/admin/projects

PUT    /api/admin/projects/:id

DELETE /api/admin/projects/:id
```

Unauthenticated users must not access protected resources.

Validate authorization on every protected request.

### Admin Registration

Do not allow unrestricted public admin registration.

Implement an administrator provisioning mechanism.

Possible approaches:

- Initial administrator setup through a secure CLI command.
- Administrator creation by an existing authorized administrator.

Disable public administrator registration unless explicitly required.

---

# PHASE 5: COMPLETE BACKEND SECURITY

Implement defense-in-depth security.

### 5.1 Rate Limiting

Introduce rate limiting for all public endpoints.

Use configurable limits.

Suggested starting values:

| Endpoint | Rate Limit |
|---|---|
| Public API | 100 requests/minute/IP |
| Admin login | 5 attempts/minute/IP |
| Contact form | 3 submissions/minute/IP |
| Admin API | 60 requests/minute/user |

Use an appropriate rate-limiting algorithm, such as token bucket.

Ensure the implementation:

- Is concurrency-safe.
- Prevents unbounded memory growth.
- Supports expiration of inactive limiter entries.
- Returns HTTP 429 when limits are exceeded.
- Includes `Retry-After` where appropriate.
- Handles proxy headers securely.

Do not trust arbitrary `X-Forwarded-For` values.

Only trust forwarded headers from explicitly configured reverse proxies.

### 5.2 DDoS Mitigation

Implement application-level protections against abusive traffic.

Include:

- Request size restrictions.
- HTTP timeouts.
- Connection limits where appropriate.
- Rate limiting.
- Resource exhaustion protection.
- Bounded background queues.
- Protection against expensive repeated operations.

Prepare the backend to operate behind Cloudflare or another trusted reverse proxy.

Document:

- Cloudflare integration.
- WAF configuration.
- Edge rate limiting.
- Origin server protection.
- Firewall rules.
- HTTPS enforcement.

Do not claim that application-level middleware alone can prevent large distributed DDoS attacks.

### 5.3 Input Validation

Validate every incoming request.

Check:

- Required fields.
- Data types.
- String lengths.
- Email addresses.
- Allowed values.
- File sizes.
- Uploaded file types.
- JSON structure.

Reject malformed input.

### 5.4 SQL Injection Prevention

Use parameterized queries.

Never construct SQL queries by directly concatenating untrusted user input.

### 5.5 XSS Protection

Prevent stored and reflected cross-site scripting.

Treat user-provided content as untrusted.

Use appropriate output encoding and safe rendering.

Coordinate with the React frontend to avoid unsafe HTML injection.

### 5.6 CSRF Protection

Implement CSRF protection for authenticated state-changing requests.

Consider:

- CSRF tokens.
- Origin verification.
- SameSite cookie settings.
- Correct CORS configuration.

Do not rely on SameSite cookies as the only defense.

### 5.7 Security Headers

Configure appropriate HTTP security headers.

Include:

```text
X-Content-Type-Options: nosniff

Referrer-Policy: strict-origin-when-cross-origin

Strict-Transport-Security: max-age=31536000

Content-Security-Policy: ...
```

Use a carefully designed Content Security Policy that is compatible with the frontend.

Only enable HSTS when HTTPS is correctly configured.

### 5.8 Secrets Management

Never hardcode:

- Passwords.
- API keys.
- Session secrets.
- Database credentials.
- Email service credentials.

Use environment variables or a secure secret manager.

Ensure `.env` files are excluded from Git.

---

# PHASE 6: CACHING AND PERFORMANCE

Implement an efficient caching strategy.

### Primary Cache

Start with in-memory caching for suitable public data.

Potential cache targets:

- Projects.
- Skills.
- Public profile information.
- Portfolio metadata.

### Requirements

Implement:

- Cache expiration.
- Cache invalidation.
- Thread-safe access.
- Memory limits.
- Cache hit and miss metrics.
- Cache cleanup.

When an administrator updates a project, invalidate the affected cached data.

### HTTP Caching

Support appropriate HTTP caching headers for public resources.

Consider:

```text
Cache-Control

ETag

Last-Modified
```

Do not cache private admin responses in shared caches.

### Redis Compatibility

Design a cache interface that allows Redis to be introduced later.

However:

**Do not make Redis a mandatory dependency for the current portfolio.**

Use in-memory caching unless performance measurements demonstrate the need for a distributed cache.

---

# PHASE 7: SCALABILITY AND HIGH CONCURRENCY

Design the backend so it can scale toward 10,000+ concurrent users.

### Requirements

- Avoid unnecessary global mutable state.
- Ensure shared structures are concurrency-safe.
- Prevent goroutine leaks.
- Use context cancellation.
- Avoid blocking operations where possible.
- Limit expensive concurrent operations.
- Optimize database access.
- Implement pagination.
- Use efficient JSON serialization.
- Prevent unbounded memory usage.
- Apply backpressure when overloaded.

### Horizontal Scaling Readiness

Prepare the application for future deployment behind a load balancer.

The architecture should support:

```text
                CLOUDFLARE
                    |
                    ▼
               LOAD BALANCER
                    |
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       GO APP 1  GO APP 2  GO APP 3
          |         |         |
          └─────────┼─────────┘
                    |
              SHARED SERVICES
```

**Important SQLite limitation:**

Do not configure multiple servers to write directly to the same SQLite database through an unsupported shared network filesystem.

For now, optimize the single-instance architecture.

Document how the application could later migrate to PostgreSQL and a shared session/cache store if horizontal scaling becomes necessary.

Do not implement that migration without approval.

---

# PHASE 8: ERROR HANDLING

Implement centralized error handling.

Use consistent JSON responses.

Example:

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred"
  }
}
```

Requirements:

- Return appropriate HTTP status codes.
- Log internal errors.
- Do not expose stack traces.
- Do not expose database errors.
- Handle invalid JSON.
- Handle missing resources.
- Handle authentication failures.
- Recover safely from panics.

Use request IDs to correlate errors with logs.

---

# PHASE 9: LOGGING AND MONITORING

Implement structured logging using Go's `log/slog`.

### Logging Requirements

Record:

- Request method.
- Request path.
- Response status.
- Request duration.
- Request ID.
- Application errors.
- Database failures.
- Failed authentication attempts.
- Administrative actions.

Never log passwords, session tokens, or sensitive credentials.

### Monitoring Requirements

Implement:

```text
GET /health/live

GET /health/ready
```

The liveness endpoint should indicate whether the process is running.

The readiness endpoint should check whether the application can serve requests, including critical dependencies.

### Metrics

Track:

- Requests per second.
- Request duration.
- Error rate.
- Memory consumption.
- CPU usage.
- Database latency.
- Active requests.
- Cache hit rate.
- Rate-limit rejections.

Use Prometheus-compatible metrics where practical.

Protect internal monitoring endpoints from unauthorized public access.

---

# PHASE 10: BACKGROUND TASKS

Implement background processing only where useful.

Potential tasks:

- Sending contact form notifications.
- Cleaning expired sessions.
- Performing scheduled backups.
- Removing expired cache entries.
- Performing database maintenance.

### Requirements

- Support graceful shutdown.
- Use bounded concurrency.
- Handle failures.
- Support retries where appropriate.
- Avoid duplicate processing.
- Log task execution.
- Prevent goroutine leaks.

For important jobs, use durable storage so tasks are not lost when the application restarts.

Do not introduce Redis or a complex message broker unnecessarily.

---

# PHASE 11: CONTACT FORM SECURITY

My portfolio includes a contact page.

Implement secure contact message handling.

### Requirements

- Validate names.
- Validate email addresses.
- Validate message content.
- Limit input length.
- Prevent SQL injection.
- Prevent XSS.
- Rate-limit submissions.
- Prevent email header injection.
- Avoid exposing internal email credentials.
- Store messages safely.
- Provide consistent API responses.

Add spam protection where appropriate.

If email notifications are enabled, use environment-based credentials.

Do not expose private contact messages through public API endpoints.

---

# PHASE 12: API DESIGN AND DOCUMENTATION

Use RESTful API design.

Preserve existing routes where possible.

Example API structure:

```text
PUBLIC ROUTES

GET    /api/projects
GET    /api/projects/{id}
GET    /api/skills
GET    /api/profile
POST   /api/contact


ADMIN ROUTES

POST   /api/admin/login
POST   /api/admin/logout

GET    /api/admin/dashboard

POST   /api/admin/projects
PUT    /api/admin/projects/{id}
DELETE /api/admin/projects/{id}

POST   /api/admin/skills
PUT    /api/admin/skills/{id}
DELETE /api/admin/skills/{id}


SYSTEM ROUTES

GET    /health/live
GET    /health/ready
```

Implement only routes appropriate to the existing application's functionality.

Document:

- HTTP methods.
- Request parameters.
- Request bodies.
- Authentication requirements.
- Success responses.
- Error responses.
- Rate limits.

Use OpenAPI documentation where practical.

---

# PHASE 13: AUTOMATED TESTING

Implement comprehensive backend tests.

### Unit Tests

Test:

- Validation functions.
- Authentication logic.
- Authorization rules.
- Password hashing.
- Session management.
- Cache behavior.
- Rate limiting.

### Integration Tests

Test:

- Database queries.
- Database migrations.
- API handlers.
- Login and logout.
- Protected routes.
- Contact submissions.

### Security Tests

Verify:

- SQL injection attempts are rejected or safely handled.
- Unauthorized access is blocked.
- CSRF protections work.
- Rate limits are enforced.
- Sessions expire correctly.
- Invalid cookies are rejected.
- Malformed JSON is handled safely.
- Sensitive information is not exposed.

### Concurrency Tests

Use:

```bash
go test ./...
```

And:

```bash
go test -race ./...
```

Resolve detected data races.

### Load Testing

Use k6 to evaluate:

- 100 concurrent virtual users.
- 500 concurrent virtual users.
- 1,000 concurrent virtual users.
- 5,000 concurrent virtual users.
- 10,000 concurrent virtual users.

Increase load gradually and only in an authorized test environment.

Measure:

- Requests per second.
- P50 latency.
- P95 latency.
- P99 latency.
- Error rate.
- CPU usage.
- Memory usage.
- Database performance.

Do not claim support for 10,000 concurrent users without test evidence.

Document the workload, hardware, test duration, results, and bottlenecks.

---

# PHASE 14: DEPLOYMENT AND PRODUCTION CONFIGURATION

Prepare the backend for production deployment.

### Docker

Create a secure multi-stage Dockerfile.

Requirements:

- Compile the Go application.
- Use a minimal runtime image.
- Run as a non-root user.
- Expose only necessary ports.
- Keep the image small.
- Avoid embedding secrets.
- Configure persistent storage for SQLite.

### Reverse Proxy

Prepare an Nginx configuration that supports:

- HTTPS termination.
- Reverse proxying.
- Appropriate timeouts.
- Request body limits.
- Security headers.
- Trusted proxy configuration.
- Access logging.

### Cloudflare

Document how to configure:

- DNS.
- HTTPS.
- WAF.
- DDoS mitigation.
- Bot protection.
- Edge rate limiting.
- Origin firewall rules.

### CI/CD

Create a GitHub Actions workflow that:

1. Checks formatting.
2. Runs static analysis.
3. Runs unit tests.
4. Runs integration tests.
5. Runs race detection where supported.
6. Builds the application.
7. Scans dependencies for known vulnerabilities.
8. Produces a deployable artifact.

Do not deploy automatically without explicit configuration and approval.

---

# PHASE 15: FRONTEND INTEGRATION

My frontend uses React + Vite.

Ensure that all backend changes remain compatible with the frontend.

### Requirements

- Preserve existing frontend functionality.
- Configure CORS correctly.
- Support secure cookie authentication.
- Ensure API responses are consistent.
- Handle authentication failures correctly.
- Avoid exposing secrets in frontend environment variables.
- Ensure admin routes are protected server-side.
- Avoid breaking existing React components.

Do not redesign the frontend.

Only modify frontend code where required for backend integration.

---

# PHASE 16: DOCUMENTATION

After implementation, update the project's existing documentation.

Include:

**ARCHITECTURE.md**
- Updated backend architecture.
- Component responsibilities.
- Data flow.
- Authentication flow.
- Caching architecture.
- Deployment architecture.

**SECURITY.md**
- Authentication protections.
- Authorization rules.
- Rate limiting.
- CSRF protection.
- DDoS mitigation.
- Secure configuration.
- Threat considerations.

**README.md**
- Installation instructions.
- Environment setup.
- Database setup.
- Running the backend.
- Testing.
- Deployment.

**DECISIONS.md**
- Architectural decisions.
- Technology choices.
- Tradeoffs.
- Reasons for retaining SQLite.

**TASKS.md**
- Completed tasks.
- Remaining tasks.
- Future improvements.

Do not overwrite useful existing documentation.

---

# PHASE 17: FINAL VERIFICATION

Before declaring the backend complete:

1. Confirm the application compiles successfully.
2. Run all automated tests.
3. Verify database migrations.
4. Test authentication and logout.
5. Test authorization.
6. Test session expiration.
7. Test rate limiting.
8. Test CSRF protection.
9. Verify input validation.
10. Verify caching behavior.
11. Test graceful shutdown.
12. Verify backup and restoration.
13. Check for data races.
14. Confirm frontend integration.
15. Verify environment configuration.
16. Confirm no secrets are committed.
17. Verify production security settings.
18. Document remaining limitations.

### Final Output

Provide a clear implementation report containing:

```text
BACKEND IMPLEMENTATION REPORT

1. Existing Backend Assessment
2. Security Issues Identified
3. Architecture Improvements
4. Features Implemented
5. Database Optimizations
6. Authentication Implementation
7. Rate Limiting and DDoS Mitigation
8. Caching Implementation
9. Performance Improvements
10. Tests Executed and Results
11. Deployment Configuration
12. Known Limitations
13. Future Scaling Recommendations
```

Clearly distinguish between:

- Implemented and verified.
- Implemented but not tested.
- Planned for future implementation.
- Not applicable to the current portfolio.

Do not report incomplete functionality as complete.

---

## FINAL IMPLEMENTATION RULES

**Follow these rules throughout the project:**

1. **Do not rebuild my portfolio from scratch.**
2. Do not change the frontend design.
3. Do not remove existing working functionality.
4. Do not introduce unnecessary dependencies.
5. Do not change SQLite to PostgreSQL.
6. Do not use Fiber or another Go web framework.
7. Do not create unnecessary microservices.
8. Prioritize security and performance.
9. Maintain compatibility with existing APIs.
10. Write clean, maintainable, production-quality Go code.
11. Implement features incrementally.
12. Test each major change before continuing.
13. Fix errors instead of bypassing them.
14. Do not hardcode credentials.
15. Document all important changes.
16. Do not introduce breaking changes without explaining them.
17. Do not claim scalability targets without benchmarking.
18. Preserve the existing architecture wherever it is already appropriate.

**Begin by auditing the existing backend. Present the audit findings and implementation plan, then implement the improvements in phases.**

---

### One Important Recommendation

Because your portfolio uses **Go + SQLite**, I recommend keeping the initial production deployment simple:

**React + Vite → Cloudflare → Nginx → Go Backend → SQLite**

This gives you a strong foundation without requiring Redis, PostgreSQL, or multiple servers immediately.

The architecture should make it possible to add those components later, but you should introduce them only when actual traffic and performance measurements justify the additional complexity.

**The goal is not just to create a backend that works. It's to create one that is secure, reliable, maintainable, and ready to grow.**