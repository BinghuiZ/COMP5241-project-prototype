# Spec: Identity and Roles

## Objective

Provide secure identity and role-based access for the AI-powered learning
platform. Users must be able to register and log in, and the platform must
distinguish instructors from learners so that protected course content and
actions are shown only to authorized users.

### User stories

- As a user, I can create an account and log in with my email and password.
- As a user, I receive a JWT access/refresh token pair after successful login.
- As a user, I can log out and invalidate my current device token.
- As an instructor, I can access instructor-only course authoring and roster
  features.
- As a learner, I can access learner-only learning content and activities.
- As an authenticated user, I can access content explicitly assigned to me or
  available to my role.

## Assumptions

1. This is a browser-based web application targeting current modern browsers.
2. Email/password authentication is required.
3. Authentication uses a JWT access token and refresh token flow.
4. A user may have one active device/token at a time. Issuing a new token pair
   invalidates the previous pair.
5. The token lifetime is one day. Once the session expires, the user is logged
   out and must authenticate again.
6. Neon-hosted PostgreSQL is the persistence layer.
7. The initial product has two roles: `instructor` and `learner`.
8. The repository is currently empty of application code, and Next.js is the
   selected web framework; deployment details remain open.
9. The first release includes email verification and password reset.
10. Learners may register through an invitation email sent to the address
    selected by an instructor or course workflow.
11. Instructors may self-register; learners must register through an invitation.
12. JWTs are transported using secure HttpOnly cookies.
13. Drizzle ORM is used for Neon/PostgreSQL migrations and persistence.
14. Resend is the initial transactional email provider behind an application
    email adapter.

## Tech Stack

### Selected/provisional

- TypeScript for application code.
- Next.js for the browser application and typed server/API layer.
- Tailwind CSS for application styling.
- Neon PostgreSQL for users, roles, token metadata, and audit events.
- Drizzle ORM and migrations for database access.
- Resend for transactional email delivery through an adapter.
- Password hashing using a maintained memory-hard password-hashing library.
- JWT signing using a maintained library and a server-side key supplied through
  environment configuration.

### Required design constraints

- Passwords must never be stored or logged in plaintext.
- Refresh tokens must be revocable and stored only as a one-way hash, not as
  reusable plaintext credentials.
- JWT claims must contain only the minimum identity and authorization data.
- All authentication and authorization failures must return consistent,
  non-sensitive errors.

## Commands

The repository has no package manifest yet. After the project framework is
selected, these commands must be implemented as the canonical project scripts:

```sh
npm run dev
npm run build
npm test -- --coverage
npm run lint
npm run typecheck
```

Identity-specific verification must include:

```sh
npm test -- identity
```

## Project Structure

The final locations may follow the selected framework, but identity concerns
must remain separated from UI components:

```text
src/
  auth/                 authentication services, token issuance, logout
  users/                user model and user repository
  roles/                role definitions and authorization policy
  api/                  request handlers and input validation
  lib/                  database, password hashing, and JWT adapters
tests/
  auth/                 authentication and token lifecycle tests
  authorization/        role and resource-access tests
```

## Code Style

- Use strict TypeScript types and explicit result/error handling.
- Use Tailwind utility classes for UI styling and keep reusable visual patterns
  in shared components rather than duplicating long class lists.
- Use `camelCase` for variables and functions, `PascalCase` for types and
  classes, and kebab-case for URL path segments.
- Keep token creation and verification behind a service boundary so handlers do
  not construct JWT payloads directly.
- Validate external input at the API boundary.

Example:

```ts
type UserRole = "instructor" | "learner";

interface AuthenticatedUser {
  readonly id: string;
  readonly role: UserRole;
}

function canManageCourse(user: AuthenticatedUser): boolean {
  return user.role === "instructor";
}
```

## Functional Requirements

1. Registration accepts a valid email, password, and initial role according to
   the platform's account policy.
2. Login verifies the password and issues an access token and refresh token.
3. Issuing a new token pair revokes the user's previous active device token.
4. Refreshing a token rotates the refresh token and revokes its predecessor.
5. Logout revokes the active refresh token and prevents further refresh.
6. Expired or revoked tokens cannot access protected resources.
7. Authorization checks enforce instructor and learner permissions on every
   protected operation.
8. User-specific content checks enforce both authentication and resource
   ownership/membership, not role alone.
9. Registration can initiate email verification, and verified status is
   enforced where required by the account policy.
10. A user can request a password reset through a single-use, expiring,
    revocable reset flow.
11. An invited learner can accept a single-use, expiring invitation token and
    complete registration without selecting an unauthorized role.
12. Authentication events record enough metadata for troubleshooting and
   security review without recording passwords or raw tokens.

## Testing Strategy

- Unit tests cover password verification, JWT claim construction and
  verification, token expiry, refresh rotation, revocation, and role policies.
- Integration tests cover registration, login, refresh, logout, and protected
  endpoint behavior against a test PostgreSQL database.
- Negative tests cover malformed credentials, invalid signatures, expired
  tokens, revoked tokens, duplicate emails, and cross-role access.
- Security tests verify that logs and persisted records contain no passwords or
  raw refresh tokens.
- Browser-level tests are required once the login UI exists; they must verify
  successful login, expiry behavior, logout, and role-specific navigation.
- The identity module should maintain at least 90% line coverage and 100%
  coverage for authorization policy branches.

## Boundaries

- **Always:** Validate and normalize emails; hash passwords; use constant-time
  comparisons where applicable; rotate and revoke refresh tokens; enforce
  authorization server-side; use secure environment-managed signing keys;
  expire and invalidate email-verification and password-reset tokens; run
  invalidate invitation tokens; run identity tests, lint, and type checking
  before commits.
- **Ask first:** Changing the token model or one-device policy; changing token
  lifetime; adding roles; changing the database schema; adding authentication
  providers or dependencies; changing CI or deployment configuration.
- **Never:** Store plaintext passwords or raw refresh tokens; put secrets in
  source control; trust client-supplied roles; expose token signing keys;
  weaken or remove failing security tests without approval.

## Success Criteria

1. A valid user can register, log in, access an authenticated endpoint, and
   log out through the documented API.
2. Login issues a one-day access/refresh token pair with minimum necessary
   claims.
3. A second login revokes the first active device token, and the first token
   cannot refresh.
4. Refresh rotates the token and makes the predecessor unusable.
5. Email verification and password-reset tokens are single-use and expire
   according to the documented policy.
6. Invitation tokens are single-use, scoped to the invited email and learner
   role, and expire according to the documented policy.
7. Expired, malformed, and revoked tokens are rejected without revealing
   sensitive details.
8. Instructor-only and learner-only operations are denied for the wrong role.
9. A user cannot access content assigned to another user or course membership.
10. Automated identity tests pass, with at least 90% line coverage and complete
   authorization-policy branch coverage.
11. No password, signing key, or raw refresh/invitation/reset token appears in logs or persisted
   token records.

## Open Questions

1. What deployment target and environment-secret provider will be used?
2. Can a user hold both instructor and learner roles?
3. Should account lockout and rate limiting be part of the first release?
4. What Resend sender identity and email templates should be used?
