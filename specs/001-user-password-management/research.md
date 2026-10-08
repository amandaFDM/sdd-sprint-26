# Research: User Password Management

## Decision: Single-role accounts, verification-first password updates, and epoch-based session invalidation

The feature will use a single account role value rather than multiple role options. Password updates will require the user to provide the current password before any change is accepted, validation will run before persistence, and authentication will be enforced at the API boundary so only valid requests are allowed to modify credentials. Successful password changes will increment a password epoch and emit a `password_changed` event, ensuring stale JWTs are rejected while the current session remains valid.

## Rationale

- The product requirement explicitly corrected the earlier ambiguity to a single role model.
- Password update flows are security-sensitive and should fail closed when credentials are missing or incorrect.
- The feature needs consistent validation for blank inputs, offensive content, duplicate usernames, and password reuse.

## Alternatives considered

1. Multiple user roles
   - Rejected because the current requirement states there is only one role.

2. Password replacement without current-password verification
   - Rejected because it weakens security and contradicts the requirement that the current password must be supplied.

3. Strictly client-side validation only
   - Rejected because server-side validation is required for security and correctness.

4. Allowing duplicate usernames
   - Rejected because identity uniqueness is required for predictable login and account management.

## Implementation approach decisions

- Credential validation should be enforced on both create-login and password-change paths.
- Error messaging should be explicit and user-readable for incorrect inputs and invalid password changes.
- The new password must meet minimum complexity rules and must differ from the current password.
- Offensive content filtering should be treated as a domain validation rule during both account creation and password change.
- The backend will enforce authentication and authorization rules at the service boundary so that password-change requests can only proceed after successful sign-in and current-password verification, with a username/current-password fallback path allowed when the caller does not yet have a valid session.
- Successful password changes will emit a non-blocking `password_changed` event and increment the password epoch so stale tokens are rejected without undoing the password update.

## Open items resolved for planning

All clarification items from the feature specification are resolved enough to proceed to design and implementation planning:
- password policy: 8+ characters, mixed case, numeric content, cannot match current password
- blank values invalid
- duplicate usernames invalid
- password change without current password invalid
- wrong current password invalid
- only a single role value exists
