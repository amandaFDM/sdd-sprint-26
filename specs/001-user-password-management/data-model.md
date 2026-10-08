# Data Model: User Password Management

## Core entities

### UserAccount

| Field | Type | Constraints | Notes |
|-------|------|------------|-------|
| id | string | required, unique | Internal account identifier |
| username | string | required, unique, trimmed, not blank | Must reject offensive content and duplicates |
| passwordHash | string | required | Stored as a hash, never as plain text |
| role | string | required | Single standardized role value for this application |
| createdAt | timestamp | required | Account creation timestamp |
| updatedAt | timestamp | required | Last update timestamp |

### PasswordChangeRequest

| Field | Type | Constraints | Notes |
|-------|------|------------|-------|
| username | string | required, not blank | Must match an existing account |
| currentPassword | string | required, not blank | Must match stored password hash |
| newPassword | string | required, not blank | Must satisfy password policy and differ from current password |

### AuthenticationResult

| Field | Type | Constraints | Notes |
|-------|------|------------|-------|
| username | string | required | Authenticated user identity |
| role | string | required | Account role value |
| success | boolean | required | Indicates whether credentials match |
| message | string | optional | User-facing error or confirmation text |

### PasswordChangeAuditLog

| Field | Type | Constraints | Notes |
|-------|------|------------|-------|
| id | string | required, unique | Internal event identifier |
| username | string | required | The user associated with the password-change attempt |
| timestamp | timestamp | required | When the attempt occurred |
| outcome | string | required | success or failure |
| reason | string | optional | Validation or authentication failure reason |

### SecuritySession

| Field | Type | Constraints | Notes |
|-------|------|------------|-------|
| sessionId | string | required, unique | Active session identity |
| username | string | required | Account associated with this session |
| status | string | required | active or revoked |
| passwordEpoch | integer | required | Session epoch marker used to validate current JWT validity |

## Validation rules

- Blank username or password is invalid.
- Duplicate usernames are rejected.
- Offensive username and password strings are rejected before persistence.
- Password policy: 8-15 characters, lowercase + uppercase required, numeric character required, special character required, username must not appear in the password, and common sequential strings are not allowed.
- New password cannot equal current password, and the immediate previous password is also rejected in the password history.
- Password changes require username + current password + new password.
- Wrong current password blocks password update and returns a user-readable error.
- Password changes are atomic: either validation passes and the update is stored, or validation fails and the current value remains unchanged.
- Every password-change attempt must generate an audit record showing the outcome as success or failure.
- Successful password changes must invalidate other active sessions and increment the user’s password epoch.
- Password-change requests are rate limited to 5 attempts per user in 15 minutes, and only 3 successful changes are allowed in a rolling 24-hour period.

## Relationships

- One UserAccount has exactly one role value.
- One UserAccount is uniquely identified by username.
- A password change operation is associated with one existing UserAccount.
- One UserAccount can have many PasswordChangeAuditLog records over time.
- One UserAccount can have many SecuritySession records, but only one active session remains valid after a successful password change.
