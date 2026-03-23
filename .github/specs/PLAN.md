# Technical Implementation Plan: User Authentication System

<!--
  FILE: .github/specs/PLAN.md
  PURPOSE: Spec-Driven Development (SDD) — Technical Implementation Plan
  SPEC-KIT: https://github.com/github/spec-kit

  WHAT IS THIS FILE?
    The PLAN is the technical complement to the SPEC. Where the SPEC describes
    WHAT to build (user stories, acceptance criteria), the PLAN describes HOW
    to build it (technology choices, architecture, data models, API contracts).

    The PLAN answers:
      - What technology stack will we use? Why?
      - How will the system be architected?
      - What does the data model look like?
      - What are the API contracts?
      - How will we test this?
      - What are the risks and mitigations?

  HOW TO CREATE THIS FILE:
    Automated: After creating SPEC.md, use the `/speckit.plan` command:
      /speckit.plan Node.js/Express backend, PostgreSQL database, JWT sessions
      stored in HttpOnly cookies, GitHub OAuth for social login.

    The command will:
      1. Read the existing SPEC.md for requirements context
      2. Check the CONSTITUTION.md for architectural constraints
      3. Generate this PLAN.md with your technology choices translated into
         detailed technical architecture
      4. Optionally generate supporting files:
         - data-model.md (entity schemas)
         - contracts/ (API endpoint specifications)
         - research.md (library comparisons, benchmarks)

    Manual: Write it yourself following the structure below.

  WHAT MAKES A GOOD PLAN?
    ✅ Every technology choice has a documented rationale
    ✅ Every architectural decision traces back to a spec requirement
    ✅ Data models are defined with field names, types, and constraints
    ✅ API contracts specify request/response shapes and error codes
    ✅ Test strategy maps back to acceptance criteria
    ✅ Risks and mitigations are explicitly documented
    ❌ Don't repeat what the SPEC already says — reference it instead
    ❌ Don't include implementation code — that's for the TASKS/IMPLEMENT phase

  SDD WORKFLOW POSITION:
    CONSTITUTION → SPEC → [PLAN] → TASKS → IMPLEMENT
                          ↑ (this file)
-->

**Specification Reference**: SPEC.md (Feature 001: User Authentication System)
**Status**: Draft
**Last Updated**: 2026-03-23

---

## Technology Decisions

<!--
  DEVELOPMENT NOTE: For each technology choice, document the alternatives
  considered and the rationale for the selection. This creates an audit trail
  that prevents the same debates from recurring and helps future team members
  (and AI agents) understand the "why" behind the choices.
-->

### Backend Framework

**Selected**: Node.js with Express.js
**Rationale**: Consistent with our existing API services (see CONSTITUTION.md).
Mature ecosystem with excellent security libraries for authentication.

**Alternatives Considered**:
| Option | Pros | Cons | Decision |
|---|---|---|---|
| Express.js | Minimal, flexible, battle-tested | Low-level, requires more setup | ✅ Selected |
| NestJS | Opinionated, great TypeScript support | Heavier, longer learning curve | Rejected |
| Fastify | Faster than Express | Smaller ecosystem | Rejected |

### Database

**Selected**: PostgreSQL
**Rationale**: Existing infrastructure (CONSTITUTION.md requires new services
to use the approved data store). ACID compliance is essential for auth data.

### Session Management

**Selected**: JWT tokens stored in HttpOnly cookies
**Rationale**: HttpOnly cookies prevent XSS theft. JWTs are stateless and
scale horizontally. Cookies are automatically sent by browsers, simplifying
client-side code.

**Alternatives Considered**:
| Option | Pros | Cons | Decision |
|---|---|---|---|
| HttpOnly cookies + JWT | XSS-safe, scalable | CSRF risk (mitigated by SameSite) | ✅ Selected |
| localStorage + JWT | Simple client code | Vulnerable to XSS attacks | Rejected |
| Server-side sessions (Redis) | Instant revocation | Requires Redis infrastructure | Future option |

### Password Hashing

**Selected**: Argon2id
**Rationale**: OWASP-recommended as of 2023. Winner of the Password Hashing
Competition. More resistant to GPU attacks than bcrypt.
**Cost factor**: 3 (time), 64MB (memory), 4 (parallelism) — adjust for hardware.

### Social Login

**Selected**: GitHub OAuth 2.0 via `passport-github2`
**Rationale**: GitHub is the primary social login provider per SPEC.md US-002.
Passport.js provides a consistent interface if more providers are added later.

---

## System Architecture

<!--
  DEVELOPMENT NOTE: An architecture diagram (even ASCII art) helps everyone
  understand the components and their relationships before implementation starts.
-->

```
Browser
  │
  │  HTTPS
  ▼
┌─────────────────────────────────────────────────────────┐
│  Express.js API Server                                  │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Auth Routes │  │  Middleware  │  │  Rate Limiter │  │
│  │  /auth/*    │  │  JWT Verify  │  │  (login only) │  │
│  └──────┬──────┘  └──────────────┘  └───────────────┘  │
│         │                                               │
│  ┌──────▼──────────────────┐                           │
│  │  Auth Service            │                           │
│  │  - register()            │                           │
│  │  - login()               │                           │
│  │  - resetPassword()       │                           │
│  └──────┬───────────────────┘                           │
│         │                                               │
└─────────│───────────────────────────────────────────────┘
          │
     ┌────┴────────────────────┐
     │                         │
     ▼                         ▼
┌─────────┐              ┌──────────────┐
│PostgreSQL│              │Email Service │
│ users    │              │ (SMTP/SES)   │
│ sessions │              └──────────────┘
└─────────┘
          │
          ▼
┌──────────────────────┐
│  GitHub OAuth         │
│  api.github.com/login │
└──────────────────────┘
```

---

## Data Model

<!--
  DEVELOPMENT NOTE: Define the database schema with enough detail that a
  developer could implement it without guessing. Include data types, nullability,
  default values, indexes, and foreign key relationships.
-->

### `users` table

| Column | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PRIMARY KEY, DEFAULT gen_random_uuid() | |
| `email` | VARCHAR(255) | UNIQUE, NOT NULL | Lowercased before storage |
| `password_hash` | VARCHAR(255) | NULL | NULL for social-only accounts |
| `email_verified` | BOOLEAN | NOT NULL, DEFAULT FALSE | |
| `github_id` | VARCHAR(100) | UNIQUE, NULL | Set when using GitHub OAuth |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | |
| `updated_at` | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Updated by trigger |
| `locked_until` | TIMESTAMPTZ | NULL | Set after 5 failed login attempts |
| `failed_login_count` | INTEGER | NOT NULL, DEFAULT 0 | Reset on successful login |

**Indexes**:
- `idx_users_email` ON `users(email)` — for login lookups
- `idx_users_github_id` ON `users(github_id)` WHERE `github_id IS NOT NULL`

### `password_reset_tokens` table

| Column | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PRIMARY KEY | |
| `user_id` | UUID | FOREIGN KEY → users(id) ON DELETE CASCADE | |
| `token_hash` | VARCHAR(255) | NOT NULL | Store hash, not raw token |
| `expires_at` | TIMESTAMPTZ | NOT NULL | NOW() + 1 hour |
| `used_at` | TIMESTAMPTZ | NULL | Set when token is consumed |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | |

**Indexes**:
- `idx_reset_tokens_token_hash` ON `password_reset_tokens(token_hash)`
- `idx_reset_tokens_user_id` ON `password_reset_tokens(user_id)`

---

## API Contracts

<!--
  DEVELOPMENT NOTE: Define every API endpoint with its HTTP method, path,
  request body, response shape, and possible error codes. This becomes the
  contract between frontend and backend teams and enables parallel development.
-->

### POST /auth/register

**Description**: Create a new user account
**Auth required**: No

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Response 201**:
```json
{
  "message": "Account created. Please check your email to verify your address.",
  "userId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Errors**:
| Status | Code | Meaning |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Missing/invalid fields |
| 409 | `EMAIL_IN_USE` | Email already registered |
| 422 | `WEAK_PASSWORD` | Password doesn't meet requirements |

### POST /auth/login

**Description**: Authenticate a user and issue a session cookie
**Auth required**: No
**Rate limit**: 5 attempts per IP per 15 minutes

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "rememberMe": true
}
```

**Response 200**:
```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "emailVerified": true
  }
}
```
Sets cookie: `session=<jwt>; HttpOnly; Secure; SameSite=Lax`
Cookie max-age: 30 days if `rememberMe: true`, session cookie if `false`

**Errors**:
| Status | Code | Meaning |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Missing fields |
| 401 | `INVALID_CREDENTIALS` | Wrong email or password (generic — see AC-006) |
| 423 | `ACCOUNT_LOCKED` | Too many failed attempts |

### POST /auth/logout

**Description**: Invalidate the current session
**Auth required**: Yes (session cookie)

**Response 200**:
```json
{ "message": "Logged out successfully." }
```
Clears the session cookie.

### POST /auth/forgot-password

**Description**: Request a password reset email
**Auth required**: No
**Rate limit**: 3 requests per email per hour

**Request Body**:
```json
{ "email": "user@example.com" }
```

**Response 200** (always, regardless of whether email exists — see AC-011):
```json
{
  "message": "If an account with that email exists, you will receive a reset link shortly."
}
```

### POST /auth/reset-password

**Description**: Set a new password using a reset token
**Auth required**: No

**Request Body**:
```json
{
  "token": "<reset-token-from-email>",
  "newPassword": "NewSecurePassword456!"
}
```

**Response 200**:
```json
{ "message": "Password reset successfully. You are now logged in." }
```
Sets new session cookie (user is automatically logged in after reset).

**Errors**:
| Status | Code | Meaning |
|---|---|---|
| 400 | `INVALID_TOKEN` | Token not found, already used, or expired |
| 422 | `WEAK_PASSWORD` | New password doesn't meet requirements |

### GET /auth/github

**Description**: Initiate GitHub OAuth flow (redirects to GitHub)
**Auth required**: No

### GET /auth/github/callback

**Description**: Handle GitHub OAuth callback
**Auth required**: No (GitHub provides temporary code)

Redirects to dashboard on success, to login page with error on failure.

---

## Security Implementation Details

<!--
  DEVELOPMENT NOTE: Document security-sensitive implementation details here
  so they are reviewed before implementation, not discovered in code review.
-->

### Password Requirements

Passwords must:
- Be at least 8 characters long
- Contain at least one uppercase letter
- Contain at least one lowercase letter
- Contain at least one number
- Not be in a list of the 1,000 most common passwords

### CSRF Protection

- Use `SameSite=Lax` on session cookies (protects against most CSRF attacks)
- For state-changing requests from JavaScript (fetch/XHR): require a CSRF token
  header that matches a value stored in the session

### Rate Limiting Implementation

- Use `express-rate-limit` middleware
- Apply to `/auth/login`, `/auth/forgot-password` endpoints
- Store rate limit state in memory (or Redis in multi-instance deployments)
- Return `Retry-After` header when limit is exceeded

---

## Testing Strategy

<!--
  DEVELOPMENT NOTE: Map your test strategy back to the acceptance criteria
  in SPEC.md. This ensures every AC has a corresponding test.
-->

### Unit Tests (Jest)

| Test | Maps to AC |
|---|---|
| `AuthService.register()` — valid input | AC-001 |
| `AuthService.register()` — duplicate email | AC-002 |
| `AuthService.register()` — weak password | AC-003 |
| `AuthService.login()` — valid credentials | AC-005 |
| `AuthService.login()` — invalid credentials | AC-006 |
| `AuthService.login()` — account lockout | AC-009 |
| `AuthService.resetPassword()` — valid token | AC-012 |
| `AuthService.resetPassword()` — expired token | AC-013 |

### Integration Tests (Supertest + test database)

| Test | Maps to AC |
|---|---|
| POST /auth/register — success + email sent | AC-001 |
| POST /auth/register — sets remember-me cookie | AC-007, AC-008 |
| POST /auth/login — rate limiting enforced | AC-009 |
| POST /auth/forgot-password — consistent response | AC-011 |
| Full reset password flow | AC-010, AC-012, AC-013 |

---

## Risks and Mitigations

<!--
  DEVELOPMENT NOTE: Identifying risks early is one of the most valuable
  things a technical plan can do. Document each risk, its likelihood,
  impact, and the plan to mitigate it.
-->

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| GitHub OAuth API changes | Low | Medium | Pin to specific GitHub OAuth API version; monitor changelog |
| Email deliverability issues | Medium | High | Use a reputable email service (SES/SendGrid); configure SPF/DKIM |
| Argon2 performance too slow | Low | Low | Benchmark on target hardware; tune cost parameters |
| Brute force attacks | Medium | High | Rate limiting + account lockout (see security section) |
| Token enumeration in reset flow | Low | Medium | Always return generic message regardless of email existence (AC-011) |
