# Identity and Roles Tasks

## Task 1: Bootstrap the Next.js TypeScript application, Tailwind, and quality scripts

**Description:** Create the Next.js and Tailwind CSS application foundation and
canonical development, build, test, lint, and typecheck scripts without
implementing business behavior.

**Acceptance criteria:**

- [x] Next.js and TypeScript application starts in development mode.
- [x] Tailwind CSS is configured and a sample utility style renders correctly.
- [x] The canonical commands from the identity specification are available.
- [x] Strict type checking and a focused identity test selector are configured.

**Verification:**

- [x] Run `npm run build`, `npm run lint`, `npm run typecheck`, and
  `npm test -- identity`.

**Dependencies:** None

**Files likely touched:**

- `package.json`
- `tsconfig.json`
- `next.config.*`
- `tailwind.config.*`
- `postcss.config.*`
- `src/app/*`
- `tests/setup.*`

**Estimated scope:** Medium

## Task 2: Add Neon PostgreSQL persistence, migrations, and identity data model

**Description:** Select and configure one Neon/PostgreSQL ORM/migration
approach, then create users, roles, active token metadata, and security-event
persistence with repositories and test-database setup.

**Acceptance criteria:**

- [ ] The schema supports unique normalized emails, the two roles, one active
  token record per user, hashed refresh tokens, expiry, revocation, and audit
  metadata.
- [ ] Migration and rollback/test-database commands are documented and work.
- [ ] Repositories do not expose raw password or refresh-token values.

**Verification:**

- [ ] Apply migrations to a disposable Neon branch or PostgreSQL test database.
- [ ] Run repository tests for uniqueness, revocation, expiry, and redaction.

**Dependencies:** Task 1

**Files likely touched:**

- `src/lib/db/*`
- `src/users/*`
- `migrations/*`
- `tests/users/*`
- `.env.example`

**Estimated scope:** Medium

## Task 3: Implement registration, email verification, and password-based login

**Description:** Add validated registration and login API flows using a
maintained memory-hard password hash, normalized email lookup, expiring
single-use email-verification tokens, safe errors, and security-event
recording.

**Acceptance criteria:**

- [ ] Valid registration creates one user with a password hash and allowed
  role.
- [ ] Verification tokens are single-use, expiring, revocable, and stored only
  as hashes.
- [ ] Login issues credentials only after successful password verification.
- [ ] Duplicate emails, malformed input, and invalid credentials return
  non-sensitive errors.

**Verification:**

- [ ] Run unit and integration tests for valid and invalid registration/login.
- [ ] Verify persisted data and logs contain no plaintext passwords.

**Dependencies:** Task 2

**Files likely touched:**

- `src/auth/passwords.*`
- `src/auth/registration.*`
- `src/auth/email-verification.*`
- `src/auth/login.*`
- `src/api/auth/*`
- `tests/auth/registration-login.*`

**Estimated scope:** Medium

## Task 4: Implement password reset and invitation-based learner registration

**Description:** Add a password-reset request and completion flow plus
invitation-based learner registration using single-use, expiring, revocable,
hashed tokens and the configured email-provider adapter.

**Acceptance criteria:**

- [ ] A reset request does not reveal whether an email exists.
- [ ] A valid reset token permits exactly one password change before expiry.
- [ ] Password reset revokes the active device token and invalidates outstanding
  reset tokens.
- [ ] An invitation is bound to one normalized email and the learner role,
  expires, and can be consumed only once.
- [ ] A learner can complete registration from a valid invitation, while an
  invalid, expired, reused, or role-modified invitation is rejected.

**Verification:**

- [ ] Run tests for unknown emails, expiry, replay, revocation, successful
  reset, invitation acceptance, invitation expiry, and role escalation.
- [ ] Verify reset tokens and passwords are absent from logs and persistence.

**Dependencies:** Task 3

**Files likely touched:**

- `src/auth/password-reset.*`
- `src/auth/invitations.*`
- `src/auth/email-provider.*`
- `src/api/auth/password-reset.*`
- `tests/auth/password-reset.*`

**Estimated scope:** Medium

## Task 5: Implement JWT issuance, refresh rotation, expiry, and logout

**Description:** Add access/refresh token creation and verification, one-active-
device replacement, atomic refresh rotation, expiry handling, and logout
revocation using the approved one-day lifetime.

**Acceptance criteria:**

- [ ] Login and refresh produce minimum-claim JWTs and a one-day expiry.
- [ ] A new login invalidates the previous active token record.
- [ ] Refresh rotates and revokes the predecessor; logout prevents refresh.
- [ ] Missing signing configuration fails explicitly rather than using a
  fallback secret.

**Verification:**

- [ ] Run token lifecycle tests for issuance, expiry, replacement, replay,
  rotation, logout, and malformed signatures.
- [ ] Verify raw refresh tokens are never persisted or logged.

**Dependencies:** Task 4

**Files likely touched:**

- `src/auth/tokens.*`
- `src/auth/session-service.*`
- `src/api/auth/refresh.*`
- `src/api/auth/logout.*`
- `tests/auth/tokens.*`

**Estimated scope:** Medium

## Task 6: Implement role and resource authorization policies

**Description:** Add reusable server-side authentication guards and role/resource
policies for instructor and learner access, including user-specific content
membership checks needed by later modules.

**Acceptance criteria:**

- [ ] Protected handlers reject missing, expired, malformed, and revoked tokens.
- [ ] Instructor-only and learner-only policies deny the wrong role.
- [ ] Resource checks deny access when the authenticated user lacks membership
  or ownership.

**Verification:**

- [ ] Run authorization unit tests with complete allow/deny branch coverage.
- [ ] Run integration tests against protected example endpoints.

**Dependencies:** Task 5

**Files likely touched:**

- `src/auth/guards.*`
- `src/roles/policies.*`
- `src/api/authenticated-example.*`
- `tests/authorization/*`

**Estimated scope:** Medium

## Task 7: Add Tailwind-styled identity UI and protected navigation behavior

**Description:** Build the Next.js login/logout experience with Tailwind CSS
and protected navigation behavior, including clear handling when the one-day
token expires.

**Acceptance criteria:**

- [ ] A user can submit login credentials and reach authenticated navigation.
- [ ] Logout revokes the active token and returns the user to public
  navigation.
- [ ] Expired authentication is cleared and the user is directed to log in
  again without exposing token details.

**Verification:**

- [ ] Run browser tests for login, logout, protected navigation, and expiry.
- [ ] Run the production build and verify no protected page is accessible
  without server-side authorization.

**Dependencies:** Task 6

**Files likely touched:**

- `src/app/login/*`
- `src/components/auth/*`
- `src/lib/auth-client.*`
- `tests/e2e/identity.*`

**Estimated scope:** Medium

## Checkpoint: Identity module complete

- [ ] `npm run build` succeeds.
- [ ] `npm run lint` succeeds.
- [ ] `npm run typecheck` succeeds.
- [ ] `npm test -- --coverage` meets the identity specification thresholds.
- [ ] Browser identity flow passes against a configured Neon/PostgreSQL test
  environment.
- [ ] Human reviews the implementation before `course-authoring` begins.
