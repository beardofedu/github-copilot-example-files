# Implementation Tasks: User Authentication System

<!--
  FILE: .github/specs/TASKS.md
  PURPOSE: Spec-Driven Development (SDD) — Executable Task List
  SPEC-KIT: https://github.com/github/spec-kit

  WHAT IS THIS FILE?
    The TASKS file is the final step before implementation. It breaks the
    technical plan (PLAN.md) into a concrete, ordered list of tasks that a
    developer (or AI coding agent) can execute one by one.

    Unlike PLAN.md (which explains architecture), TASKS.md is action-oriented:
    each task is a specific unit of work with a clear deliverable.

  HOW TO CREATE THIS FILE:
    Automated: After creating SPEC.md and PLAN.md, use `/speckit.tasks`:
      /speckit.tasks

    The command will:
      1. Read PLAN.md (required) as the primary input
      2. Optionally read supporting files: data-model.md, contracts/, research.md
      3. Generate this TASKS.md with:
         - Sequentially ordered tasks
         - Parallel task markers [P] for tasks that can run simultaneously
         - Acceptance criteria references for each task
         - Clear deliverables for each task

    Manual: Write it yourself following the structure below.

  TASK STRUCTURE:
    Each task should specify:
      - A clear action verb (Create, Implement, Write, Configure, etc.)
      - The specific deliverable (file name, function, endpoint, etc.)
      - Which acceptance criteria it satisfies (from SPEC.md)
      - Dependencies on other tasks

  PARALLEL TASKS:
    Tasks marked [P] can be worked on simultaneously without blocking each other.
    The spec-kit convention uses parallel groups to indicate which tasks can
    safely run in parallel. Example:
      Parallel Group A: Tasks 3 and 4 can run concurrently
      Sequential: Task 5 requires Tasks 3 and 4 to be complete

  SDD WORKFLOW POSITION:
    CONSTITUTION → SPEC → PLAN → [TASKS] → IMPLEMENT
                                 ↑ (this file)

  NOTE ON AI AGENTS:
    This file is designed to be consumed by Copilot coding agent. When you
    assign a task to the agent, link it to this TASKS.md and the specific task
    number. The agent will use SPEC.md, PLAN.md, and this file to understand
    the full context of what needs to be built.
-->

**Specification Reference**: SPEC.md (Feature 001: User Authentication System)
**Plan Reference**: PLAN.md
**Status**: Ready for Implementation
**Last Updated**: 2026-03-23

---

## Prerequisites

Before beginning implementation, ensure:

- [ ] PostgreSQL database is set up and accessible
- [ ] Environment variables are configured (see `.env.example`)
- [ ] Node.js version matches the project's `.nvmrc`
- [ ] `npm install` has been run to install dependencies

---

## Phase 1: Foundation

*Tasks in this phase must be completed sequentially. Nothing else can start
until Phase 1 is done.*

### Task 1 — Create database migrations

**Deliverable**: Two migration files in `db/migrations/`:
- `001_create_users_table.sql` — creates the `users` table (see PLAN.md)
- `002_create_password_reset_tokens_table.sql` — creates the `password_reset_tokens` table

**Satisfies**: Enables all acceptance criteria (foundational)
**Dependencies**: None
**Notes**: Include all indexes defined in PLAN.md. Use `gen_random_uuid()` for UUIDs.

---

### Task 2 — Create user model / data access layer

**Deliverable**: `src/models/User.ts` with the following methods:
- `User.create(email, passwordHash)` — insert a new user, return user object
- `User.findByEmail(email)` — return user or null
- `User.findById(id)` — return user or null
- `User.findByGithubId(githubId)` — return user or null
- `User.update(id, fields)` — update specified fields, return updated user
- `User.incrementFailedLoginAttempts(id)` — increment counter, set lockout if threshold reached

**Satisfies**: Foundational for all auth flows
**Dependencies**: Task 1 (migrations must exist)
**Notes**: Use parameterized queries only — no string interpolation in SQL.

---

## Phase 2: Core Authentication

*Tasks 3, 4, and 5 in this phase can be worked on in parallel [P] after Phase 1 is complete.*

### Task 3 — Implement password utilities [P]

**Deliverable**: `src/utils/password.ts` with:
- `hashPassword(plaintext)` — hash with Argon2id (cost factors from PLAN.md)
- `verifyPassword(plaintext, hash)` — compare and return boolean
- `validatePasswordStrength(password)` — returns `{valid: boolean, errors: string[]}`

**Satisfies**: AC-003 (weak password validation), AC-006 (secure comparison)
**Dependencies**: Task 1 (none — this is pure utility code)
**Notes**: Use the `argon2` npm package. See PLAN.md for cost factors.

### Task 4 — Implement JWT session utilities [P]

**Deliverable**: `src/utils/jwt.ts` with:
- `generateToken(userId, rememberMe)` — sign a JWT with correct expiry
- `verifyToken(token)` — verify and decode, throw on invalid/expired
- `SESSION_COOKIE_OPTIONS` — exported cookie configuration object

**Satisfies**: AC-007, AC-008 (remember me behavior)
**Dependencies**: None (pure utility code)
**Notes**: Use the `jsonwebtoken` package. Cookie must be HttpOnly, Secure, SameSite=Lax.

### Task 5 — Create password reset token utilities [P]

**Deliverable**: `src/models/PasswordResetToken.ts` with:
- `createToken(userId)` — generate random token, hash it, store, return raw token
- `findValidToken(tokenHash)` — return token record if valid and unexpired
- `markTokenUsed(tokenId)` — set `used_at` to NOW()

**Satisfies**: AC-010, AC-012, AC-013 (password reset flow)
**Dependencies**: Task 1 (migration must exist), Task 2 (user model)
**Notes**: Use `crypto.randomBytes(32).toString('hex')` for the token. Store only the hash.

---

## Phase 3: Registration and Email Verification

### Task 6 — Implement registration service

**Deliverable**: `src/services/auth/register.ts` with:
- `registerUser(email, password)` function that:
  1. Validates email format and password strength (use Task 3 utility)
  2. Checks for duplicate email (throws `EMAIL_IN_USE` error if found)
  3. Hashes the password (use Task 3 utility)
  4. Creates the user (use Task 2 model)
  5. Sends a verification email (stub the email service — implement in Task 7)
  6. Returns the created user (without password hash)

**Satisfies**: AC-001, AC-002, AC-003
**Dependencies**: Tasks 2, 3, 7

### Task 7 — Implement email service

**Deliverable**: `src/services/email.ts` with:
- `sendVerificationEmail(email, token)` — send verification link
- `sendPasswordResetEmail(email, token)` — send reset link
- Use nodemailer configured from environment variables (SMTP settings)

**Satisfies**: AC-001, AC-004, AC-010
**Dependencies**: None (standalone service)
**Notes**: Use environment variables for SMTP config. Never hardcode credentials.

### Task 8 — Create registration endpoint

**Deliverable**: `POST /auth/register` route in `src/routes/auth.ts`:
- Validate request body (email, password)
- Call `registerUser()` from Task 6
- Return 201 with userId, or appropriate error code (see PLAN.md API contracts)

**Satisfies**: AC-001, AC-002, AC-003
**Dependencies**: Task 6

---

## Phase 4: Login and Session Management

### Task 9 — Implement login service

**Deliverable**: `src/services/auth/login.ts` with:
- `loginUser(email, password, rememberMe)` function that:
  1. Finds user by email
  2. Checks if account is locked
  3. Verifies password (use Task 3 utility)
  4. On failure: increments failed attempt counter, throws `INVALID_CREDENTIALS`
  5. On success: resets failed attempt counter, returns user + session token

**Satisfies**: AC-005, AC-006, AC-007, AC-008, AC-009
**Dependencies**: Tasks 2, 3, 4

### Task 10 — Create login and logout endpoints

**Deliverable**: Routes in `src/routes/auth.ts`:
- `POST /auth/login` — call `loginUser()`, set session cookie, return user data
- `POST /auth/logout` — clear session cookie, return 200

**Satisfies**: AC-005, AC-006, AC-007, AC-008
**Dependencies**: Task 9

### Task 11 — Implement rate limiting middleware

**Deliverable**: `src/middleware/rateLimiter.ts`:
- Create a rate limiter for auth endpoints (5 attempts per IP per 15 minutes)
- Apply to `/auth/login` and `/auth/forgot-password` routes

**Satisfies**: AC-009 (rate limiting as first layer of brute force protection)
**Dependencies**: None
**Notes**: Use `express-rate-limit`. Return `Retry-After` header on limit.

---

## Phase 5: Password Reset

### Task 12 — Implement password reset service

**Deliverable**: `src/services/auth/passwordReset.ts` with:
- `requestPasswordReset(email)` — create token, send email (always return success)
- `resetPassword(token, newPassword)` — validate token, hash new password, update user

**Satisfies**: AC-010, AC-011, AC-012, AC-013
**Dependencies**: Tasks 2, 3, 5, 7

### Task 13 — Create password reset endpoints

**Deliverable**: Routes in `src/routes/auth.ts`:
- `POST /auth/forgot-password` — call `requestPasswordReset()`, return generic message
- `POST /auth/reset-password` — call `resetPassword()`, set new session cookie

**Satisfies**: AC-010, AC-011, AC-012, AC-013
**Dependencies**: Task 12

---

## Phase 6: GitHub OAuth [P]

*This phase can run in parallel with Phases 3-5 after Phase 2 is complete.*

### Task 14 — Configure GitHub OAuth (Passport.js)

**Deliverable**: `src/config/passport.ts`:
- Configure `passport-github2` strategy
- On GitHub callback: find or create user by GitHub ID
- Handle account linking (email match with existing account)

**Satisfies**: US-002, US-005
**Dependencies**: Task 2 (user model)
**Notes**: Requires `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` env vars.

### Task 15 — Create GitHub OAuth endpoints

**Deliverable**: Routes in `src/routes/auth.ts`:
- `GET /auth/github` — redirect to GitHub authorization page
- `GET /auth/github/callback` — handle callback, set session cookie, redirect

**Satisfies**: US-002, US-005
**Dependencies**: Task 14

---

## Phase 7: Authentication Middleware

### Task 16 — Create JWT authentication middleware

**Deliverable**: `src/middleware/authenticate.ts`:
- Extract JWT from cookie
- Verify token (use Task 4 utility)
- Attach user to `req.user`
- Return 401 if no valid token

**Satisfies**: Enables all protected routes
**Dependencies**: Task 4

---

## Phase 8: Tests

*Tests can be written in parallel with implementation. Each test task mirrors its implementation task.*

### Task 17 — Write unit tests for utilities [P]

**Deliverable**: Test files for Tasks 3, 4, 5 (password utils, JWT utils, token model).
Each file in `src/__tests__/unit/`.

**Satisfies**: Test coverage for AC-003, AC-007, AC-008, AC-013

### Task 18 — Write integration tests for auth endpoints [P]

**Deliverable**: Integration test suite in `src/__tests__/integration/auth.test.ts`
covering all acceptance criteria from SPEC.md.

**Satisfies**: AC-001 through AC-013 (all acceptance criteria)
**Dependencies**: Tasks 8, 10, 11, 13, 15, 16 (endpoints must exist to test)
**Notes**: Use `supertest` + a separate test database. Seed data as needed.

---

## Summary Table

| Task | Description | Phase | Parallel? | Deps |
|---|---|---|---|---|
| 1 | Database migrations | 1 | No | — |
| 2 | User model | 1 | No | 1 |
| 3 | Password utilities | 2 | Yes [P] | — |
| 4 | JWT utilities | 2 | Yes [P] | — |
| 5 | Reset token model | 2 | Yes [P] | 1, 2 |
| 6 | Registration service | 3 | No | 2, 3, 7 |
| 7 | Email service | 3 | No | — |
| 8 | Registration endpoint | 3 | No | 6 |
| 9 | Login service | 4 | No | 2, 3, 4 |
| 10 | Login/logout endpoints | 4 | No | 9 |
| 11 | Rate limiting middleware | 4 | No | — |
| 12 | Password reset service | 5 | No | 2, 3, 5, 7 |
| 13 | Password reset endpoints | 5 | No | 12 |
| 14 | GitHub OAuth config | 6 | Yes [P] | 2 |
| 15 | GitHub OAuth endpoints | 6 | No | 14 |
| 16 | Auth middleware | 7 | No | 4 |
| 17 | Unit tests | 8 | Yes [P] | 3, 4, 5 |
| 18 | Integration tests | 8 | Yes [P] | 8, 10, 11, 13, 15, 16 |
