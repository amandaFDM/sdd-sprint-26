# Feature Specification: User Password Management

**Feature Branch**: `001-user-password-management`

**Created**: 2026-10-08

**Status**: Draft

**Input**: User description: "The feature must allow user to create a log in with username and password, and be able to change the password when username and existing password is provided. If password is wrong, there must be an error message. If password is correct, then password is updated. There is only 1 role, not 3."

## Clarifications

### Session 2026-10-08

- Q: What password policy should the system enforce when a user creates an account or changes a password? → A: At least 8 characters with uppercase, lowercase, and a number; cannot match the current password.
- Q: How should the system treat offensive words in usernames and passwords? → A: Block a short, fixed list of clearly offensive words and reject any value containing them.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create a login account (Priority: P1)

A person creating an account needs a username and password so they can access the system and later change that password when required.

**Why this priority**: Without a valid account, the user cannot sign in or use the password change flow. This is the foundation for every other action in the feature.

**Independent Test**: A user can complete account creation with a username and password, then sign in successfully using those credentials.

**Acceptance Scenarios**:

1. **Given** the user is creating a new account, **When** they provide a unique username and a valid password, **Then** the account is created successfully and can be used to sign in.
2. **Given** the user tries to create an account with a username that already exists, **When** they submit the form, **Then** the system prevents the duplicate account and provides a clear error message.

---

### User Story 2 - Change a password with current credentials (Priority: P1)

An existing user needs to update their password using their username and current password so they can maintain secure access without losing their account.

**Why this priority**: This is the primary user task in the request and directly protects account security. It must be reliable and user-friendly.

**Independent Test**: A user can submit their username and current password, change to a new password, and then sign in with the new password.

**Acceptance Scenarios**:

1. **Given** the user knows their current password, **When** they submit their username, the current password, and a new password, **Then** the password is updated and the user can sign in with the new password.
2. **Given** the user enters an incorrect current password, **When** they submit the password change request, **Then** the system shows an error message and does not change the password.
3. **Given** the user submits a password change without a valid username or current password, **When** the request is processed, **Then** the request is rejected and the user receives a clear validation message.

---

### User Story 3 - Authenticate through the backend service (Priority: P2)

A returning user needs a secure authentication API so they can prove their identity before changing credentials or accessing account-sensitive operations.

**Why this priority**: The authentication and authorization boundary is the gateway to all secure account actions. Without reliable validation, the password-change flow cannot be trusted.

**Independent Test**: A user submits valid credentials to the authentication API and, after successful sign-in, can use the authenticated password-change flow.

**Acceptance Scenarios**:

1. **Given** a user account exists, **When** they submit their valid username and password to the authentication service, **Then** the system accepts the sign-in and returns a successful authentication result.
2. **Given** a user enters an invalid username or password, **When** they attempt to sign in, **Then** the system rejects the sign-in and returns a clear error message.
3. **Given** the user is authenticated, **When** they submit a password change request with a valid current password and a compliant new password, **Then** the system processes the request and updates the stored credentials.

---

### Edge Cases

- If a user submits a blank username or blank password during account creation or sign-in, the system MUST reject the request and show an error message indicating the input is incorrect.
- If a user enters an offensive username or password, the system MUST reject the request and show an error message indicating the value is not allowed.
- If the current password is wrong during a password change attempt, the system MUST reject the request and show an error message without updating the password.
- If the same username is submitted for more than one account, the system MUST reject the duplicate account and show an error message.
- If the new password is the same as the current password, the system MUST reject the request and inform the user that the password is the same.
- If a user attempts to change the password without providing the current password, the system MUST reject the request and ask for the required current password.
- If a password change attempt succeeds or fails, the system MUST record the event with the username, timestamp, and outcome so the result is auditable.
- If multiple password change requests are submitted concurrently, the system MUST prevent race conditions from allowing a stale password check to bypass current validation.
- If a user reaches the password-change rate limit, the system MUST reject additional attempts and return a rate-limit error until the cool-down period ends.
- If an unauthenticated user submits a password-change request, the system MUST reject the request and require valid authentication before allowing the operation.
- If an authenticated user submits a valid password-change request, the system MUST process the update only after the current password has been verified successfully.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow a user to create a login using a username and password.
- **FR-002**: The system MUST reject account creation or sign-in attempts with a blank username or blank password and show an error message stating that the provided input is incorrect.
- **FR-003**: The system MUST reject usernames and passwords containing any value from a fixed list of clearly offensive words and show an error message indicating the value is not allowed.
- **FR-004**: The system MUST treat the account as having a single standard role value instead of multiple role types.
- **FR-005**: The system MUST require a new password to be between 8 and 15 characters long, include uppercase and lowercase letters, include at least one numeric character, include at least one special character, and not match the current password.
- **FR-006**: The system MUST reject a new password that contains the username or common sequential patterns such as repeated numbers or keyboard sequences.
- **FR-007**: The system MUST allow a user to sign in using their username and password.
- **FR-008**: The system MUST reject sign-in attempts when the username and password do not match a valid account.
- **FR-009**: The system MUST allow a password change when the user is authenticated and provides either a valid session/token or a valid username with the existing password as a fallback validation path. When both are supplied, the authenticated session/token is the primary mechanism and the username/current-password values are used only as a fallback when no valid session is present.
- **FR-010**: The system MUST display a clear error message when the provided current password is incorrect.
- **FR-011**: The system MUST prevent the password from being updated when the current password is incorrect.
- **FR-012**: The system MUST reject a password change when the new password is the same as the current password and inform the user that the password is the same.
- **FR-013**: The system MUST reject a password change attempt if the current password is not provided and require the user to enter it first.
- **FR-014**: The system MUST update the stored password only after validating the current password, the password history, and the new password policy.
- **FR-015**: The system MUST maintain a recent password history and block reuse of the user’s immediate previous password.
- **FR-016**: The system MUST enforce a maximum of 3 successful password changes within a rolling 24-hour window, to prevent account hijacking loops.
- **FR-017**: The system MUST log every password change attempt, including the username, timestamp, and whether it succeeded or failed.
- **FR-018**: On a successful password change, the system MUST publish a `password_changed` security event and dispatch a non-blocking notification through the configured notification channel. Notification delivery failure MUST be logged and MUST NOT roll back the successful password update.
- **FR-019**: The system MUST allow a user to sign in again using the updated password after a successful password change.
- **FR-020**: The system MUST preserve the user’s single role setting during password updates.
- **FR-021**: The system MUST provide user-friendly validation and error feedback for missing, invalid, or incorrect credentials.
- **FR-022**: The system MUST prevent duplicate usernames from creating multiple accounts for the same user identity.
- **FR-023**: The system MUST protect the password-change endpoint with a rate limit of 5 attempts per user per 15 minutes.
- **FR-024**: The system MUST use a password epoch or equivalent session invalidation strategy so that old JWTs and stale sessions are rejected after a password change.
- **FR-025**: The system MUST provide an authentication endpoint that accepts a username and password and returns a clear success or error result.
- **FR-026**: The system MUST prefer an authenticated session/token for password-change requests, while also allowing a username-plus-current-password fallback when the caller does not yet have a valid session.
- **FR-027**: The system MUST reject password-change requests from unauthenticated callers when neither a valid session/token nor a valid username/current-password pair is supplied.
- **FR-028**: The backend MUST enforce password validation, current-password verification, password history checks, rate limiting, session invalidation, and audit logging regardless of any client-side validation state.
- **FR-029**: The backend MUST return user-readable validation and error messages without exposing implementation details or internal system errors.

### Key Entities *(include if feature involves data)*

- **User Account**: Represents the login identity for a person. It includes a username, password, a single role setting, password history, and a token epoch or equivalent session revocation marker.
- **Role**: Represents the account’s single role classification, rather than multiple business-role options.
- **Password Policy**: Defines the rules that acceptable passwords must satisfy, including length, character composition, disallowing reuse of the current password, and rejecting the username or obvious sequential content.
- **Password Change Audit Log**: Records each password change attempt with the username, timestamp, and outcome so successful and failed attempts can be reviewed.
- **Security Session**: Tracks active user sessions that are invalidated when a password change succeeds, except for the current session that initiated the change.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of users can create an account and sign in successfully on their first attempt when they provide valid credentials.
- **SC-002**: 100% of password change attempts with a correct current password result in a successful password update and successful follow-up sign-in with the new password.
- **SC-003**: 100% of password change attempts with an incorrect current password result in a clear error message and no password change.
- **SC-004**: 100% of user accounts retain a single assigned role value through account creation and password update workflows.
- **SC-005**: 100% of successful password changes invalidate all other active sessions except the current one and publish a password_changed event for downstream notification processing.
- **SC-006**: 100% of password-change requests are logged with username, timestamp, and success or failure outcome for audit review.
- **SC-007**: Users can complete an account creation and password change workflow without needing support in under 5 minutes when the process is followed as designed.
- **SC-008**: 100% of password-change requests are accepted only when the caller is authenticated or provides a valid username/current-password fallback, and otherwise they are rejected with a clear authentication error.
- **SC-009**: 100% of successful password changes emit a `password_changed` event and create an auditable notification-attempt record, while notification failures do not prevent the password update from succeeding.
- **SC-010**: 100% of JWTs issued before a password change are rejected after the password epoch changes, while the current active session remains valid.

## Assumptions

- Users create accounts with a unique username and a password that meets the project’s standard password policy.
- A valid password must be between 8 and 15 characters, include uppercase and lowercase letters, include a number, include a special character, and differ from the current password during change requests.
- Offensive usernames and passwords are rejected before account creation or password update.
- Each user account has a single role value rather than multiple selectable role types.
- Password changes require the user to know the current password before the system will accept a new one.
- A password change must invalidate all other active sessions, except the current session, and must publish a `password_changed` event for downstream notification handling after the change.
- Notification delivery is asynchronous and best-effort; a failed notification must not roll back the password change itself.
- The system tracks password history and enforces the 3-change-per-24-hour maximum and rate limit for password change attempts.
- Every password change attempt, whether successful or unsuccessful, is recorded in an audit log with the username, time, and outcome.
- Login and password change flows are intended for valid registered users only, and invalid credentials should be rejected with a clear message.
- The system will preserve the user’s identity and role setting when a password is updated, and future sign-ins will require the new password.
- Password-change requests may use either a valid authenticated session/token or a username/current-password fallback, with the authenticated session preferred when available.
- The backend is the source of truth for security enforcement, authentication checks, and user-visible validation responses for all account-management operations.
