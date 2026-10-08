# Quickstart: Validate the User Password Management Flow

## Prerequisites

- Java 25 or the project-supported runtime
- Maven wrapper available in the repository root
- Application running locally on the configured Spring Boot port

## Start the application

```bash
./mvnw spring-boot:run
```

## Validate the authentication API

1. Submit a valid sign-in request with a known username and password and confirm success.
2. Submit a sign-in request with an invalid password and confirm the request is rejected with a clear error.
3. Submit an unauthenticated password-change request and confirm it is rejected until valid authentication is established.

## Validate account creation

1. Submit a valid account creation request with unique username and a compliant password.
2. Confirm the account is created successfully.
3. Submit a duplicate username and confirm the request is rejected with a clear duplicate-account error.
4. Submit a blank username or blank password and confirm the request is rejected.

## Validate sign-in

1. Sign in with a valid username and matching password.
2. Confirm sign-in succeeds.
3. Attempt sign-in with an incorrect password and confirm the request is rejected with a clear error message.

## Validate password change

1. After successful sign-in, submit a valid password change using the authenticated username, current password, and a new compliant password.
2. Confirm the password is updated successfully.
3. Attempt to sign in with the new password and confirm success.
4. Attempt a password change with the wrong current password and confirm it is rejected without changing the stored password.
5. Attempt a password change with the same new password and confirm the request is rejected with the "password is the same" message.
6. Attempt a password change without supplying the current password and confirm the request is rejected.

## Expected outcomes

- Valid flows succeed.
- Invalid inputs produce user-readable error responses.
- Password updates never occur without successful current-password validation.
- Duplicate usernames and prohibited values are blocked.
