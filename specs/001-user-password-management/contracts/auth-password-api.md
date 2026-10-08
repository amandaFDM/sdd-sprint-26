# Authentication and Password API Contract

## POST /users/register

Creates a new user account.

### Request body

```json
{
  "username": "string",
  "password": "string"
}
```

### Validation rules

- username required and not blank
- password required and not blank
- username must not be duplicate
- password must satisfy policy rules
- username and password must not contain offensive content
- backend is the source of truth for all validation

### Success response

```json
{
  "success": true,
  "message": "Account created successfully"
}
```

### Error response

```json
{
  "success": false,
  "code": "USERNAME_EXISTS",
  "message": "Username already exists"
}
```

## POST /users/login

Authenticates an existing user.

### Request body

```json
{
  "username": "string",
  "password": "string"
}
```

### Validation rules

- username and password required
- username must exist
- password must match stored credentials
- invalid credentials must not expose internal stack traces or password policy details

### Success response

```json
{
  "success": true,
  "username": "userName",
  "role": "single-role-value",
  "token": "jwt-token"
}
```

### Error response

```json
{
  "success": false,
  "code": "INVALID_CREDENTIALS",
  "message": "Incorrect username or password"
}
```

## PUT /users/password

Changes an existing user password after verification.

### Request body

```json
{
  "username": "string",
  "currentPassword": "string",
  "newPassword": "string"
}
```

### Validation rules

- authenticated session/token is preferred; if both a valid session and a username/currentPassword input are supplied, the session/token takes precedence
- username required if using the fallback path
- currentPassword required in the fallback path
- newPassword required
- currentPassword must match stored password
- newPassword must satisfy policy rules
- newPassword must not equal currentPassword
- username and password values must not be offensive
- all password-change rules are enforced server-side before storing the new password
- successful updates invalidate all other active sessions and increment the password epoch
- any JWT used after the password change must include the current epoch value or the request is rejected as expired

### Success response

```json
{
  "success": true,
  "message": "Password updated successfully"
}
```

### Error response

```json
{
  "success": false,
  "code": "INVALID_CURRENT_PASSWORD",
  "message": "Current password is incorrect"
}
```

### Stale-session rejection response

```json
{
  "success": false,
  "code": "AUTH_TOKEN_REVOKED",
  "message": "Session expired due to a password change."
}
```
