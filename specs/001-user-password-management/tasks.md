# Tasks: User Password Management

**Input**: Design documents from `/specs/001-user-password-management/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Initial setup for the Spring Boot password-management backend service

- [ ] T001 Create the feature folder structure and baseline Java package layout for `src/main/java/sdd/project` and `src/test/java/sdd/project`
- [ ] T002 [P] Update `pom.xml` to include the required Spring Boot configuration and validation support for the password-management feature
- [ ] T003 [P] Create the base package structure for `auth`, `user`, and `security` under `src/main/java/sdd/project`
- [ ] T004 [P] Configure application logging and backend error response handling in `src/main/java/sdd/project` for credential validation failures

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared domain and validation components required before any story implementation

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

- [ ] T005 Create the `UserAccount` domain model in `src/main/java/sdd/project/auth/UserAccount.java` with fields for `id`, `username`, `passwordHash`, `role`, `createdAt`, and `updatedAt`
- [ ] T006 [P] Create the `PasswordChangeAuditLog` entity in `src/main/java/sdd/project/auth/PasswordChangeAuditLog.java` with `username`, `timestamp`, `outcome`, and `reason`
- [ ] T007 [P] Implement password policy validation in `src/main/java/sdd/project/security/PasswordPolicyValidator.java` to enforce: 8-15 characters, mixed case, at least one number, at least one special character, no blank values, no offensive content, no username or sequential-content patterns, no reuse of current password, and no immediate reuse of the previous password in history
- [ ] T008 Create the `UserAccountRepository` contract in `src/main/java/sdd/project/user/UserAccountRepository.java` for account lookup, username uniqueness checks, password persistence, password history, and password epoch tracking
- [ ] T009 Implement the shared password-change audit service in `src/main/java/sdd/project/security/PasswordChangeAuditService.java` for recording successful and failed password change attempts and rate-limit events

---

## Phase 3: User Story 1 - Create a login account (Priority: P1) 🎯 MVP

**Goal**: Allow a user to create a username/password account with a single role and reject invalid or duplicate sign-up attempts.

**Independent Test**: Create a valid account, verify sign-in succeeds, and confirm duplicate or blank-value inputs are rejected with a clear error.

### Implementation for User Story 1

- [ ] T010 [P] [US1] Implement account creation logic in `src/main/java/sdd/project/user/UserAccountService.java` to validate input, enforce uniqueness, and reject offensive values
- [ ] T011 [P] [US1] Implement the registration endpoint in `src/main/java/sdd/project/auth/AuthController.java` for `POST /users/register`
- [ ] T012 [US1] Implement authentication logic in `src/main/java/sdd/project/auth/AuthService.java` to validate sign-in requests and return correct success or error messages
- [ ] T013 [US1] Add rejection paths for blank username/password and duplicate usernames in `src/main/java/sdd/project/auth/AuthService.java` and `src/main/java/sdd/project/user/UserAccountService.java`
- [ ] T014 [US1] Add user-facing validation messages and error handling in the controller/service flow for invalid account creation and sign-in cases

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently.

---

## Phase 4: User Story 2 - Change a password with current credentials (Priority: P1)

**Goal**: Allow an authenticated user to change their password only when they provide the correct current password and a compliant new password.

**Independent Test**: Submit a valid change request and confirm the password updates; then verify wrong current password, missing current password, reused password, and blank values are rejected without changing the stored password.

### Implementation for User Story 2

- [ ] T015 [P] [US2] Implement password-change validation in `src/main/java/sdd/project/security/PasswordPolicyValidator.java` for new-password rules, username/sequential-content rejection, password-history checks, and reused-password rejection
- [ ] T016 [US2] Implement the password change service flow in `src/main/java/sdd/project/user/UserAccountService.java` to verify the username, current password, new password, password history, and per-user change limits before update
- [ ] T017 [P] [US2] Implement the password change endpoint in `src/main/java/sdd/project/auth/AuthController.java` for `PUT /users/password`
- [ ] T018 [US2] Add the wrong-current-password rejection path in `src/main/java/sdd/project/auth/AuthService.java` and ensure no password update occurs
- [ ] T019 [US2] Add the same-password, missing-current-password, and rate-limit rejection checks in the service and controller flow with clear user messages
- [ ] T020 [US2] Ensure `PasswordChangeAuditService` records each password-change attempt with username, timestamp, and success/failure outcome in `src/main/java/sdd/project/security/PasswordChangeAuditService.java`
- [ ] T021 [US2] Add password_epoch-based session invalidation so successful password changes revoke all other active sessions, increment the password epoch used for JWT validation, and reject stale tokens after the update
- [ ] T021A [US2] Publish a `password_changed` security event and log notification delivery failure without rolling back the successful password update

**Checkpoint**: At this point, User Story 2 should be fully functional and independently testable.

---

## Phase 5: User Story 3 - Authenticate and authorize secure password operations (Priority: P2)

**Goal**: Ensure the backend authenticates valid users and enforces secure access to account-sensitive password operations while preserving the single role setting and secure access rules.

**Independent Test**: Submit valid credentials to the authentication service, then verify that a password change request succeeds only after successful sign-in and valid current-password verification.

### Implementation for User Story 3

- [ ] T022 [P] [US3] Confirm the single-role account contract in `src/main/java/sdd/project/auth/UserAccount.java` and ensure the role remains intact through password updates
- [ ] T023 [US3] Update `src/main/java/sdd/project/auth/AuthService.java` so successful sign-in returns the expected username and role after login and password update flows
- [ ] T024 [US3] Add validation for invalid usernames and passwords in the sign-in flow and ensure all failures emit a user-readable error message
- [ ] T025 [P] [US3] Enforce authentication gating for password-change operations with a preferred session/token flow and a username/current-password fallback path, ensuring the session/token takes precedence when both are supplied
- [ ] T026 [US3] Validate the authenticated password-change flow through backend service tests to confirm the existing password, password policy, and session invalidation rules are all enforced together
- [ ] T027 [US3] Run the API-focused validation scenarios documented in `specs/001-user-password-management/quickstart.md` and confirm the sign-in, stale-token rejection, and password-change flows are consistent across success and failure cases

**Checkpoint**: At this point, all user stories should be independently functional.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final verification and clean support for the password-management feature

- [ ] T030 [P] Review and update documentation in `specs/001-user-password-management/spec.md` and `specs/001-user-password-management/quickstart.md` to ensure they reflect the final rules and audit logging behavior
- [ ] T031 [P] Review all validation and logging paths for consistency across account creation, sign-in, and password change in `src/main/java/sdd/project/auth` and `src/main/java/sdd/project/security`
- [ ] T032 Run the focused validation checklist against the password management flow and confirm all acceptance criteria in `spec.md` are satisfied

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1**: No dependencies - can start immediately.
- **Phase 2**: Depends on Phase 1 completion and blocks all user story work.
- **Phase 3 / Phase 4 / Phase 5**: Depend on Phase 2 completion.
- **Phase 6**: Depends on all user-story work being complete.

### User Story Dependencies

- **User Story 1 (US1)**: Can start after Phase 2; no dependency on other stories.
- **User Story 2 (US2)**: Can start after Phase 2; no dependency on other stories, but it depends on the same validation and account model foundation.
- **User Story 3 (US3)**: Can start after Phase 2; validates the account lifecycle across login and password update flows.

### Parallel Opportunities

- T002, T003, and T004 can run in parallel.
- T006, T007, and T009 can run in parallel after Phase 1.
- T010 and T011 can run in parallel for User Story 1.
- T015 and T017 can run in parallel for User Story 2.
- T022 and T023 can run in parallel for User Story 3.
- T030 and T031 can run in parallel in the final polish phase.

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Implement User Story 1 to deliver account creation and basic sign-in.
3. Validate the sign-up and login flow.
4. Then implement User Story 2 for password change verification and audit logging.
5. Finish with User Story 3 to verify the end-to-end login experience after password updates.

### Incremental Delivery

- Deliver the account creation and sign-in flow first as the minimum viable functionality.
- Add password updates next, with validation and audit logging.
- Complete the login confidence checks and final cross-cutting verification last.
