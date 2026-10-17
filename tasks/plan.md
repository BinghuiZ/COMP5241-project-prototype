# Implementation Plan: Identity and Roles

## Overview

Implement the first dependency module of the AI-powered learning platform:
Next.js application foundations, Neon PostgreSQL-backed users and roles, email
and password registration/login, invitation-based learner registration, email
verification, password reset, and one-device JWT access/refresh token lifecycle,
logout/revocation, and server-enforced authorization. This plan implements
`SPEC-identity-and-roles.md`; it does not implement course authoring, content
generation, enrollment, learning delivery, or analytics.

## Architecture Decisions

- Use Next.js with TypeScript and a typed server/API layer.
- Use Tailwind CSS for styling, with reusable components for repeated visual
  patterns.
- Use Drizzle ORM and migrations against Neon PostgreSQL.
- Transport JWT credentials in secure HttpOnly cookies.
- Allow instructor self-registration and require invitations for learner
  registration.
- Send verification, reset, and invitation messages through a Resend adapter.
- Keep authentication, authorization, persistence, and UI concerns behind
  separate module boundaries.
- Store only password hashes and hashed refresh tokens; never persist raw
  credentials.
- Store email-verification and password-reset tokens only as one-way hashes,
  with single-use and expiry enforcement.
- Keep invitation tokens hashed, single-use, expiring, and scoped to the invited
  email and learner role.
- Enforce the one-active-device policy by revoking the existing token record
  whenever a new login or refresh rotation succeeds.
- Use environment-provided JWT signing configuration and fail explicitly when
  required configuration is missing.
- Keep authorization policies independent from route handlers so they can be
  unit tested and reused by later course modules.

## Task List

### Phase 1: Foundation

- [x] Task 1: Bootstrap the Next.js TypeScript application, Tailwind, and quality scripts
- [ ] Task 2: Add Neon PostgreSQL persistence, migrations, and identity data model

### Checkpoint: Foundation

- [x] Application starts in development mode
- [x] Production build, lint, typecheck, and focused test commands are wired
- [ ] Test database migration succeeds

### Phase 2: Authentication

- [ ] Task 3: Implement registration, email verification, and password-based login
- [ ] Task 4: Implement password reset and invitation-based learner registration
- [ ] Task 5: Implement JWT issuance, refresh rotation, expiry, and logout

### Checkpoint: Authentication

- [ ] Registration and login work through the API
- [ ] Refresh rotation and logout reject revoked predecessors
- [ ] No raw passwords or refresh tokens appear in logs or persistence

### Phase 3: Authorization and UI

- [ ] Task 6: Implement role and resource authorization policies
- [ ] Task 7: Add identity UI and protected navigation behavior

### Checkpoint: Complete

- [ ] Instructor and learner access rules are enforced server-side
- [ ] Browser login, logout, and expiry behavior work
- [ ] Identity tests meet the specification's coverage thresholds
- [ ] Human review confirms the module is ready for the next dependent spec

## Dependency Graph

```text
Next.js/tooling
      |
Neon PostgreSQL schema and repositories
      |
Registration, verification, and login
      |
Password reset
      |
JWT refresh/revocation lifecycle
      |
Authorization policies
      |
Identity UI and protected navigation
```

## Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| JWT signing configuration is missing or misconfigured | High | Validate configuration at startup and fail with an actionable server error |
| Refresh token replay after rotation | High | Persist only a hash, rotate on refresh, and revoke the token record atomically |
| Client bypasses role restrictions | High | Keep all authorization checks in server/API code and test denied paths |
| Password or token data leaks through logs | High | Redact credentials at boundaries and add explicit log-safety tests |
| Email verification or reset delivery is misconfigured | Medium | Keep delivery behind a provider adapter and test token issuance independently from delivery |
| Invitation links are reused or grant an unauthorized role | High | Hash and atomically consume invitations, bind them to the normalized email and learner role, and test replay/role escalation |
| Framework/database tooling is selected inconsistently | Medium | Record the final choices in the spec and use one Neon/PostgreSQL migration/repository pattern |
| UI expiry handling diverges from API behavior | Medium | Add browser tests for expiry, logout, and protected navigation after API tests pass |

## Open Questions

- What deployment target and environment-secret provider will be used?
- Can a user hold both instructor and learner roles?
- Should account lockout and rate limiting be included in the first release?
- What Resend sender identity and email templates should be used?
- What deployment target and environment-secret provider will be used?
