<!-- Sync Impact Report: Version change: placeholder draft -> 0.1.0 | Modified principles: I. Security by Default, II. Evidence Before Delivery, III. User-Centered Change, IV. Simple, Testable Design, V. Ownership & Traceability | Added sections: Security & Quality Requirements, Delivery Workflow | Removed sections: none | Follow-up TODOs: TODO(RATIFICATION_DATE): original adoption date is not recorded; confirm before formal ratification. -->

# SDD Sprint 26 Constitution

## Core Principles

### I. Security by Default
All user-facing features, especially authentication and account-management flows, MUST enforce least-privilege access, protect secrets, and validate every input before storage or processing. The project MUST treat security as a product requirement, not a late-stage add-on. When a change affects access control or sensitive data handling, the team MUST document the risk, confirm the mitigation, and verify the behavior with automated tests before release.

### II. Evidence Before Delivery
Every significant change MUST be backed by concrete evidence: tests for the intended behavior, clear acceptance criteria, and review of any operational impact. Claims of correctness without verification are not accepted. When a requirement is ambiguous, the team MUST resolve the ambiguity before implementation, and any exception must be explicitly documented with a reason and an owner.

### III. User-Centered Change
The product MUST prioritize user trust, clarity, and predictable behavior in all workflows. A feature is not complete if it is technically correct but confusing, inconsistent, or unsafe for the end user. Changes affecting onboarding, authentication, account recovery, or access management MUST preserve a clear user journey and surface actionable errors instead of opaque failures.

### IV. Simple, Testable Design
The team MUST favor the smallest correct solution, clear naming, and explicit boundaries between responsibilities. Complex logic MUST be decomposed into readable components, validated with focused tests, and documented when the intent is not obvious from code. We do not accept unnecessary abstraction, hidden side effects, or untested branching paths in production code.

### V. Ownership & Traceability
Each change MUST have a clear owner, known impact, and traceable link to the requirement or issue that motivated it. This project requires that task status, assumptions, and decisions remain understandable to future contributors. When behavior changes, the associated tests, documentation, and issue references MUST be updated to match the current state.

## Security & Quality Requirements

The project MUST operate within secure Spring Boot defaults and use explicit validation for all requests that modify account data or authentication state. Password operations MUST reject invalid inputs, enforce the configured policy, and never expose sensitive state in logs, responses, or error messages. The application MUST fail safely when downstream services or data stores are unavailable and MUST provide deterministic, user-readable error handling for recoverable failures.

All code that processes user credentials or account changes MUST be covered by tests that validate both successful outcomes and failure paths. Sensitive data handling MUST minimize exposure, use parameterized queries and safe serialization, and maintain clear separation between business logic and infrastructure concerns. Any material change to security behavior MUST be reviewed for compliance with the governing principles before approval.

## Delivery Workflow

The project MUST follow a disciplined workflow from specification through validation. Features begin with the requirement, assumptions, and constraints captured in a reviewable format; implementation follows only after the expected behavior is clear; and changes are validated through tests and review before merge. Changes to authentication, authorization, and user profile operations require explicit verification of input validation, persistence, and user-visible messaging.

Pull requests MUST document the intent of the change, the tests used to validate it, and any assumptions or constraints that remain open. Operational changes, new dependencies, or security-sensitive updates MUST be reviewed by a knowledgeable project owner before release. Any accepted exception to this workflow must be justified in writing and reviewed as a deliberate governance decision.

## Governance

This constitution governs all project decisions that affect software quality, security, user trust, and delivery discipline. It supersedes informal preferences when the two conflict. Any amendment MUST explain the rationale, identify affected principles or sections, update the version, and document the review outcome for the change.

The project MUST use semantic versioning for constitution changes. A major version change is required for backward-incompatible governance removals or redefinitions; a minor version change is required for new or materially expanded principles or sections; and a patch change is required for clarifications, wording fixes, and non-semantic refinements. When the version bump type is uncertain, the author MUST state the reasoning before the amendment is accepted.

Compliance review is expected for all material changes. Reviewers MUST confirm that the change matches the constitution, that the versioning decision is correct, and that no unintended governance gap remains. Until a ratification date is formally recorded, the project MUST track the original adoption date as TODO(RATIFICATION_DATE): original adoption date is not recorded.

**Version**: 0.1.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date is not recorded | **Last Amended**: 2026-10-08
