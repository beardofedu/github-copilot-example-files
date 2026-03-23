# Project Constitution

<!--
  FILE: .github/specs/CONSTITUTION.md
  PURPOSE: Spec-Driven Development (SDD) — Project Constitution
  SPEC-KIT: https://github.com/github/spec-kit

  WHAT IS SPEC-DRIVEN DEVELOPMENT?
    Spec-Driven Development (SDD) inverts the traditional relationship between
    code and specifications. Instead of writing code and then writing docs to
    describe it, you write precise specifications FIRST and use AI to generate
    code FROM those specifications.

    Core idea: Specifications are the source of truth. Code is their output.

  WHAT IS THE CONSTITUTION?
    The Constitution is the foundational document of your project. It captures:
      - Guiding principles and values
      - Non-negotiable quality standards
      - Architectural constraints and patterns
      - Development philosophy and process rules

    The Constitution is created once (or rarely updated) and informs all other
    spec-kit documents. Every technical decision should be checked against these
    principles.

  HOW TO CREATE THIS FILE:
    1. Automated: Use the `/speckit.constitution` command in a Copilot-enabled
       tool to generate a constitution through dialogue with an AI assistant.
       Example prompt:
         /speckit.constitution Create principles focused on code quality,
         testing standards, user experience consistency, and performance.

    2. Manual: Write this document yourself following the structure below.

  HOW IT FITS INTO THE SDD WORKFLOW:
    CONSTITUTION → SPEC → PLAN → TASKS → IMPLEMENT
         ↑             (this file)
    The constitution is the foundation. Every subsequent spec, plan, and
    implementation decision should align with these principles.

  RELATIONSHIP TO OTHER COPILOT CUSTOMIZATION FILES:
    The constitution overlaps conceptually with copilot-instructions.md but
    serves a different purpose:
      - copilot-instructions.md: Gives Copilot tactical instructions
        (naming conventions, test frameworks, code style).
      - CONSTITUTION.md: Captures strategic decisions and the "why" behind
        them (architectural patterns, quality philosophies, team values).
    Both can coexist; they complement each other.

  DEVELOPMENT NOTE:
    Customize this file for your actual project. The content below is an
    example for a web application team.
-->

## Purpose and Values

This document defines the guiding principles that govern all development
decisions in this project. These principles take precedence over personal
preferences, expediency, or short-term convenience.

When in doubt about a technical decision, consult this constitution.

---

## Core Principles

### 1. Correctness Before Speed

<!--
  CONSTITUTION NOTE: Defining your team's priority ordering matters because
  trade-offs are inevitable. Being explicit prevents later disagreements.
-->

We ship correct software, even if it takes longer. A bug in production costs
more than the time saved by cutting corners. Quality is not optional:

- All code must have corresponding tests before it is merged
- Performance optimizations are not applied at the expense of correctness
- We do not ship "temporary" workarounds — if something is broken, we fix it

### 2. Simplicity Over Cleverness

- Prefer readable code over clever code
- If a function needs a lengthy comment to explain what it does, it is too complex
- Premature optimization is a violation of this principle
- Functions do one thing; modules have a single responsibility

### 3. Explicit Over Implicit

- Configuration is explicit and documented, never magic
- Dependencies are declared, not assumed
- Side effects are minimized and clearly documented when present
- Naming must communicate intent — avoid abbreviations in public APIs

### 4. Tests Are First-Class Citizens

<!--
  CONSTITUTION NOTE: Specifying your testing philosophy at the constitutional
  level means every team member and AI agent will follow the same approach.
  This prevents inconsistent testing across the codebase.
-->

- Tests are written BEFORE or ALONGSIDE the code they test (TDD/BDD preferred)
- The test suite is the safety net that enables confident refactoring
- A feature is not "done" until it has tests
- Tests are maintained with the same discipline as production code
- Flaky tests are fixed or removed immediately — they erode trust

### 5. Security Is Non-Negotiable

- Secrets and credentials are never committed to source control
- All user input is validated and sanitized before use
- Authentication and authorization are applied consistently
- Security reviews are part of the PR process, not an afterthought
- Dependencies are kept up-to-date to receive security patches

### 6. Documentation Is Part of the Work

- Public APIs are documented with their inputs, outputs, and side effects
- Architectural decisions are recorded in ADRs (Architecture Decision Records)
- README files are kept current with setup and usage instructions
- This constitution and all spec documents are maintained as the codebase evolves

---

## Architectural Principles

<!--
  CONSTITUTION NOTE: Capturing architectural decisions here prevents the same
  debates from happening repeatedly. When a new developer or AI agent asks
  "should we use X or Y?", the answer is here.
-->

### Service Architecture

- Services are independently deployable and follow the single responsibility principle
- Communication between services uses well-defined, versioned contracts
- No direct database access across service boundaries — use APIs
- Each service owns its data store

### Data Management

- Data is normalized at the source; denormalization is explicit and intentional
- Migrations are additive where possible (no destructive schema changes without a plan)
- Sensitive data is encrypted at rest
- Data retention policies are enforced in code, not just policy documents

### API Design

- All APIs follow REST conventions unless there is a compelling reason not to
- API versioning is required for public-facing endpoints
- Errors return structured, informative responses (not raw stack traces)
- Pagination is applied to all list endpoints from day one

### Frontend Architecture

- Business logic lives in the backend; frontend handles presentation only
- Client-side state is minimized; server state is the source of truth
- Accessibility (WCAG 2.1 AA) is a requirement, not a nice-to-have
- Mobile-first responsive design

---

## Development Process

<!--
  CONSTITUTION NOTE: Codifying process in the constitution means AI agents
  (and new team members) know how work should be organized and reviewed.
-->

### Branching Strategy

- `main` is always deployable
- Feature development happens in branches named: `<type>/<description>`
  - Types: `feat/`, `fix/`, `chore/`, `docs/`, `refactor/`
  - Example: `feat/user-authentication`, `fix/login-redirect-loop`
- Branches are short-lived; merge within days, not weeks

### Code Review

- All code is reviewed by at least one other team member before merging
- Reviewers check correctness, tests, security, and alignment with this constitution
- "Ship it" is not sufficient feedback — reviews provide actionable commentary
- Authors respond to or resolve all review comments before merging

### Continuous Integration

- The CI pipeline must pass before any merge to `main`
- The pipeline includes: linting, type checking, unit tests, integration tests
- Build times are monitored; slow pipelines are fixed proactively
- Broken builds are the top priority — no one merges until CI is green

### Spec-Driven Workflow

1. New features start with a spec (SPEC.md) describing user needs
2. A technical plan (PLAN.md) translates the spec into implementation decisions
3. Tasks (TASKS.md) break the plan into executable units
4. Implementation follows the tasks and references the spec throughout
5. The spec is updated if requirements change — not silently abandoned

---

## Definition of Done

A feature is considered "done" when:

- [ ] The spec's acceptance criteria are all satisfied
- [ ] Unit tests cover the new behavior
- [ ] Integration tests cover the critical user paths
- [ ] Documentation is updated (README, API docs, ADRs as appropriate)
- [ ] Code review is approved
- [ ] CI pipeline passes
- [ ] The feature is deployed and verified in a staging environment
